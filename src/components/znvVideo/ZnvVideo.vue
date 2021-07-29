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

export default {
  name: 'ZnvVideo',
  data() {
    return {
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
    vId: {
      type: String,
      required: true,
      default: '',
    },
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
    setSrc() {
      if (this.type === 'hls') {
        this.initHls()
      } else if (this.type === 'rtsp') {
        this.initRtsp()
      } else if (this.type === 'flv') {
        this.initFlv()
      } else if (this.type === 'mp4') {
        this.initMp4()
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
    initMp4() {
      this.player.src({
        src: this.src
      })
    }
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
