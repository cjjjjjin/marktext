<template>
  <div
    v-if="currentNotification"
    class="editor-notifications"
    :class="currentNotification.style"
    :style="{'max-width': showSideBar ? `calc(100vw - ${sideBarWidth}px` : '100vw' }"
  >
    <div class="msg">
      {{ currentNotification.msg }}
    </div>
    <div class="controls">
      <div>
        <span
          class="inline-button"
          v-if="currentNotification.showConfirm"
          @click.stop="handleClick(true)"
        >
          Ok
        </span>
        <span
          class="inline-button"
          @click.stop="handleClick(false)"
        >
          <svg class="close-icon icon" aria-hidden="true">
            <use id="default-close-icon" xlink:href="#icon-close-small"></use>
          </svg>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data () {
    return {
      autoCloseTimer: null
    }
  },
  computed: {
    ...mapState({
      currentFile: state => state.editor.currentFile,
      showSideBar: state => state.layout.showSideBar,
      sideBarWidth: state => state.layout.sideBarWidth
    }),
    currentNotification () {
      const notifications = this.currentFile.notifications
      if (!notifications || notifications.length === 0) {
        return null
      }
      return notifications[0]
    }
  },
  watch: {
    currentNotification (value) {
      this.clearAutoCloseTimer()
      if (value && value.dismissAfter > 0) {
        this.autoCloseTimer = setTimeout(() => {
          this.dismissNotification(false, false)
        }, value.dismissAfter)
      }
    }
  },
  beforeDestroy () {
    this.clearAutoCloseTimer()
  },
  methods: {
    clearAutoCloseTimer () {
      if (this.autoCloseTimer) {
        clearTimeout(this.autoCloseTimer)
        this.autoCloseTimer = null
      }
    },
    dismissNotification (status, invokeAction = true) {
      this.clearAutoCloseTimer()
      const notifications = this.currentFile.notifications
      if (!notifications || notifications.length === 0) {
        console.error('notifications::dismissNotification: Cannot find notification on stack.')
        return
      }

      const item = notifications.shift()
      if (invokeAction && item.action) {
        item.action(status)
      }
    },
    handleClick (status) {
      this.dismissNotification(status)
    }
  }
}
</script>

<style scoped>
  .editor-notifications {
    position: relative;
    display: flex;
    flex-direction: row;
    max-height: 100px;
    margin-top: 4px;
    background: var(--notificationPrimaryBg);
    color: var(--notificationPrimaryColor);
    padding: 8px 10px;
    user-select: none;
    overflow: hidden;
    &.warn {
      background: var(--notificationWarningBg);
      color: var(--notificationWarningColor);
    }
    &.crit {
      background: var(--notificationErrorBg);
      color: var(--notificationErrorColor);
    }
  }
  .msg {
    font-size: 13px;
    flex: 1;
  }
  .controls {
    display: flex;
    flex-direction: column;
    justify-content: center;
    & > div {
      display: flex;
      flex-direction: row;
    }
    & .inline-button:not(:last-child) {
      margin-right: 3px;
    }
    & .inline-button {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 24px;
      height: 24px;
      font-size: 12px;
      cursor: pointer;
      border: 1px solid rgba(255, 255, 255, 0.1);
    }
    & .inline-button:hover {
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.6);
    }
  }
</style>
