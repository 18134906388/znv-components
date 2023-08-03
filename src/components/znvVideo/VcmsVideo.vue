<template>
  <div class="box-video">
    <video :id="vId" class="znv-video" muted autoplay controls></video>
  </div>
</template>

<script>
export default {
  name: "VcmsVideo",
  data() {
    return {};
  },
  props: {
    vId: {
      type: String,
      required: true,
      default: ""
    },
    src: {
      type: String,
      required: true,
      default: ""
    }
  },
  watch: {
    src() {
      this.dispose();
      this.initVideo();
    }
  },
  mounted() {
    this.initVideo();
  },
  methods: {
    initVideo() {
      if (Streamedian && this.src) {
        this.playRtsp(0);
      }
    },
    initRtsp() {},
    playRtsp(streamType) {
      let self = this;
      if (this.src) {
        // $.ajax({
        //   url: '/apis/rtsp/queryUrl?cameraId=' + this.src + '&streamType=' + streamType,
        //   type: 'GET',
        //   headers: {
        //     Authorization: '12CBD9B708D887A41AFAB97DAC46AAC6'
        //   },
        //   success: function (res) {
        //     let data = res.data[0]
        //     self.streamMark = data.streamMark
        //     let opt = {
        //       src: data.rtspUrl,
        //       socket: data.wsUrl,
        //       redirectNativeMediaErrors: true,
        //       bufferDuration: 10,
        //       statuHandler: self.stuHandler,
        //       isH265: true,
        //       isH265Url: data.wsUrl,
        //       streamMark: data.streamMark
        //     }
        //     var videoElement = document.getElementById(this.vId)
        //     var rtspPlayer = Streamedian.player(videoElement, opt)
        //     self.rtspPlayer = rtspPlayer
        //   },
        //   error: function () {
        //     console.log('H265参数请求失败')
        //   }
        // })
        let data = {
          rtspUrl:
            "rtsp://10.72.76.233:5542/live/14020000001310000041/0?A=94564befcc59e44b42f045d1d1e102d3",
          rtmpUrl:
            "rtmp://10.72.76.233:1935/live/14020000001310000041/0?A=94564befcc59e44b42f045d1d1e102d3",
          wsUrl:
            "ws://10.72.76.233:8070/live/14020000001310000041/0.live.mp4?A=94564befcc59e44b42f045d1d1e102d3",
          hlsUrl:
            "http://10.72.76.233:8070/live/14020000001310000041/0/hls.m3u8?A=94564befcc59e44b42f045d1d1e102d3",
          httpUrl:
            "http://10.72.76.233:8070/live/14020000001310000041/0.live.mp4?A=94564befcc59e44b42f045d1d1e102d3",
          flvUrl:
            "http://10.72.76.233:8070/live/14020000001310000041/0.flv?A=94564befcc59e44b42f045d1d1e102d3",
          srcUrl: "rtsp://10.72.76.233:555/live/14020000001110000002-1/0",
          port: "8070",
          ip: "10.72.76.233",
          streamMark:
            "pXDpho0vo5MQFdXXu3+f/74qdd+XbSFe5zWIwO/tGtB+ccFLfvtEEo6SdtwBvpPY9fU6R+IBJu1YK14PxkxoNCaNXFo=",
          extendsParams: {}
        };
        this.streamMark = data.streamMark;
        let opt = {
          src: data.rtspUrl,
          socket: data.wsUrl,
          redirectNativeMediaErrors: true,
          bufferDuration: 10,
          statuHandler: this.stuHandler,
          isH265: true,
          isH265Url: data.wsUrl,
          streamMark: data.streamMark
        };
        var videoElement = document.getElementById(this.vId);
        videoElement.src = data.rtspUrl;
        var rtspPlayer = Streamedian.player(videoElement, opt);
        this.rtspPlayer = rtspPlayer;
      }
    },
    stuHandler(currentProxy, message) {
      if (message === 9000001) {
        this.rtspMessage = "正在请求视频";
      } else if (message === 9000002) {
        this.rtspMessage = "请求视频失败";
      } else if (message === 9000003) {
        this.rtspMessage = "正在重新拉取视频";
      } else if (message === 9000004) {
        this.rtspMessage = "正在重连视频";
      } else {
        this.rtspMessage = message.toFixed(2) + "kbps";
      }
      console.log(this.rtspMessage);
    },
    destoryRtspStream() {
      if (this.streamMark) {
        $.ajax({
          url:
            "apis/rtsp/closeStream?streamMark=" +
            encodeURIComponent(this.streamMark),
          type: "GET",
          headers: {
            Authorization: "12CBD9B708D887A41AFAB97DAC46AAC6"
          },
          success: function(res) {},
          error: function() {
            console.log("销毁rtsp失败");
          }
        });
      }
    },
    dispose() {
      if (this.rtspPlayer) {
        this.destoryRtspStream();
        this.rtspPlayer.destroy();
        this.rtspPlayer = null;
      }
    }
  },
  beforeDestroy() {
    this.dispose();
  }
};
</script>

<style>
@import url(./viedo.css);
</style>
