## 前提条件

1. 目前WebRTC 支持的浏览器，如下表所示： MgRtcPlayer web SDK  推荐使用最新版本的 Chrome 浏览器。

   | 平台         | Chrome 58+ | Firefox 56+ | Safari 11+ | Opera 45+ | QQ 浏览器 10.5+ | 360 安全浏览器 | 微信浏览器 |
   | ------------ | ---------- | ----------- | ---------- | --------- | --------------- | -------------- | ---------- |
   | Android 4.1+ | ✔          | ✘           | **N/A**    | ✘         | ✘               | ✘              | ✔       |
   | iOS 13+      | ✘          | ✘           | ✔          | ✘         | ✘               | ✘              | ✔         |
   | macOS 10+    | ✔          | ✔           | ✔          | ✔         | ✔               | ✘              | ✘          |
   | Windows 7+   | ✔          | ✔           | **N/A**    | ✔         | ✔               | ✔              | ✘          |

2. 在支持的浏览器及内核之上，大部分机型浏览器中支持H264编码，然而在使用Webview时目前还是建议选择VP8编码

## 设置开发环境


你需要准备一个自己的项目并且将 MgRtcPlayer web SDK 集成到其中。

### 创建 Web 项目


这个项目只需要用到一个 HTML 文件。
[查看示例代码](https://github.com/gffsolely/MG-WebRTC-demo)

1. 新建一个 HTML 文件。这里我们将文件命名为 index.html（以下简称项目文件）。
2. 用一个代码编辑器（例如 VS Code）打开该文件。
3. 你可以自定义你的 UI。
4. 项目中需要使用一些第三方的库
- [adapter.js](https://github.com/Temasys/AdapterJS)
- [jsencrypt.min.js](https://npmcdn.com/jsencrypt@2.3.1/bin/jsencrypt.js)




### SDK版本列表

集成时请尽量使用最新版本

版本 | 说明 | 备注
---|---|---
v0.0.1 | 测试版本，实现基础功能 | 不支持https协议
v0.1.2 | 测试版本，优化了部分兼容性 | 不支持https协议，ios微信内置、UC等部分浏览器暂不支持
v0.1.3 | 测试版本，增加了试玩关键指标值 | 不支持https协议，ios微信内置、UC等部分浏览器暂不支持
v0.1.6 | 测试版本，对WebRTC主要服务更新升级，增加对https支持 | ios12及以下微信内置、UC等部分浏览器暂不支持
v0.1.7 | 内测版本，优化投屏启动速度 | 兼容性同上一版本


### 集成 SDK

#### 方法 1. 使用 CDN 方法获取 SDK

该方法无需下载安装包。在项目文件中，将以下代码添加到页面中，尽量为 `<body>` 底部：

```javascript
<script src="//sw-cdn.shunwanyun.com/webrtc/jssdk/mg-rtc-player-0.1.7.js"></script>
```

现在，我们已经将 MgRtcPlayer web SDK 集成到项目中了。接下来我们要通过调用 MgRtcPlayer web SDK 提供的 API 来实现功能。

## 实现试玩功能

本节介绍如何使用 MgRtcPlayer web SDK 实现试玩功能。

实现试玩功能前准备：

- 申请试玩接口、appkey、webrtc roomServer等地址（sdk当前只需要appkey）
- 通过瞬玩的运营后台部署试玩游戏与试玩设备


SDK中主要对象说明：

- mgRtcPlayer  sdk入口对象（核心）
- MgUtil  公共工具方法库（如getQueryString、sendUrlRequest、parseJSON等）
- $ 获取dom对象， 实际为document.querySelector(selector);
- mgRtcPlayer.appController  操作控制对象，如发送手机按键（home、back等）
- mgRtcPlayer.appWebRTC  WebRTC控制对象

### 初始化客户端

为方便起见，我们为下面要用到的代码定义了三个变量。此步骤不是必须的，你可以根据你的项目有其他的实现。
```javascript
   let playElemId="video_box", // 试玩画面所在控件id
   playRoomId="123456",  //webrtc房间标识
   appId =123; //试玩游戏id（取自在瞬玩运营后台安装到设备的appId）
 ```

开始试玩前，我们需要先创建并初始化mgRtcPlayer客户端对象。
```javascript
   let playElemId="video_box",playRoomId="123456", appId =123;
   mgRtcPlayer.init({
    base:{isDebug:true},
    paasConfig: {
      playType: 2, 
      playValue: appId,
      palyAppKey: 'EkQc4lN3MqBFgf9y2y8hsoDX', 
    },
    rtcConfig:{ 
        roomId: playRoomId,
        elementId:playElemId,},
    videoConfig: {
      videoConf: { vCodec: "H264", vBitrate: 4000, vLevel: 1, mute: false },
    },
    events: {
      onStart: function () {
        console.log('mg html onStart' );
      },
      onRunInfo: function (netDelay,fps,jitterDelay,recBitrate,w,h){
        //网络延迟,fps,编码&传送延迟,网络速率,视频宽,视频高
      },
      onStop: function () {
        console.log('mg html onStop');
      },
      onError: function (type, code, msg) {
        console.log("mg html onError,type:" + type + ",code:" + code + ",msg:" + msg );
      }
    }
  });
 ```
 在以上初始化过程中用到的对象或方法说明：

- init 初始化入口

### base
- base 基础配置
- base.isDebug 是否调试模式 取值 true|false，取值true时会打印执行log方便分析，上线时设置为false（不设置默认为false）

#### paasConfig
- paasConfig 对游戏调度所需的接口配置
- paasConfig.playType 取值说明：
    - 0:使用控制服配置对象，直连设备， playType取值:0 时playValue取值控制服配置对象（**控制服配置对象请联系商务获取**）
    - 2:使用appid试玩 playType取值:2 时playValue取值:playAppId
    - 3:使用app包名试玩 playType取值:3 时playValue取值:playAppPackageName
- paasConfig.palyAppKey 瞬玩运营平台中分配给对应帐号的appkey（**appkey请联系商务获取**），用于资源调度

#### rtcConfig
- rtcConfig WebRTC参数配置
- rtcConfig.roomId   webrtc房间标识，建议每次投屏都换一个值（不设置时 默认取系统随机值）
- rtcConfig.elementId 试玩画面所在控件id

#### videoConfig
- videoConfig 试玩画面质量配置 （注意：参数配置优先级 mgRtcPlayer.setVideoConf方法 > 服务端参数配置 > init.videoConfig.videoConf ）
- videoConfig.videoConf.vCodec 设置默认编码，根据浏览器支持范围（注意：目前webview时必须设置为VP8）
- videoConfig.videoConf.vBitrate 码率 单位kb
- videoConfig.videoConf.vLevel 分辨率,取值 : 
  -  1: 720 X 1280
  -  2: 576 X 1024
  -  3: 432 X 768
  -  4: 288 X 512
- videoConfig.videoConf.mute 是否静音

#### events
- events 回调事件合集
- events.onStart 试玩播放开始
- events.onStop 试玩停止
- events.onError 试玩过程中异常 参数说明：
  - type 错误类型
  - code 错误标识号（详情请见 **错误标识**）
  - msg 错误描述
- events.onRunInfo 试玩过程中关键指标值（**部分值在某些浏览器中无法获取**）
  - netDelay     网络延迟
  - fps          帧率
  - jitterDelay  编码&传送延迟
  - recBitrate   网络速率（Bps）
  - w            视频宽
  - h            视频高

### 开始试玩

初始化完成之后我们就可以通过mgRtcPlayer开启试玩
```javascript
 mgRtcPlayer.start(); 
 
 //也可以与初始一起链式执行，如下
 //mgRtcPlayer.init({...}).start(); 
```
### 设置试玩参数

 参数说明请查看 **videoConfig**

```javascript
 mgRtcPlayer.setVideoConf({ vCodec: v_vcodec, vBitrate: v_bitrate, vLevel: v_level, mute: v_mute });
```
**注意:** 试玩参数 只有在 mgRtcPlayer.init() 之后并且 mgRtcPlayer.start()之前设置才能生效

### 设置 开启声音

```javascript
/**
    * 开启声音
    * 大多数浏览器默认必须静音条件下 video 才可自动播放
    * 必须在当前页面有用户交互才可开启声音
    */
mgRtcPlayer.enableVoice();
```
方法可接收一个参数 `isMuted`（是否静音）
```javascript
//设置为静音
mgRtcPlayer.enableVoice(true);

//开启声音
mgRtcPlayer.enableVoice(false);
mgRtcPlayer.enableVoice();  //无参数时默认是开启声音
 
```
 

有用户交互 举例：可以在当前页面添加一个开始按钮，点击按钮事件中调用,如下
```javascript
btn.click(()=>{
 mgRtcPlayer.init({...
    events: {
      onStart: function () {
        mgRtcPlayer.enableVoice();
        console.log('mg html onStart' );
      },}
    ...}).start(); 
});
 
```


### 结束试玩

```javascript
 mgRtcPlayer.stop(); 
```


### 错误标识

    ERROR_APPCONTROLLER_CATCH:200101,  //控制服请求异常
    ERROR_APPCONTROLLER_PARAM: 200102,  //控制服参数错误
    ERROR_APPCONTROLLER_SC:200103,  // 控制服错误
    ERROR_PAASAPI_BASE:200200,  // 控制服基础错误
    ERROR_PAASAPI_DATA_NULL: 200201,  //paas api data is null
    ERROR_APPWEBRTC_BASE:200300, // WebRTC基础错误
    ERROR_APPWEBRTC_CALL_HANGUP:200301,  // 服务端断开
    ERROR_APPWEBRTC_CALL_TIMEOUT:200302  // 呼叫超时
    
    其他标识：
    196612：设备掉线
    196613：被其他人踢掉
    196614：空闲时间过长(2分钟内无任何操作)
    196626：控制时间不足
    196643：APP异常
    196646：管理员踢出
    
### 上线部署

#### 静态页面部署  

通过Web服务静态部署，如nginx、apache、iis

使用nginx部署请参考 [nginx搭建静态资源web服务器](https://yq.aliyun.com/articles/758471?spm=a2c4e.11153940.0.0.26eb59ce3VF5D3)

**注意:** sdk 版本v0.1.1及以下版本暂不支持https协议部署，请先使用http协议，我们会尽快在之后的版本中支持https协议
