<template>
  <div class="box-video" :id="'box-video-' + vId">
    <video :id="'znv-video-' + vId" class="video-js znv-video" :class="type + '-video'">
      <p class="vjs-no-js">
        To view this video please enable JavaScript, and consider upgrading to a
        web browser that
        <a
          href="https://videojs.com/html5-video-support/"
          target="_blank"
        >supports HTML5 video</a>
      </p>
    </video>
  </div>
</template>

<script>
/**
 * 流媒体播放器2.0
 * @module 组件存放位置
 * @desc 同时支持rtsp、flv和hls三种流媒体，相比1.0去掉的rtmp的支持，同时使用video.js统一UI和API,代码更加精简，更容易集成以及使用。
 * @author 李志伟0049003294
 * @date 2021年05月20日10:22:43
 * @param {String} [src]    - 流媒体链接 必传
 * @param {String} [type]    - 流媒体类型 必传
 */

export default {
  name: 'ZnvVideo',
  data() {
    return {
      vId: '',
      options: {
        techOrder: ['html5', 'Flvjs', 'Streamedianjs'],
        autoplay: true,
        controls: true, // 是否显示控制条
        language: 'zh-CN', // 设置语言
        muted: true, // 是否静音
        controlBar: {
          // progressControl: false,
          // liveDisplay: false,
          fullscreenToggle: true, // 全屏按钮，默认为true
          pictureInPictureToggle: true, // 画中画按钮，默认为true
          volumePanel: true, // 声音面板
          currentTimeDisplay: false, // 当前播放时间
          timeDivider: false, // 时间分割线
          durationDisplay: false, // 总时间
          remainingTimeDisplay: false, // 剩余时间
        },
      },
    }
  },
  props: {
    src: {
      type: String,
      required: true,
      default: '',
    },
    type: {
      type: String,
      required: true,
      default: '',
    },
    ws: {
      type: String,
      required: false,
      default: '',
    },
  },
  created() {
    this.vId = this.createUUID()
  },
  mounted() {
    this.player = videojs('znv-video-' + this.vId, this.options)
    this.setSrc()
  },
  watch: {
    src() {
      this.setSrc()
    },
  },
  methods: {
    createUUID() {
      let d = new Date().getTime()
      if (window.performance && typeof window.performance.now === 'function') {
        d += performance.now() // use high-precision timer if available
      }
      let uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(
        /[xy]/g,
        function (c) {
          let r = (d + Math.random() * 16) % 16 | 0
          d = Math.floor(d / 16)
          return (c == 'x' ? r : (r & 0x3) | 0x8).toString(16)
        }
      )
      return uuid
    },
    setSrc() {
      if (this.type === 'hls') {
        this.initHls()
      } else if (this.type === 'rtsp') {
        this.initRtsp()
      } else if (this.type === 'flv') {
        this.initFlv()
      }
    },
    initHls() {
      this.player.src({
        src: this.src,
        type: 'application/x-mpegURL',
      })
    },
    initRtsp() {
      this.player.src({
        src: this.src,
        type: 'video/rtsp',
        withCredentials: false,
        socket: this.ws,
        bufferDuration: 5,
      })
    },
    initFlv() {
      this.player.src({
        src: this.src,
        type: 'video/x-flv',
      })
    },
  },
  beforeDestroy() {
    if (this.player) {
      this.player.dispose()
      this.player = null
    }
  }
}
</script>

<style lang="scss">
.box-video {
  width: 100%;
  height: 100%;
  .znv-video{
    width: 100%;
    height: 100%;
    .vjs-tech {
      pointer-events: none;
    }
  }
  .rtsp-video{
    .vjs-progress-control{
      visibility: hidden;
    }
  }
}
</style>
