<template>
  <div id="app">
    <znv-video :src="item.src" :type="item.type" :ws="item.ws" :ref="item.vId"></znv-video>
  </div>
</template>

<script>
import ZnvVideo from './components/znvVideo/ZnvVideo'
import st from 'supper-tools'
export default {
  name: 'app',
  components: { ZnvVideo },
  data () {
    return {
      item: {
        src: 'rtsp://10.72.55.30:5542/live/61000000001310000004/0',
        type: 'rtsp',
        ws: 'ws://10.72.55.30:8060/ws'
      }
    }
  }
}
</script>

<style lang="scss">
html,body{
  margin: 0;
  padding: 0;
  font-size: 10px;
  // overflow: hidden;
  font-family: "Microsoft YaHei";
  width: 100%;
  height: 100%;
  color: #E5EDFF;
}
#app {
  width: 100%;
  height: 100%;
  .video-box {
    width: 100%;
    height: 100%;
    display: inline-block;
  }
}
</style>
