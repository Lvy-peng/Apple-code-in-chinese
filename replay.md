ReplayKit (API 参考)
===
[原文地址](https://developer.apple.com/reference/replaykit?language=objc)

翻译人：pzm 翻译时间：2017/1/12

通过从 APP 中的屏幕或者音频和麦克风的录制或者说流化视频，人们可以通过 email,信息和社交媒体分享他们的经历。

###导览
ReplayKit 框架提供了通过屏幕录制视频和通过 App 和麦克风录制音频。用户可以在社交媒体上分享其他的人的录屏结果。而且你可以通过构建你的应用程序使得你的实时直播内容变成分享服务。 ReplayKit 并不兼容 AVplayer的内容。

###符号
-
|         |    处理剪辑媒体       |
| ------------- |:---------------------------------------:|
|               | RPBroadcastController  (一个包含启动和控制直播流的对象)|
|              | RPBroadcastHandle (向直播应用程序发送消息的对象) |
|               | RPBroadcastMP4ClipHandler (通过 ReplayKit处理 MP4视频对象 )|
|              |  RPBroadcastSampleHandler (处理从 ReplayKit 收到的 CMSampleBufferRef对象) |

-

| |  实现和配置一个在线直播|
|-----------|:--------------------------------:|
|           | RPBroadcastActivityViewController (提供给用户选择一个直播服务的接口) |
|           | RPBroadcastActivityViewCOntrollerDelegate (对直播的用户界面更改的响应，RPBroadcastActitibyViewController的代理)|
|           | RPBroadcastConfiguration (视频直播中进行视频剪辑)|
|           | RPBroadcastControllerDelegate (直播中状态改变的协议)|

| |  创建和分享一个回放|
|-----------|:--------------------------------:|
|           | RPPreviewViewController (允许用户预览和编辑通过 ReplayKit 创建的一个屏幕录像) |
|           |RPPreviewViewControllerDeleagte (RPPreviewViewController 的代理，返回屏幕录制中用户界面的改变)|
|           | RPScreenRecorder (录制音频和视频)|
|           | RPScreenRecorderDeleagte (收到 RPScreenRecorder 对象的广播协议)|

| |  引用|
|-----------|:--------------------------------:|
|           | ReplayKit 常量 |
|           | ReplayKit 枚举|

     
   
      
                  
            