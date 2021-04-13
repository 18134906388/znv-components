<template>
  <div id="app">
    <znv-video v-if='item.src' :src="item.src" :type="item.type" :playerType='item.playerType' :ws='item.ws'></znv-video>
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
        src: '',
        type: '',
        playerType: 'videoJs',
        ws: ''
      }
    }
  },
  mounted() {
    let queryObj = st.url2json(location.href)
    queryObj.src && (this.item.src = unescape(queryObj.src))
    queryObj.type && (this.item.type = queryObj.type)
    queryObj.playerType && (this.item.playerType = queryObj.playerType)
    queryObj.ws && (this.item.ws = unescape(queryObj.ws))
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
