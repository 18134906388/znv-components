<template>
  <div id="app">
    <div class="video1" v-for="item in items">
      <znv-video :type="item.type" :src="item.src"></znv-video>
    </div>
  </div>
</template>

<script>
// hls点播测试地址https://yunqivedio.alicdn.com/2017yq/v2/0x0/96d79d3f5400514a6883869399708e11/96d79d3f5400514a6883869399708e11.m3u8
// hls直播测试地址https://hls01open.ys7.com/openlive/f01018a141094b7fa138b9d0b856507b.m3u8
// flv点播测试地址https://mister-ben.github.io/videojs-flvjs/bbb.flv
// flv直播测试地址https://flvopen.ys7.com:9188/openlive/376d82d20a6c479b829222acca274b61.hd.flv
import ZnvVideo from "./components/znvVideo/Video";
export default {
  name: "app",
  components: { ZnvVideo },
  data() {
    return {
      items: [
        {
          type: "vcms",
          src: "14020000001310000041"
        }
      ]
    };
  },
  mounted() {}
};
</script>

<style>
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
  display: flex;
}
.video1 {
  width: 500px;
  height: 600px;
}
</style>
