<template>
  <div id="app">
    <div class="video1">
      <znv-video
        :type="item.type"
        :hikLayer="item.hikLayer"
        :cameraIndexCodes="item.cameraIndexCodes"
      ></znv-video>
    </div>
  </div>
</template>

<script>
// hls点播测试地址https://yunqivedio.alicdn.com/2017yq/v2/0x0/96d79d3f5400514a6883869399708e11/96d79d3f5400514a6883869399708e11.m3u8
// hls直播测试地址https://hls01open.ys7.com/openlive/f01018a141094b7fa138b9d0b856507b.m3u8
// flv点播测试地址https://mister-ben.github.io/videojs-flvjs/bbb.flv
// flv直播测试地址https://flvopen.ys7.com:9188/openlive/376d82d20a6c479b829222acca274b61.hd.flv
import ZnvVideo from "./components/znvVideo/Video";
import st from "supper-tools";
export default {
  name: "app",
  components: { ZnvVideo },
  data() {
    return {
      item: {
        type: "hik",
        hikLayer: '2x2',
        cameraIndexCodes:
          "37035100001310666505,37035100001310666505,37035100001310666505,37035100001310666505"
      }
    };
  },
  mounted() {
    setTimeout(() => {
      this.item.cameraIndexCodes = '1,2'
    }, 3000);
    // let queryObj = st.url2json(location.href)
    // queryObj.src && (this.item.src = unescape(queryObj.src))
    // queryObj.type && (this.item.type = queryObj.type)
    // queryObj.playerType && (this.item.playerType = queryObj.playerType)
    // queryObj.ws && (this.item.ws = unescape(queryObj.ws))
  }
};
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
  font-size: 10px;
  font-family: "Microsoft YaHei";
  width: 100%;
  height: 100vh;
  color: #e5edff;
}
#app {
  width: 100%;
  height: 100%;
  .video1 {
    width: 500px;
    height: 600px;
  }
}
</style>
