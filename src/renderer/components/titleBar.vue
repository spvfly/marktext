<template>
  <div class="title-bar"
    :class="[{ 'active': active }, theme]"
  >
    <div class="title">
      <span
        v-for="(path, index) of paths"
        :key="index"
      >
        {{ path }}
        <svg class="icon" aria-hidden="true">
          <use xlink:href="#icon-arrow-right"></use>
        </svg>
      </span>
      <span @click="rename">{{ filename }}</span>
      <span class="save-dot" :class="{'show': !isSaved}"></span>
    </div>
    <div :class="platform !== 'darwin' ? 'left-toolbar' : 'right-toolbar'">
      <div
        v-if="platform !== 'darwin'"
        class="frameless-titlebar-menu title-no-drag"
        @click.stop="handleMenuClick"
      >&#9776;</div>
      <div
        class="word-count"
        :class="[{ 'title-no-drag': platform !== 'darwin' }]"
        @click.stop="handleWordClick"
      >{{ `${HASH[show]} ${wordCount[show]}` }}</div>
    </div>
    <div
      v-if="platform !== 'darwin'"
      class="right-toolbar"
      :class="[{ 'title-no-drag': platform !== 'darwin' }]"
    >
      <div class="frameless-titlebar-button frameless-titlebar-close" @click.stop="handleCloseClick">
        <div>
          <svg width="10" height="10">
            <path :d="windowIconClose" />
          </svg>
        </div>
      </div>
      <div class="frameless-titlebar-button frameless-titlebar-toggle" @click.stop="handleMaximizeClick">
        <div>
          <svg width="10" height="10">
            <path v-show="!isMaximized" :d="windowIconMaximize" />
            <path v-show="isMaximized" :d="windowIconRestore" />
          </svg>
        </div>
      </div>
      <div class="frameless-titlebar-button frameless-titlebar-minimize" @click.stop="handleMinimizeClick">
        <div>
          <svg width="10" height="10">
            <path :d="windowIconMinimize" />
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { remote } from 'electron'
  import { minimizePath, restorePath, maximizePath, closePath } from '../assets/window-controls.js'

  export default {
    data () {
      this.HASH = {
        'word': 'W',
        'character': 'C',
        'paragraph': 'P',
        'all': 'A'
      }
      this.windowIconMinimize = minimizePath
      this.windowIconRestore = restorePath
      this.windowIconMaximize = maximizePath
      this.windowIconClose = closePath
      return {
        isMaximized: remote.getCurrentWindow().isMaximized() || remote.getCurrentWindow().isFullScreen(),
        show: 'word'
      }
    },
    created () {
      const win = remote.getCurrentWindow()
      ;['maximize', 'unmaximize', 'enter-full-screen', 'leave-full-screen'].forEach(type => {
        win.on(type, this.handleWindowStateChanged)
      })
    },
    props: {
      filename: String,
      pathname: String,
      active: Boolean,
      wordCount: Object,
      theme: String,
      platform: String,
      isSaved: Boolean
    },
    computed: {
      paths () {
        const pathnameToken = this.pathname.split('/').filter(i => i)
        return pathnameToken.slice(0, pathnameToken.length - 1).slice(-3)
      }
    },
    methods: {
      handleWordClick () {
        const ITEMS = ['word', 'paragraph', 'character', 'all']
        const len = ITEMS.length
        let index = ITEMS.indexOf(this.show)
        index += 1
        if (index >= len) index = 0
        this.show = ITEMS[index]
      },

      handleCloseClick () {
        remote.getCurrentWindow().close()
      },

      handleMaximizeClick () {
        const win = remote.getCurrentWindow()
        if (win.isFullScreen()) {
          win.setFullScreen(false)
        } else if (win.isMaximized()) {
          win.unmaximize()
        } else {
          win.maximize()
        }
      },

      handleMinimizeClick () {
        remote.getCurrentWindow().minimize()
      },

      handleMenuClick () {
        const win = remote.getCurrentWindow()
        remote
          .Menu
          .getApplicationMenu()
          .popup({ window: win, x: 23, y: 20 })
      },

      handleWindowStateChanged () {
        this.isMaximized = remote.getCurrentWindow().isMaximized() || remote.getCurrentWindow().isFullScreen()
      },

      rename () {
        if (this.platform === 'darwin') {
          this.$store.dispatch('RESPONSE_FOR_RENAME')
        }
      }
    },
    beforeDestroy () {
      const win = remote.getCurrentWindow()
      ;['maximize', 'unmaximize', 'enter-full-screen', 'leave-full-screen'].forEach(type => {
        win.removeListener(type, this.handleWindowStateChanged)
      })
    }
  }
</script>

<style scoped>
  .title-bar {
    background: rgb(252, 252, 252);
    -webkit-app-region: drag;
    user-select: none;
    width: 100%;
    height: 22px;
    box-sizing: border-box;
    color: #F2F6FC;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1;
    transition: color .4s ease-in-out;
    cursor: default;
  }
  .active {
    color: #909399;
  }
  img {
    height: 90%;
    margin-top: 1px;
    vertical-align: top;
  }
  .title {
    padding: 0 100px;
    height: 100%;
    line-height: 22px;
    font-size: 12px;
    font-weight: 500;
    text-align: center;
    transition: all .25s ease-in-out;
  }

  .active .save-dot {
    margin-left: 3px;
    width: 7px;
    height: 7px;
    display: inline-block;
    border-radius: 50%;
    background: rgba(242, 134, 94, .7);
    visibility: hidden;
  }
  .active .save-dot.show {
    visibility: visible;
  }
  .title:hover {
    color: #303133;
  }
  .title:hover .save-dot {
    background: rgb(242, 134, 94);
  }
  .right-toolbar {
    padding: 0 10px;
    height: 100%;
    position: absolute;
    top: 0;
    right: 0;
    width: 100px;
    display: flex;
    flex-direction: row-reverse;
  }
  .left-toolbar {
    padding: 0 10px;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    width: 100px;
    display: flex;
    flex-direction: row;
  }
  .word-count {
    cursor: pointer;
    font-size: 12px;
    color: #F2F6FC;
    height: 15px;
    line-height: 15px;
    margin-top: 4px;
    padding: 1px 5px;
    border-radius: 1px;
    transition: all .25s ease-in-out;
  }
  .active .word-count {
    color: #DCDFE6;
  }
  .word-count:hover {
    background: #F2F6FC;
    color: #606266;
  }
  .title-no-drag {
    -webkit-app-region: no-drag;
  }
  /* frameless window controls */
  .frameless-titlebar-button {
    position: relative;
    display: block;
    width: 25px;
    height: 25px;
  }
  .frameless-titlebar-button > div {
    position: absolute;
    display: inline-flex;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
  }
  .frameless-titlebar-menu {
    color: #606266;
  }
  .frameless-titlebar-close:hover {
    background-color: rgb(228, 79, 79);
  }
  .frameless-titlebar-minimize:hover,
  .frameless-titlebar-toggle:hover {
    background-color: rgba(0, 0, 0, 0.1);
  }
  .frameless-titlebar-button svg {
    fill: #000000
  }
  .frameless-titlebar-close:hover svg {
    fill: #ffffff
  }
  /* css for dark theme */
  .dark {
    background: rgb(43, 43, 43);
    color: #909399;
  }
  .dark .title:hover {
    color: #F2F6FC;
  }
  .dark .word-count:hover {
    background: rgb(71, 72, 66);
    color: #C0C4CC;
  }
  .dark .frameless-titlebar-button svg {
    fill: #909399
  }
  .dark .frameless-titlebar-close:hover svg {
    fill: #ffffff
  }
</style>
