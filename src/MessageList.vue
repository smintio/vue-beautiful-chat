<template>
  <div class="sc-message-list" ref="scrollList" :style="{backgroundColor: colors.messageList.bg}" @scroll="handleScroll">
    <Message v-for="(message, idx) in messages" :message="message" ref="messageEntry" :user="profile(message.author)" :key="idx" :colors="colors" :messageStyling="messageStyling" @remove="$emit('remove', message)">
      <template v-slot:user-avatar="scopedProps">
        <slot name="user-avatar" :user="scopedProps.user" :message="scopedProps.message">
        </slot>
      </template>
      <template v-slot:text-message-body="scopedProps">
        <slot name="text-message-body" :message="scopedProps.message" :messageText="scopedProps.messageText" :messageColors="scopedProps.messageColors" :me="scopedProps.me">
        </slot>
      </template>
      <template v-slot:text-message-toolbox="scopedProps">
        <slot name="text-message-toolbox" :message="scopedProps.message" :me="scopedProps.me">
        </slot>
      </template>
    </Message>
    <Message v-show="showTypingIndicator !== ''" :message="{author: showTypingIndicator, type: 'typing'}" :user="{}" :colors="colors" :messageStyling="messageStyling" />
  </div>
</template>
<script>
import Message from './Message.vue'
import chatIcon from './assets/chat-icon.svg'

export default {
  components: {
    Message
  },
  data: function data() {
    return {
      currentMessageInScrollView: {
        message: undefined,
        offsetTop: 0,
      },
    };
  },
  props: {
    participants: {
      type: Array,
      required: true
    },
    messages: {
      type: Array,
      required: true
    },
    showTypingIndicator: {
      type: String,
      required: true
    },
    colors: {
      type: Object,
      required: true
    },
    alwaysScrollToBottom: {
      type: Boolean,
      required: true
    },
    messageStyling: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    _scrollDown () {
      this.$refs.scrollList.scrollTop = this.$refs.scrollList.scrollHeight
    },
    handleScroll (e) {

      let currentMinimalOffsetTop = Number.MAX_SAFE_INTEGER;
      let currentMessageAtTop = undefined;
      for (let i=0; this.$refs.messageEntry && i < this.$refs.messageEntry.length; i++) {

        const offsetTop = this.$refs.messageEntry[i].offsetTop;
        if (offsetTop >= 0 && offsetTop < currentMinimalOffsetTop) {

          currentMinimalOffsetTop = offsetTop;
          currentMessageAtTop = this.messageList[i];

          if (offsetTop === 0) {
            break;
          }
        }
      }

      this.currentMessageInScrollView.message = currentMessageAtTop;
      this.currentMessageInScrollView.offsetTop = currentMinimalOffsetTop;

      if (e.target.scrollTop === 0) {
        this.$emit('scrollToTop')
      }
    },
    shouldScrollToBottom() {
      return this.alwaysScrollToBottom || (this.$refs.scrollList.scrollTop > this.$refs.scrollList.scrollHeight - 600)
    },
    scrollIntoView(message, offsetTop) {

      if (message === undefined || message === null || typeof message !== 'object') {
        return;
      }

      if (offsetTop === undefined || offsetTop === null || typeof offsetTop !== 'number') {
        offsetTop = 0;
      }

      // find the message in the list to scroll to top
      let messageToScrollIntoView = undefined;
      let currentMessageScrollTop = 0;
      for (let i=0; this.$refs.messageEntry && i < this.$refs.messageEntry.length; i++) {

        const currentMessage = this.messageList[i];
        if (currentMessage.id === message.id || (
            currentMessage.author === message.author
            && currentMessage.type === message.type
            && currentMessage.data.meta === message.data.meta
            && (
              currentMessage.data.emoji === message.data.emoji
              || currentMessage.data.text === message.data.text
              || (
                currentMessage.data.file !== undefined && currentMessage.data.file !== null
                && message.data.file !== undefined && message.data.file !== null
                && currentMessage.data.file.name === message.data.file.name
                && currentMessage.data.file.url === message.data.file.url
              )
            )
          )
        ) {
          messageToScrollIntoView = currentMessage;
          currentMessageScrollTop = this.$refs.messageEntry[i].scrollTop;
          break;
        }
      }

      if (messageToScrollIntoView && currentMessageScrollTop) {
        this.$refs.scrollList.scrollTop += offsetTop - currentMessageScrollTop;
      }

    },
    profile(author) {
      const profile = this.participants.find(profile => profile.id === author)

      // A profile may not be found for system messages or messages by 'me'
      return profile || {imageUrl: '', name: ''}
    }
  },
  computed: {
    defaultChatIcon() {
      return chatIcon
    }
  },
  mounted () {
    this.$nextTick(this._scrollDown())
  },
  updated () {

    if (this.shouldScrollToBottom()) {
      this.$nextTick(this._scrollDown())
    } else {
      this.$nextTick(function() {
        this.scrollIntoView(this.currentMessageInScrollView.message, this.currentMessageInScrollView.offsetTop);
      }.bind(this));
    }
  },
}
</script>

<style scoped>
.sc-message-list {
  height: 80%;
  overflow-y: auto;
  background-size: 100%;
  padding: 40px 0px;
}
</style>
