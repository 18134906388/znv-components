# znv-video

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

```
import ZnvVideo from 'znv-components'
Vue.use(ZnvVideo)
```

webworkify es6-promise 这两个模块需要手动安装

1.0.2 修改 video-js.swf获取位置

1.0.3 新增flv 播放器优化

1.0.4 新增测试用例, flv实现自动延时追赶
打包和调试时 webpack.config.js 修改相应配置

1.0.7 对接VCMS RTSP视频播放

1.0.8 控制条优化
