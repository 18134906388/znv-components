<template>
  <div id="app">
    <znv-video :src="item.src" :type="item.type" :ref="item.vId"></znv-video>
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
        src: 'https://flvopen.ys7.com:9188/openlive/c447724d034446ce8f40f02a661d8eaf.flv',
        type: 'flv'
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
