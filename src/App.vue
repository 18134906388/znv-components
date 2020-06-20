<template>
  <div id="app">
    <div class="video-box" v-for="(item, index) in videoList" :key="index">
      <znv-video :src="item.src" :type="item.type" :vId="item.vId" :playerType='item.playerType'></znv-video>
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
        src: 'rtmp://58.200.131.2:1935/livetv/hunantv',
        type: 'rtmp',
        vId: 'test4',
        playerType: 'EZUIPlayer'
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
