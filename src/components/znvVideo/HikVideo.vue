<template>
  <div :id="vId" class="playWnd" style="left: 109px; top: 133px;"></div>
</template>

<script>
export default {
  name: "HikVideo",
  props: {
    vId: {
      type: String,
      required: true,
      default: ""
    },
    cameraIndexCode: {
      type: String,
      required: false,
      default: ""
    }
  },
  data() {
    return {
      oWebControl: null,
      initCount: 0,
      pubKey: ""
    };
  },
  mounted() {
    this.$nextTick(() => {
      this.initPlugin();
    })
  },
  methods: {
    initPlugin() {
      let self = this;
      self.oWebControl = new WebControl({
        szPluginContainer: self.vId, // 指定容器id
        iServicePortStart: 15900, // 指定起止端口号，建议使用该值
        iServicePortEnd: 15909,
        szClassId: "23BF3B0A-2C56-4D97-9C03-0CB103AA8F11", // 用于IE10使用ActiveX的clsid
        cbConnectSuccess: function() {
          // 创建WebControl实例成功
          self.oWebControl
            .JS_StartService("window", {
              // WebControl实例创建成功后需要启动服务
              dllPath: "./VideoPluginConnect.dll" // 值"./VideoPluginConnect.dll"写死
            })
            .then(
              function() {
                // 启动插件服务成功
                self.oWebControl.JS_SetWindowControlCallback({
                  // 设置消息回调
                  cbIntegrationCallBack: self.cbIntegrationCallBack
                });

                self.oWebControl
                  .JS_CreateWnd(self.vId, 1000, 600)
                  .then(function() {
                    //JS_CreateWnd创建视频播放窗口，宽高可设定
                    self.init(); // 创建播放实例成功后初始化
                  });
              },
              function() {
                console.log("启动插件服务失败");
              }
            );
        },
        cbConnectError: function() {
          // 创建WebControl实例失败
          self.oWebControl = null;
          console.log("插件未启动，正在尝试启动，请稍候...");
          WebControl.JS_WakeUp("VideoWebPlugin://"); // 程序未启动时执行error函数，采用wakeup来启动程序
          self.initCount++;
          if (self.initCount < 3) {
            setTimeout(function() {
              initPlugin();
            }, 3000);
          } else {
            console.log("插件启动失败，请检查插件是否安装！");
          }
        },
        cbConnectClose: function(bNormalClose) {
          // 异常断开：bNormalClose = false
          // JS_Disconnect正常断开：bNormalClose = true
          console.log("cbConnectClose");
          self.oWebControl = null;
        }
      });
    },
    init() {
      let self = this;
      self.getPubKey(function() {
        ////////////////////////////////// 请自行修改以下变量值	////////////////////////////////////
        var appkey = "27884407"; //综合安防管理平台提供的appkey，必填
        var secret = self.setEncrypt("vo2IbnwLJTegEXJDSA9B"); //综合安防管理平台提供的secret，必填
        var ip = "172.26.78.212"; //综合安防管理平台IP地址，必填
        var playMode = 0; //初始播放模式：0-预览，1-回放
        var port = 443; //综合安防管理平台端口，若启用HTTPS协议，默认443
        var snapDir = "D:\\SnapDir"; //抓图存储路径
        var videoDir = "D:\\VideoDir"; //紧急录像或录像剪辑存储路径
        var layout = "1x1"; //playMode指定模式的布局
        var enableHTTPS = 1; //是否启用HTTPS协议与综合安防管理平台交互，这里总是填1
        var encryptedFields = "secret"; //加密字段，默认加密领域为secret
        var showToolbar = 1; //是否显示工具栏，0-不显示，非0-显示
        var showSmart = 1; //是否显示智能信息（如配置移动侦测后画面上的线框），0-不显示，非0-显示
        var buttonIDs =
          "0,16,256,257,258,259,260,512,513,514,515,516,517,768,769"; //自定义工具条按钮
        ////////////////////////////////// 请自行修改以上变量值	////////////////////////////////////

        self.oWebControl
          .JS_RequestInterface({
            funcName: "init",
            argument: JSON.stringify({
              appkey: appkey, //API网关提供的appkey
              secret: secret, //API网关提供的secret
              ip: ip, //API网关IP地址
              playMode: playMode, //播放模式（决定显示预览还是回放界面）
              port: port, //端口
              snapDir: snapDir, //抓图存储路径
              videoDir: videoDir, //紧急录像或录像剪辑存储路径
              layout: layout, //布局
              enableHTTPS: enableHTTPS, //是否启用HTTPS协议
              encryptedFields: encryptedFields, //加密字段
              showToolbar: showToolbar, //是否显示工具栏
              showSmart: showSmart, //是否显示智能信息
              buttonIDs: buttonIDs //自定义工具条按钮
            })
          })
          .then(function(oData) {
            self.oWebControl.JS_Resize(1000, 600); // 初始化后resize一次，规避firefox下首次显示窗口后插件窗口未与DIV窗口重合问题
            self.previewVideo();
          });
      });
    },
    //获取公钥
    getPubKey(callback) {
      let self = this;
      self.oWebControl
        .JS_RequestInterface({
          funcName: "getRSAPubKey",
          argument: JSON.stringify({
            keyLength: 1024
          })
        })
        .then(function(oData) {
          console.log(oData);
          if (oData.responseMsg.data) {
            self.pubKey = oData.responseMsg.data;
            callback();
          }
        });
    },
    //RSA加密
    setEncrypt(value) {
      let self = this;
      var encrypt = new JSEncrypt();
      encrypt.setPublicKey(self.pubKey);
      return encrypt.encrypt(value);
    },
    previewVideo() {
      let self = this;
      var cameraIndexCode = self.cameraIndexCode; //获取输入的监控点编号值，必填
      var streamMode = 0; //主子码流标识：0-主码流，1-子码流
      var transMode = 1; //传输协议：0-UDP，1-TCP
      var gpuMode = 0; //是否启用GPU硬解，0-不启用，1-启用
      var wndId = -1; //播放窗口序号（在2x2以上布局下可指定播放窗口）

      cameraIndexCode = cameraIndexCode.replace(/(^\s*)/g, "");
      cameraIndexCode = cameraIndexCode.replace(/(\s*$)/g, "");

      self.oWebControl.JS_RequestInterface({
        funcName: "startPreview",
        argument: JSON.stringify({
          cameraIndexCode: cameraIndexCode, //监控点编号
          streamMode: streamMode, //主子码流标识
          transMode: transMode, //传输协议
          gpuMode: gpuMode, //是否开启GPU硬解
          wndId: wndId //可指定播放窗口
        })
      });
    },
    // 推送消息
    cbIntegrationCallBack(oData) {
      console.log(JSON.stringify(oData.responseMsg));
    },
    stopAllPreview() {
      let self = this;
      self.oWebControl.JS_RequestInterface({
        funcName: "stopAllPreview"
      });
    },
    destoryVideo() {
      let self = this;
      if (self.oWebControl != null) {
        self.oWebControl.JS_HideWnd(); // 先让窗口隐藏，规避可能的插件窗口滞后于浏览器消失问题
        self.oWebControl.JS_Disconnect().then(
          function() {
            console.log("断开与插件服务连接成功");
          },
          function() {
            console.log("断开与插件服务连接失败");
          }
        );
      }
    }
  },
  beforeDestroy() {
    this.stopAllPreview();
    this.destoryVideo();
  }
};
</script>

<style lang="scss">
.playWnd {
  margin: 30px 0 0 400px;
  width: 1000px;
  height: 600px;
  border: 1px solid red;
}
</style>
