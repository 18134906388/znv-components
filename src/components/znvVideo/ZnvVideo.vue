<template>
    <div class="znv-video" @mouseenter="soltShow = true" @mouseleave="soltShow = false">
      <div v-if="type === 'hls'" class="hls-container">
        <video v-if="playerType === 'videoJs'" :id="vId" style="width: 100%; height:100%;" class="video-js vjs-default-skin vjs-big-play-centered video"></video>
        <video v-else muted :id="vId" class="video" autoplay controls></video>
      </div>
      <video v-if="type === 'rtsp'" :id="vId" controls autoplay muted class="video">
        <source :src="src" type="application/x-rtsp" />
      </video>
      <div v-if="type === 'rtmp'" class="hls-container">
        <video v-if="playerType === 'videoJs'" :id="vId" style="width: 100%; height:100%;" class="video-js vjs-default-skin vjs-big-play-centered video">
          <source :src="src" type="rtmp/flv">
        </video>
        <video v-else :id="vId" class="video" autoplay controls muted playsInline webkit-playsinline></video>
      </div>
      <video v-if="type === 'mp4'" :src="src" :id="vId" controls autoplay muted loop class="video"></video>
      <video v-if="type === 'flv'" :src="src" :id="vId" controls autoplay muted loop class="video"></video>
      <transition name="fade">
        <div class="opt-box" v-show="$scopedSlots.$hasNormal && soltShow">
          <slot></slot>
        </div>
      </transition>
    </div>
</template>

<script>
/**
* 流媒体播放器
* @module 组件存放位置
* @desc 同时支持rtsp、rtmp、flv和hls四种流媒体，hls、rtmp支持VideoJsPlayer和EZUIPlayer两种解码库
* @author 李志伟0049003294
* @date 2019年11月23日10:22:43
* @param {String} [src]    - 流媒体链接 必传
* @param {String} [vId] - 播放器ID 必传
* @param {String} [type]    - 流媒体类型 必传
* @param {String} [playerType] - 针对HLS、RTMP类型的流媒体指定解码库 非必传 EZUIPlayer videoJs
*/
import videojs from 'video.js'
import 'videojs-flash'
import 'video.js/dist/video-js.css'
import language from '../../../video/hlsVideoLanguage.js'
import '../../../video/streamedian/streamedian.min.js'
import EZUIKit from '../../../video/EZUIKit-JavaScript-master/ezuikit.js'
import flvjs from '../../../video/flv-epubcn/index.js'
import $ from 'jquery'

if (!window.EZUIPlayer) {
  window.EZUIPlayer = EZUIKit.EZUIPlayer
}
// videojs.options.flash.swf = 'video-js.swf'
videojs.addLanguage('zh-CN', {
  ...language.language,
  'Picture-in-Picture': '画中画'
})

export default {
  name: 'ZnvVideo',
  data() {
    return {
      soltShow: false,
      isSmallDom: false
    }
  },
  props: {
    src: {
      type: String,
      required: true,
      default: ''
    },
    vId: {
      type: String,
      required: true,
      default: ''
    },
    type: {
      type: String,
      required: true,
      default: ''
    },
    playerType: {
      type: String,
      required: false,
      default: 'videoJs'
    },
    rtspWebSocketUrl: {
      type: String,
      required: false,
      default: ''
    },
  },
  mounted () {
    if (this.type === 'hls') {
      this.initHls()
    } else if (this.type === 'rtsp') {
      this.initRtsp()
    } else if (this.type === 'rtmp') {
      this.initRtmp()
    } else if (this.type === 'flv') {
      this.initFlv()
    }
  },
  watch: {
    src () {
      this.initZnvVideo()
      if (this.type === 'hls') {
        if (this.playerType === 'videoJs') {
          this.player.src({
            src: this.src,
            type: 'application/x-mpegURL',
            withCredentials: false
          })
        } else {
          this.$nextTick(() => {
            $('#' + this.vId).attr('src', this.src)
            this.player = new window.EZUIPlayer(this.vId)
            this.player.play()
            this.player.znvVideoSrcType = 'hls'
            this.player.znvVideoPlayerType = 'EZUIPlayer'
          })
        }
      } else if (this.type === 'rtsp') {
        this.initRtsp()
      } else if (this.type === 'rtmp') {
        if (this.playerType === 'videoJs') {
          this.player.src({
            src: this.src,
            type: 'rtmp/flv'
          })
        } else {
          this.$nextTick(() => {
            $('#' + this.vId).attr('src', this.src)
            this.player = new window.EZUIPlayer(this.vId)
            this.player.play()
            this.player.znvVideoSrcType = 'rtmp'
            this.player.znvVideoPlayerType = 'EZUIPlayer'
          })
        }
      } else if (this.type === 'flv') {
        this.initFlv()
      }
    }
  },
  methods: {
    // 切换视频流时调用，初始化播放器
    initZnvVideo() {
      if (this.player) {
        switch (this.player.znvVideoSrcType) {
          case 'hls' :
            if (this.player.znvVideoPlayerType === 'EZUIPlayer') {
              this.player.stop()
              $('#' + this.vId).attr('src', '')
            }
            break
          case 'rtsp' :
            this.destroyRtsp()
            break
          case 'rtmp' :
            if (this.player.znvVideoPlayerType === 'EZUIPlayer') {
              this.player.stop()
              $('#' + this.vId).replaceWith(`<video id="${this.vId}" class="video" autoplay controls muted></video>`)
            }
            break
          case 'flv' :
            this.destroyFlv()
            break
        }
      }
    },
    // 初始化HLS视频流
    initHls () {
      if (!this.src) {
        return
      }
      if (this.playerType === 'videoJs') {
        this.player = videojs(this.vId, {
          autoplay: true,
          controls: true,
          muted: true,
          language: 'zh-CN',
          techOrder: ['html5'],
          controlBar: {
            fullscreenToggle: true, // 全屏按钮，默认为true
            pictureInPictureToggle: true, // 画中画按钮，默认为true
            volumePanel: true, // 声音面板
            currentTimeDisplay: false, // 当前播放时间
            timeDivider: false, // 时间分割线
            durationDisplay: false, // 总时间
            remainingTimeDisplay: false // 剩余时间，
          }
        })
        this.player.src({
          src: this.src,
          type: 'application/x-mpegURL',
          withCredentials: false
        })
        this.player.znvVideoSrcType = 'hls'
        this.player.znvVideoPlayerType = 'videoJs'
      } else {
        this.$nextTick(() => {
          $('#' + this.vId).attr('src', this.src)
          this.player = new window.EZUIPlayer(this.vId)
          this.player.play()
          this.player.znvVideoSrcType = 'hls'
          this.player.znvVideoPlayerType = 'EZUIPlayer'
        })
      }
    },
    // 初始化RTSP视频流
    initRtsp () {
      let self = this
      this.$nextTick(() => {
        if (window.Streamedian && this.src) {
          let errHandler = function (err) {
            console.log(err.message)
          }
          let stuHandler = function (currentProxy, message) {
            if (message === 9000003) {
              self.reconnectWs()
            }
          }
          let playerOptions = {
            socket: this.rtspWebSocketUrl,
            redirectNativeMediaErrors: true,
            bufferDuration: 5,
            errorHandler: errHandler,
            statuHandler: stuHandler
          }
          this.player = window.Streamedian.player(
            this.vId,
            playerOptions
          )
          this.player.znvVideoSrcType = 'rtsp'
          this.player.znvVideoPlayerType = 'streamedian'
        }
      })
    },
    reconnectWs() {
      this.destroyRtsp()
      this.initRtsp()
    },
    // 初始化RTMP视频流
    initRtmp () {
      if (!this.src) {
        return
      }
      if (this.playerType === 'videoJs') {
        this.$nextTick(() => {
          this.player = videojs(this.vId, {
            autoplay: true,
            controls: true,
            muted: true,
            language: 'zh-CN',
            controlBar: {
              fullscreenToggle: true, // 全屏按钮，默认为true
              pictureInPictureToggle: false, // 画中画按钮，默认为true
              volumePanel: true, // 声音面板
              currentTimeDisplay: false, // 当前播放时间
              timeDivider: false, // 时间分割线
              durationDisplay: false, // 总时间
              remainingTimeDisplay: false // 剩余时间，
            }
          })
          this.player.src({
            src: this.src,
            type: 'rtmp/flv'
          })
          this.player.znvVideoSrcType = 'rtmp'
          this.player.znvVideoPlayerType = 'videoJs'
        })
      } else {
        this.$nextTick(() => {
          $('#' + this.vId).attr('src', this.src)
          this.player = new window.EZUIPlayer(this.vId)
          this.player.play()
          this.player.znvVideoSrcType = 'rtmp'
          this.player.znvVideoPlayerType = 'EZUIPlayer'
        })
      }
    },
    // 初始化flv视频流
    initFlv() {
      let enableDurationMonitor = true
      console.log(this.getDeviceKind())
      if (this.getDeviceKind() === 'mobile') {
        enableDurationMonitor = false
      }
      this.$nextTick(() => {
        if (flvjs.isSupported() && this.src) {
          let videoElement = document.getElementById(this.vId)
          this.player = flvjs.createPlayer({
            type: 'flv',
            url: this.src
          }, {
            isLive: true,                
            enableStashBuffer: false,
            autoCleanupMaxBackwardDuration: 60,
            autoCleanupMinBackwardDuration: 30,
            statisticsInfoReportInterval: 2000,
            stashInitialSize: 128 * 1024,
            // 如果是Android浏览器，建议enableDurationMonitor设置为false
            enableDurationMonitor: enableDurationMonitor,    // true表示监测当前直播流延时，当发现延时过大时，主动追赶
            enableVideoFrozenMonitor: true, // 监测视频解码是否停滞（画面卡停），当因为某些原因导致无法解码时，将上报VIDEO_FROZEN事件，收到后建议重拉流
            videoStateMonitorInterval: 1000, // 多长时间（毫秒）检查一次视频状态（延时、停滞）
            // 针对手机浏览器上对MSE以及网络连接更加容易不稳定，建议将maxDurationGap设置高一点，比如2.5、3、3.5，否则可能会频繁追赶延时导致画面卡顿
            maxDurationGap: 1.5,         // 当前播放位置与缓冲区末尾的距离（秒）如果超过这个值，就触发一次追赶，不易过短
            decreaseDurationStep: 0.4,   // 每次追赶至缓冲区末尾之前的多少秒
            frozenTimesThreshold: 5,        // 解码停滞次数达到此阈值，上报VIDEO_FROZEN事件。注意如果设置过小的阈值，当推流端关闭摄像头后可能会频繁触发VIDEO_FROZEN事件
            // webrtc合流未能给cdn推送正确的视频分辨率信息，导致从MetaData/AVCDecoderConfigurationRecord中无法拿到正确的视频分辨率
            // 在内核低于Chromium 70的浏览器（如360浏览器、搜狗浏览器、PC微信内嵌浏览器等）中
            // 如果传递不对的视频分辨率，将会导致画面放大显示异常，因此，强行设置一个最大的视频宽高信息，来避免此问题
            // 如果是Safari浏览器，enableConstVideoViewSize建议设置为false
            enableConstVideoViewSize: true, 
            // constVideoViewWidth: 1920,
            // constVideoViewHeight: 1080,
          })
          this.player.attachMediaElement(videoElement)
          this.player.load() // 加载
          this.player.play() // 播放
          this.player.znvVideoSrcType = 'flv'
          this.player.znvVideoPlayerType = 'flvjs'
        }
      })
    },
    // pc & mobile
    getDeviceKind () {
      if (/(iPhone|iPad|iPod|iOS|Android)/i.test(navigator.userAgent)) {
        return 'mobile'
      } else {
        return 'pc'
      }
    },
    // 修改是否静音
    setMuted (_muted) {
      if (this.player.znvVideoPlayerType === 'videoJs') {
        this.player.muted(_muted)
      } else if (this.player.znvVideoPlayerType === 'flvjs') {
        this.player.muted = _muted
      } else if (this.player.znvVideoPlayerType === 'EZUIPlayer') {
        this.player.video.muted = _muted
      }  else if (this.player.znvVideoPlayerType === 'streamedian') {
        this.player.player.muted = _muted
      }
    },
    // 全屏
    setFullscreen () {
      if (this.player.znvVideoPlayerType === 'videoJs') {
        if (this.type === 'hls') {
          document.getElementById(this.vId + '_html5_api').requestFullscreen()
        } else {
          this.player.el().requestFullscreen()
        }
      } else if (this.player.znvVideoPlayerType === 'flvjs') {
        this.player._mediaElement.requestFullscreen()
      } else if (this.player.znvVideoPlayerType === 'EZUIPlayer') {
        if (this.player.znvVideoSrcType === 'hls') {
          this.player.video.requestFullscreen()
        } else if (this.player.znvVideoSrcType === 'rtmp') {
          this.player.fullScreen()
        }
      } else if (this.player.znvVideoPlayerType === 'streamedian') {
        this.player.player.requestFullscreen()
      }
    },
    // 获取当前帧
    getCurrentFrame() {
      if (this.type === 'rtmp') {
        console.log('rtmp暂时无法截图')
        return false
      }
      return this.savePic(+new Date() + '', this.vId)
    },
    savePic(fileName, videoDomId) {
      let fileType = 'png' // 如果文件名中没有带后缀，默认使用png
      switch (
        fileName // 判断保存的图片格式
      ) {
        case fileName.indexOf('png') > -1:
          fileType = 'png'
          break
        case fileName.indexOf('jpg') > -1:
          fileType = 'jpg'
          break

        case fileName.indexOf('jpeg') > -1:
          fileType = 'jpeg'
          break
        case fileName.indexOf('bmp') > -1:
          fileType = 'bmp'
          break
        case fileName.indexOf('gif') > -1:
          fileType = 'gif'
          break
        default:
          fileType = 'png'
          break
      }
      let video = document.querySelector('#' + videoDomId) // 找到需要截图的video标签
      let canvas = (window.canvas = document.createElement('canvas'))
      canvas.width = video.videoWidth
      canvas.height = video.videoHeight
      canvas
        .getContext('2d')
        .drawImage(video, 0, 0, canvas.width, canvas.height) // 图片大小和视频分辨率一致
      let strDataURL = canvas.toDataURL('image/' + fileType) // canvas中video中取一帧图片并转成base64
      // base64转blob
      let arr = strDataURL.split(',')
      let mime = arr[0].match(/:(.*?);/)[1]
      let bstr = atob(arr[1])
      let n = bstr.length
      let u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      let blob = new Blob([u8arr], {
        type: mime
      })
      // blob转file
      let file = new window.File([blob], fileName + '.' + fileType, { type: mime })
      return file
      // 图片下载
      // let url = window.URL.createObjectURL(blob)
      // var img = document.createElement('img')
      // img.src = url
      // img.className = 'image'
      // document.querySelector('#myPlayer').after(img)
      // let a = document.createElement('a')
      // a.style.display = 'none'
      // a.href = url
      // a.download = fileName
      // document.body.appendChild(a)
      // a.click()
      // setTimeout(function () {
      //   document.body.removeChild(a)
      //   window.URL.revokeObjectURL(url)
      // }, 1000)
    },
    // 销毁HLS视频流以及对应播放器实例
    destroyHls () {
      if (this.player && this.playerType === 'videoJs') {
        this.player.dispose()
      } else {
        this.player.stop()
        $('#' + this.vId).attr('src', '')
      }
      this.player = null
    },
    // 销毁RTSP视频流以及对应播放器实例
    destroyRtsp () {
      if (this.player) {
        this.player.destroy()
        this.player = null
      }
    },
    // 销毁RTMP视频流以及对应播放器实例
    destroyRtmp () {
      if (this.player && this.playerType === 'videoJs') {
        this.player.dispose()
      } else {
        this.player.stop()
        $('#' + this.vId).replaceWith(`<video id="${this.vId}" class="video" autoplay controls muted></video>`)
      }
      this.player = null
    },
    // 销毁flv视频流以及对应播放器实例
    destroyFlv () {
      if (this.player) {
        this.player.pause()
        this.player.unload()
        this.player.detachMediaElement()
        this.player.destroy()
        this.player = null
      }
    }
  },
  destroyed () {
    if (this.type === 'hls') {
      this.destroyHls()
    } else if (this.type === 'rtsp') {
      this.destroyRtsp()
    } else if (this.type === 'rtmp') {
      this.destroyRtmp()
    }
  }
}
</script>

<style scoped lang="scss">
.znv-video{
  position: relative;
  min-width: 100px;
  min-height: 100px;
  width: 100%;
  height: 100%;
  overflow: hidden;

  .hls-container{
    width: 100%;
    height: 100%;
  }
  .small-window-rtmp {
    width: 200%;
    height: 200%;
    transform: scale(.5);
    transform-origin: left top;
  }
  video{
    width: 100%;
    height: 100%;
    object-fit: fill;
  }
  .opt-box{
    position: absolute;
    width: 100%;
    height: 20%;
    left: 0;
    top: 0;
    background-image: linear-gradient(rgba(0,0,0,.9), rgba(0,0,0,.2));
    padding: 0 5%;
  }
}
.fade-leave-active {
  transition: all .8s ease;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transform: translateY(-100%);
}
</style>
