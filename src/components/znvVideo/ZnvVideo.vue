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
function isAppleMobileDevice() {
  return /iphone|ipod|ipad|Macintosh/i.test(navigator.userAgent.toLowerCase());
}

export default {
  name: 'ZnvVideo',
  data() {
    return {
      options: {
        techOrder: ['html5', 'Hlsjs', 'Flvjs', 'Streamedianjs'],
        autoplay: true,
        controls: true, // 是否显示控制条
        language: 'zh-CN', // 设置语言
        muted: true, // 是否静音
        controlBar: {
          progressControl: true,
          liveDisplay: true,
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
    cameraId: {
      type: String,
      required: false,
      default: ""
    },
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
    let self = this
    const Button = window.videojs.getComponent('Button')
    const Component = window.videojs.getComponent('Component')
    class VideoBackButton extends Button {
      constructor(player, options = {}) {
        super(player, options)
        this.controlText('Halfscreen')
      }
      buildCSSClass() {
        return 'vjs-video-big'
      }
      handleClick() {
        self.setBigscreen(this.el_)
      }
    }
    Component.registerComponent('VideoBackButton', VideoBackButton)
    this.player = videojs('znv-video-' + this.vId, this.options)
    this.player.getChild('controlBar').addChild('VideoBackButton', {}, 4)
    this.setSrc()
  },
  watch: {
    src() {
      this.setSrc()
    },
  },
  methods: {
    setBigscreen(el) {
      if (this.isBigScreen) {
        document
          .getElementById('znv-video-' + this.vId)
          .classList.remove('big-center-video')
        let tempDom = document.getElementById('tempDiv' + this.vId)
        tempDom.remove()
        document
          .getElementById('box-video-' + this.vId)
          .append(document.getElementById('znv-video-' + this.vId))
        el.className = 'vjs-video-big'
      } else {
        let tempDom = document.createElement('div')
        tempDom.id = 'tempDiv' + this.vId
        tempDom.className = 'temp-div'
        this.$el.appendChild(tempDom)
        document.body.append(document.getElementById('znv-video-' + this.vId))
        document
          .getElementById('znv-video-' + this.vId)
          .classList.add('big-center-video')
        el.className = 'vjs-video-small'
      }
      this.isBigScreen = !this.isBigScreen
    },
    setSrc() {
      if (this.type === 'hls') {
        if (isAppleMobileDevice()) {
          this.initMp4()
        } else {
          this.initHls()
        }
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
        type: 'video/x-hls',
      })
    },
    initRtsp() {
      let self = this
      if (this.cameraId) {
        $.ajax({
          url: 'apis/rtsp/queryUrl?cameraId=' + this.cameraId,
          type : "GET",
          headers: {
            Authorization: '12CBD9B708D887A41AFAB97DAC46AAC6'
          },
          success : function(res) {
            let data = res.data[0]
            self.player.src({
              src: data.rtspUrl,
              type: 'video/rtsp',
              withCredentials: false,
              statuHandler: self.stuHandler,
              socket: data.rtspWebSocketUrl,
              bufferDuration: 5,
            })
          },
          error : function() {
            console.log('H264参数请求失败')
          }
        })
      } else {
        this.player.src({
          src: this.src,
          type: 'video/rtsp',
          withCredentials: false,
          statuHandler: this.stuHandler,
          socket: this.ws,
          bufferDuration: 5,
        })
      }
    },
    stuHandler(currentProxy, message) {
      if (message === 9000001) {
        this.rtspMessage = '正在请求视频'
      } else if (message === 9000002) {
        this.rtspMessage = '请求视频失败'
      } else if (message === 9000003) {
        this.rtspMessage = '正在重新拉取视频'
      } else if (message === 9000004) {
        this.rtspMessage = '正在重连视频'
      } else if (message === 9000008) {
        this.rtspMessage = '请求h265'
        this.connectH265()
      } else {
        this.rtspMessage = message.toFixed(2) + 'kbps'
      }
    },
    connectH265() {
      let self = this
      this.dispose()
      $.ajax({
        url: 'apis/rtsp/queryUrl?isH265=1&cameraId=' + this.cameraId,
        type : "GET",
        success : function(res) {
          let data = res.data[0]
          self.streamMark = data.streamMark
          self.player.src({
            socket: data.wsUrl,
            redirectNativeMediaErrors: true,
            bufferDuration: 5,
            statuHandler: self.stuHandler,
            isH265: true,
            isH265Url: data.wsUrl,
            streamMark: data.streamMark
          })
        },
        error : function() {
          console.log('H265参数请求失败')
        }
      })
    },
    initFlv() {
      this.player.src({
        src: this.src,
        type: 'video/x-flv',
      })
      this.player.on( 'play', e =>{
        this.player.tech_.flvPlayer.on('error', e => {
          console.log(e)
          console.log('测试城市之眼错误开始重连')
        })
        // flv帧追赶技术 目前2S执行一次 如果延迟超出2S则往前追赶2S
        this.flvInterval = setInterval(() => {
          if (this.player.tech_.flvPlayer.buffered.length) {
            let end = this.player.tech_.flvPlayer.buffered.end(0); // 获取当前buffered值
            let diff = end - this.player.tech_.flvPlayer.currentTime; // 获取buffered与currentTime的差值
            if (diff >= 2) { // 如果差值大于等于0.5 手动跳帧 这里可根据自身需求来定
              this.player.tech_.flvPlayer.currentTime += 2 // 手动跳帧
            }
          }
        }, 2000); //2000毫秒执行一次 
      })
    },
    initMp4() {
      this.player.src({
        src: this.src
      })
    },
    dispose() {
      if (this.player) {
        this.player.dispose()
        this.player = null
      }
    }
  },
  beforeDestroy() {
    let self = this
    if (this.rtspMessage === '请求h265') {
      $.ajax({
        url: 'apis/rtsp/closeStream?streamMark=' + self.streamMark,
        type : "GET",
        success : function(res) {
          self.dispose()
        },
        error : function() {
          console.log('销毁H265失败')
        }
      })
    } else {
      self.dispose()
    }
    this.flvInterval && clearInterval(this.flvInterval)
  }
}
</script>

<style lang="scss">
.big-center-video {
  position: absolute;
  left: 27.4%;
  top: 15%;
  width: 46.8%;
  height: 75%;
  font-size: 20px;
  z-index: 9;
  .vjs-tech {
    pointer-events: none;
    object-fit: fill;
  }
}
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
  .rtsp-video, .hls-video{
    .vjs-progress-control{
      visibility: hidden;
    }
  }
  video{
    object-fit: fill;
  }
}
</style>
