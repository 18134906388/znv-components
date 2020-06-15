<template>
  <div id="app">
    <div class="video-box" v-for="(item, index) in videoList" :key="index">
      <znv-video :src="item.src" :type="item.type" :vId="item.vId"></znv-video>
    </div>
  </div>
</template>

<script>
import ZnvVideo from './components/znvVideo/ZnvVideo'
export default {
  name: 'app',
  components: { ZnvVideo },
  data () {
    return {
      videoList: [{
        src: 'http://10.45.157.180:88/live?app=znv&stream=live',
        type: 'flv',
        vId: 'test1'
      }, {
        src: 'http://10.45.157.180:88/live?app=znv&stream=live',
        type: 'flv',
        vId: 'test2'
      }, {
        src: 'http://10.45.157.180:88/live?app=znv&stream=live',
        type: 'flv',
        vId: 'test3'
      }, {
        src: 'rtmp://10.45.157.180/znv/live',
        type: 'rtmp',
        vId: 'test4'
      }]
    }
  }
}
</script>

<style lang="scss">
#app {
  .video-box {
    width: 500px;
    height: 500px;
    display: inline-block;
  }
}
</style>
