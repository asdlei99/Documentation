## 一、集成指南

### 适用范围

本集成文档适用于 Android ARRtmax SDK 3.0.0 + 版本。

### 准备环境

- Android Studio 2.1 或以上版本
- Android 版本不低于 4.0.3 且支持音视频的 Android 设备（不支持模拟器）
- Android 设备已经连接到有效网络

### 导入SDK

**Gradle方式导入**[ ![Download](https://api.bintray.com/packages/dyncanyrtc/ar_dev/rtmax/images/download.svg) ](https://bintray.com/dyncanyrtc/ar_dev/rtmax/_latestVersion)


```
dependencies {
  implementation 'org.ar:rtmax_kit:3.1.2'
}
```


---

## 二、开发指南

集成SDK后，还需对SDK进行初始化操作，建议在Application中完成。

#### 1.1 初始化SDK并配置开发者信息

调用 initEngine() 方法配置开发者信息，开发者信息可在anyRTC管理后台中获得，详见[创建anyRTC账号](https://docs.anyrtc.io)


**示例代码：**

```
public class ARApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        ARMaxEngine.Inst().initEngine(getApplicationContext(), false , "AppId",  "AppToken");
    }
}

```

> 自定义的Application需在AndroidManifest.xml注册 

#### 1.2 获取ARMax配置类并设置相关配置

**示例代码：**

``` 
//获取ARMax配置类
ARMaxOption arMaxOption =ARMaxEngine.Inst().getArMaxOption();
arMaxOption.setVideoProfile(ARVideoCommon.ARVideoProfile.ARVideoProfile480x640);
//更多配置查看API文档

```
#### 1.3 实例化ARMaxKit对象并设置回调

**示例代码：**

``` 
1- ARMaxKit mRTMaxKit = new ARMaxKit(ARMaxEvent maxEvent);

2- ARMaxKit mRTMaxKit = new ARMaxKit(ARMaxEvent maxEvent,true);//是否需要检测网络类型
```
#### 1.4 实例化视频显示View

**示例代码：**

``` 
ARVideoView videoView = new ARVideoView(rl_video,  ARRtcpEngine.Inst().Egl(),this,false,mRTMaxKit);

arVideoView.setVideoViewLayout(false, Gravity.CENTER,LinearLayout.HORIZONTAL);

```
> ARVideoView 对象是显示视频，调整视频窗口摆放位置的类，可由开发者自定义，具体可参照Demo

#### 1.5 加入对讲组

**示例代码：**

``` 
mRTMaxKit.joinTalkGroup( String groupId,  String userId,  String userData);

```
> 加入对讲组成功会回调onRTCJoinTalkGroupOK()方法，失败回调onRTCJoinTalkGroupFailed()

#### 1.6 申请对讲

**示例代码：**

``` 
mRTMaxKit.applyTalk(int nPriority)

```
> nPriority是申请抢麦用户的级别，数值越大，权限越小。返回值等于0的时候表示调用成功。成功后会回调onRTCApplyTalkOk()，表示语音通道已建立，可以开始说话了。

#### 1.7 取消对讲

**示例代码：**

``` 
mRTMaxKit.cancelTalk()

```
> 调用取消对讲后将会回调 onRTCApplyTalkClosed()方法


#### 1.6 发起/停止监看

**示例代码：**

``` 
//发起监看
mRTMaxKit.monitorVideo()
//停止监看
mRTMaxKit.closeVideoMonitor()

```
> 发起监看后对方将收到onRTCVideoMonitorRequest()回调，在该回调中可以同意或者拒绝监看，参见 1.7。 停止监看后对方将收到onRTCVideoMonitorClose()回调。

#### 1.7 同意/拒绝监看

**示例代码：**

``` 
//同意监看
mRTMaxKit.acceptVideoMonitor()
//拒绝监看
mRTMaxKit.rejectVideoMonitor()

```
> 同意或者拒绝对方都将收到onRTCVideoMonitorResult()回调。通常，在同意监看后应该开启本地摄像头，对方停止监看后应当关闭本地摄像头。参见 1.8

#### 1.8 打开/关闭本地摄像头

**示例代码：**

``` 
打开本地摄像头采集
mRTMaxKit.setLocalVideoCapturer(arVideoView.openLocalVideoRender().GetRenderPointer());

关闭摄像头采集 移除本地像
arVideoView.removeLocalVideoRender();
mRTMaxKit.closeLocalVideoCapture();

```

#### 1.9 打开/关闭视频上报

**示例代码：**

``` 
打开视频上报
mRTMaxKit.reportVideo()
关闭视频上报
mRTMaxKit.closeReportVideo();
//关闭上报时应将本地摄像头关闭
arVideoView.removeLocalVideoRender();
mRTMaxKit.closeLocalVideoCapture();
```

> - 打开视频上报后应及时打开本地摄像头，对方将收到onRTCVideoReportRequest()回调，在onRTCVideoReportRequest()回调中，**可以调用发起监看方法同意上报视频mRTMaxKit.monitorVideo()**，同意后将会回调 onRTCOpenRemoteVideoRender()方法，此时应在此方法中设置显示对方视频。参见 2.0。
> - 关闭视频上报后对方将收到OnRtcCloseVideoRender()回调，在此方法中应移除上报视频图像 参见 2.1 **关闭上报时应将本地摄像头关闭**


#### 2.0 对方视频即将显示

**示例代码：**

```
//显示对方视频
 mRTMaxKit.setRTCRemoteVideoRender(publishId, arVideoView.openRemoteVideoRender(publishId).GetRenderPointer());

```
> 双方视频通道建立后将会回调onRTCOpenRemoteVideoRender方法，在该回调中应显示对方视频，参照上述代码，具体可查看demo

#### 2.1  对方视频即将关闭

**示例代码：**

```
//移除对方视频
if (null != mRTMaxKit) {
//移除
mRTMaxKit.setRTCRemoteVideoRender(publishId, 0);
arVideoView.removeRemoteRender(publishId);
}

```
> 双方视频通道关闭会回调onRTCCloseRemoteVideoRender()方法，在该回调中应移除对方视频，参照上述代码，具体可查看demo

#### 2.2 呼叫其他人

**示例代码：**

```
//主动呼叫
mRTMaxKit.makeCall(String userId,  int type/*0:视频 1:音频*/,  String userData);

```
> 返回值为0时呼叫成功，被呼叫人会收到onRTCMakeCall()回调


#### 2.3 取消呼叫他人/挂断

**示例代码：**

```
//取消呼叫
mRTMaxKit.endCall(userId);
//主动呼叫他人 挂断
mRTMaxKit.endCall(userId);
//被呼叫 挂断
mRTMaxKit.leaveCall();

```
> 主叫端挂断，对方将收到onRTCEndCall()，被叫端挂断，对方将收到onRTCLeaveCall()
，在这俩回调中应做将本地摄像头关闭等操作，同时还将收到onRTCCloseRemoteVideoRender(),此时还应移除对方视频， 具体查看demo

#### 2.4 拒绝他人呼叫

**示例代码：**

```
mRTMaxKit.rejectCall(callId);
}

```
> 对方将收到onRTCRejectCall()回调


#### 2.5 接受他人呼叫

**示例代码：**

```
mRTMaxKit.acceptCall(callId);

```
> 接受呼叫后，双方呼叫通道开启，将回调onRTCOpenRemoteVideoRender()方法，在该回调中设置显示对方视频，参考 2.0,接受后对方还将回调onRTCAcceptCall()，此时还应将本地摄像头打开，具体参照demo

#### 2.6 权限说明

使用ARRtmax SDK需以下权限

```
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```
#### 2.7 混淆配置
在Proguard混淆文件中增加以下配置：

```
-dontwarn org.anyrtc.**
-keep class org.anyrtc.**{*;}
-dontwarn org.ar.**
-keep class org.ar.**{*;}
-dontwarn org.webrtc.**
-keep class org.webrtc.**{*;}
```


---

## 三、API接口文档

### ARMaxEngine 类

### 1. 初始化并配置开发者信息

**定义**

```
void initEngine(Context context, boolean useJaveRecord，String appId, String token)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
context | Context | 上下文对象
useJaveRecord | boolean | 是否使用Java层采集声音
appId | String | appId
token | String  | token

**说明**

该方法为配置开发者信息，上述参数均可在https://www.anyrtc.io/ 应用管理中获得；建议在Application调用。

### 2. 配置私有云

**定义**

```
void configServerForPriCloud(String address,int port)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
address | String | 私有云服务地址
port | int | 私有云服务端口

**说明**

配置私有云信息，当使用私有云时才需要进行配置，默认无需配置。

### 3. 获取SDK版本号

**定义**

```
String getSdkVersion()
```
**返回值**

SDK版本号

### 4. 关闭硬解码(安卓特有)

**定义**

```
void disableHWDecode()
```

### 5. 关闭硬编码(安卓特有)

**定义**

```
void disableHWEncode()
```

### 6. 设置日志显示级别

**定义**

```
void setLogLevel(ARLogLevel logLevel) 
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
logLevel | ARLogLevel | 日志显示级别
---

### ARMaxOption 配置类

### 1. 获取配置类

**定义**

```
ARMaxOption option = ARMaxEngine.Inst().getArMaxOption()
```

### 2. 设置可配置参数

**定义**
```
void setOptionParams(boolean isDefaultFrontCamera, ARVideoCommon.ARVideoOrientation mScreenOriention, ARVideoCommon.ARVideoProfile videoProfile,ARVideoCommon.ARVideoFrameRate videoFps) 
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
isDefaultFrontCamera | boolean | 是否默认前置摄像头 true 前置 false 后置  默认true
videoOrientation | ARVideoOrientation |视频方向 默认竖直
videoProfile | ARVideoProfile | 视频分辨率  默认360x640
videoFps | ARVideoFrameRate |视频帧率  默认 Fps15
**说明**  

可通过上面方法配置，也可单独设置


--- 

### ARMaxKit 类

### 1.实例化ARMaxKit对象

**定义** 

```
ARMaxKit arMaxKit = new ARMaxKit(ARMaxEvent maxEvent)

或

ARMaxKit arMaxKit = new ARMaxKit(ARMaxEvent maxEvent，boolean bCheckNetType)

```
**参数**

参数名 | 类型 | 描述
---|:---:|---
maxEvent | ARMaxEvent | ARMaxEvent回调实现类
bCheckNetType | boolean | 是否需要监测网络类型
### 2. 加入对讲组

**定义**

```
int joinTalkGroup(String groupId,  String userId,  String userData)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | String | 对讲组id（同一个anyrtc平台的appid内保持唯一性）
userId | String | 开发者自己平台的用户Id
userData | String | 开发者自己平台的相关信息（昵称，头像等），可选


**返回值**

0：成功


### 3. 切换对讲组

**定义**

```
int switchTalkGroup(String groupId,  String userData)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | String | 对讲组id（同一个anyrtc平台的appid内保持唯一性）
userData | String | 开发者自己平台的相关信息（昵称，头像等），可选


**返回值**

0：成功


### 4. 申请对讲

**定义**

```
int applyTalk( int priority) 
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
nPriority | int | 优先级

**说明**  

优先级：申请抢麦用户的级别（0权限最大（数值越大，权限越小）；除0外，可以后台设置0-10之间的抢麦权限大小）


**返回值**

0: 调用OK  -1:未登录  -2:正在对讲中  -3: 资源还在释放中 -4: 操作太过频繁


### 5. 取消对讲

**定义**

```
void cancelTalk()
```
**说明**   

申请对讲成功（onRTCApplyTalkOk）之后方可结束对讲 


### 6. 发起呼叫

**定义**

```
int makeCall( String userId,  int type,  String userData)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 被叫用户userid
type | int | 0:视频 1:音频
userData | String |自定义数据

**返回值**

0:调用OK;-1:未登录;-2:没有通话-3:视频资源占用中;-5:本操作不支持自己对自己;-6:会话未创建（没有被呼叫用户）


### 7. 呼叫邀请

**定义**

```
int inviteCall( String userId,  String userData)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 被邀请用户userid
userData | String |自定义数据

**返回值**

0:调用OK;-1:未登录;-2:没有通话-3:视频资源占用中;-5:本操作不支持自己对自己;-6:会话未创建（没有被呼叫用户）


### 8. 主叫端结束某一路正在进行的通话

**定义**

```
void endCall( String userId)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 指定用户userid

### 9. 接受呼叫或通话邀请

**定义**

```
int acceptCall( String callId)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | String | 呼叫请求时收到的callId


### 10.拒绝呼叫

**定义**

```
void rejectCall()
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | String | 呼叫请求时收到的callId

### 11.退出当前通话

**定义**

```
void leaveCall()
```


### 12.发起视频监看（或者收到视频上报请求时查看视频）

**定义**

```
int monitorVideo( String userId,  String userData) 
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 被监看用户userId
userData | String |自定义数据

**返回值**

0: 调用OK  -1:未登录	-5:本操作不支持自己对自己


### 13.同意视频监看

**定义**

```
int acceptVideoMonitor( String hostId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
hostId | String | 发起人的userid

**返回值**

0: 调用OK  -1:未登录 -3:视频资源占用中 -5:本操作不支持自己对自己

### 14.拒绝视频监看

**定义**

```
 int rejectVideoMonitor( String hostId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
hostId | String | 发起人的userid

**返回值**

0: 调用OK  -1:未登录 -3:视频资源占用中 -5:本操作不支持自己对自己

### 15.监看发起者关闭视频监看

**定义**

```
void closeVideoMonitor( String userId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 被监看用户的userid

### 16.视频上报

**定义**

```
int reportVideo( String userId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 要向谁上报的用户userid 

**说明**  

接收端收到上报请求时，调用monitorVideo进行视频查看


### 17.上报者关闭视频上报

**定义**

```
void closeReportVideo()
```


### 18. 发送消息

**定义**

```
int sendMessage( String userId,  String head,  String strContent)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userName | String | 用户昵称(最大256字节)，不能为空，否则发送失败；
head | String | 用户头像(最大512字节)，可选；
content | String | 消息内容(最大1024字节)不能为空，否则发送失败；


**返回值**

0 成功


### 19. 设置视频竖屏

**定义**

```
void setScreenToPortrait()
```

### 20. 设置视频横屏

**定义**

```
void setScreenToLandscape()
```

### 21. 设置本地音频是否传输

**定义**

```
void setLocalAudioEnable(boolean enabled)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | boolean| 打开或关闭本地音频传输

**说明**

true为传输音频，false为不传输音频，默认传输

### 22. 设置本地视频是否传输

**定义**

```
void setLocalVideoEnable(boolean enabled)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | boolean| 打开或关闭本地视频传输

**说明**

true为传输视频，false为不传输视频，默认视频传输

### 23. 设置本地视频采集窗口

**定义**

```
int setLocalVideoCapturer(long render)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
render | long | 底层视频渲染对象


**返回值**

0/1/2：没有相机权限/打开相机成功/打开相机失败


### 24. 停止本地视频采集

**定义**

```
 void closeLocalVideoCapture()
```

### 25.设置其他人视频窗口

**定义**

```
void setRTCRemoteVideoRender(String publishId,long render)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
publishId | String | RTC服务生成的视频标识Id 
 render|long|SDK底层视频显示对象 

**说明**  

该方法用于视频呼叫接通后，回调（OnRTCOpenRemoteVideoRender）使用

### 26. 切换前后摄像头

**定义**

```
void switchCamera()
```



### 27. 关闭P2P通话

**定义**

```
void closeP2PTalk()
```
**说明**   

在控制台强插对讲后，关闭和控制台之间的P2P通话


### 28. 设置录音文件的路径

**定义**

```
int setRecordPath( String callPath,  String talkPath,  String talkP2PPath)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
callPath | String | 呼叫文件的保存路径（文件夹路径）
talkPath | String | 对讲文件保存路径（文件夹路径）
talkP2PPath | String | 强插P2P文件保存路径（文件夹路径）

**返回值**

0/1:设置成功/文件夹不存在


### 29. 设置本地前置摄像头镜像是否打开

**定义**

```
void setFrontCameraMirrorEnable(boolean bEnable)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | boolean | true为打开，alse为关闭 



### 30. 设置视频网络状态是否打开

**定义**

```
void setNetworkStatus(boolean enable)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
enable | boolean | true打开，false关闭

**说明**

默认视频网络状态关闭

### 31. 获取当前视频网络状态是否打开

**定义**

```
boolean networkStatusEnabled() 
```

**返回值**

视频网络状态检测打开与否

### 32.是否打开回音消除

**定义**

```
void setForceAecEnable( boolean enable)
```

**返回值**

需在加入对讲组之前设置

### 33. 设置token验证

**定义**

```
boolean setUserToken(String userToken) 
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userToken | String | token字符串:客户端向自己服务器申请

**说明**

设置token验证必须放在joinGroup之前


### 34. 不接收某路视频

**定义**

```
 void muteRemoteVideoStream(String publishId,boolean mute)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
mute | boolean | true禁止，false接收
publishId | String | RTC服务生成的通道Id 


### 35. 不接收某路音频

**定义**

```
void muteRemoteAudioStream(String publishId,  boolean mute)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
mute | boolean | true禁止，false接收
publishId | String | 流ID

### 36. 设置远程（其他人）音视频状态

**定义**

```
void setRemoteAVEnable( String peerId,  boolean audioEnable,  boolean videoEnable)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | 用户的ID
audioEnable | boolean | true 打开 false 关闭
videoEnable | boolean | true 打开 false 关闭

### 37. 打开关闭摄像头闪光灯

**定义**

```
void openCameraTorchMode( boolean open)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
open | boolean | 是否开启闪光灯

### 38. 是否打开音频检测

**定义**

```
void setAudioActiveCheck( boolean audioOnly, final boolean audioDetect)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
audioOnly | boolean |  true:仅音频模式; false: 音视频模式
audioDetect | boolean |  true:打开; false: 关闭

### 39. 退出对讲组

**定义**

```
void leaveTalkGroup()
```

### 40. 销毁对象

**定义**

```
void clear()
```

### 41. 设置单次上麦最上时间

**定义**

```
void setTalkLimitTime(int time)
```
**参数**

参数名 | 类型 | 描述
---|:---:|---
time | int |  时间（秒）

### 42.群内视频上报

**定义**

```
void reportGroupVideo()
```
**说明**

群内视频上报，同一组内在线设备都可以收到上报请求（接收端收到上报请求时，调用monitorVideo进行视频查看）


### 43.应用内视频上报

**定义**

```
void reportAppVideo()
```
**说明**

应用内视频上报，同一应用id内在线设备都可以收到上报请求（接收端收到上报请求时，调用monitorVideo进行视频查看）

### 44.设置仅对讲模式

**定义**

```
void setOnlyTalkMode(boolean enable)
```

### 45.打开或关闭音频数据回调

**定义**

```
void setAudioNeedPcm(final boolean bEnable) 
```

--- 

### ARMaxEvent 回调类

### 1. 加入对讲组成功

**定义**

```
void onRTCJoinTalkGroupOK(String groupId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | String | 群组Id  

### 2. 加入对讲组失败

**定义**

```
void onRTCJoinTalkGroupFailed(String groupId, int code, String reason)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
groupId | String | 群组Id  
code | int | 错误码
reason | String | 错误原因

### 3. 离开对讲组

**定义**

```
void onRTCLeaveTalkGroup(int code)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | int | 错误码

**说明**  

0：正常退出；100：网络错误，与服务器断开连接；207：强制退出

### 4. 申请对讲成功

**定义**

```
void onRTCApplyTalkOk()
```

### 5. 语音通道建立成功

**定义**

```
void onRTCTalkCouldSpeak()
```
**说明**  

申请对讲成功后，语音通道建立成功回调，可以开始讲话

### 6. 其他人正在对讲组中讲话

**定义**

```
void onRTCTalkOn(String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据


### 7. 当用户处于对讲状态时，控制台强制发起P2P通话

**定义**

```
void onRTCTalkP2POn(String userId, String userData);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 8. 与控制台的P2P讲话结束

**定义**

```
void onRTCTalkP2POff(String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userData | String | 用户的自定义数据

### 9.结束对讲回调

**定义**

```
void onRTCTalkClosed(int code, String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | int | 错误码
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 10.视频监看请求

**定义**

```
 void onRTCVideoMonitorRequest(String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据

**说明**  

有用户向本机用户发起视频监看请求

### 11.视频监看关闭

**定义**

```
 void onRTCVideoMonitorClose(String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 12.视频监看请求结果

**定义**

```
void onRTCVideoMonitorResult(String userId, int code, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
code | int | 错误码
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 13.收到视频上报请求

**定义**

```
 void onRTCVideoReportRequest(String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 14.视频上报关闭

**定义**

```
 void onRTCVideoReportClose(String userId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid

### 15.主叫方发起通话成功

**定义**

```
void onRTCMakeCallOK(String callId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | String | 呼叫成功时服务器生成的CallId

### 16.主叫方收到被叫方同意通话

**定义**

```
void onRTCAcceptCall(String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据

### 17.主叫方收到被叫方拒绝通话

**定义**

```
void onRTCRejectCall(String userId, int code, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid
userData | String | 用户的自定义数据


### 18.被叫方挂断或者被邀请方离开通话

**定义**

```
void onRTCLeaveCall(String userId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid

### 19.通话释放

**定义**

```
void onRTCLeaveCall(String userId)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
userId | String | 用户的userid

**说明**  

主叫方收到通话结束的回调（被叫方和被邀请方已全部退出或者主叫方挂断所有参与者）


### 20.被叫方收到通话请求

**定义**

```
void onRTCMakeCall(String callId, int nCallType, String userId, String userData)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | String | 呼叫成功时服务器生成的CallId   
nCallType | int | 呼叫类型（0：视频；1：音频）
userId | String | 用户的userid
userData | String | 用户的自定义数据


### 21.被叫方收到主叫方挂断通话

**定义**

```
void onRTCEndCall(String callId, String userId, int code)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
callId | String | 呼叫成功时服务器生成的CallId   
userId | String | 用户的userid
code | int | 状态码


### 22.视频呼叫接通视频窗口打开

**定义**

```
void onRTCOpenRemoteVideoRender(String peerId, String publishId, String userId, String userData);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | RTC服务生成的标识Id
publishId | String | RTC服务生成的视频通道Id
userId | String | 开发者自己平台的用户Id
userData | String | 开发者自己平台的相关信息（昵称，头像等）

**说明**  

其他通话参与者进入视频通话的回调，开发者需调用设置其他与会者视频窗口（setRemoteVideoRender）方法。

### 23. 视频窗口关闭

**定义**

```
void onRTCCloseRemoteVideoRender(String peerId, String publishId, String userId);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | RTC服务生成的标识Id 
publishId | String | RTC服务生成的视频通道Id
userId | String | 开发者自己平台的用户Id

**说明**  

其他对讲者离开的回调，开发者需移除该抢麦者视频窗口，仅视频呼叫时有用


### 24. 音频模式下呼叫打开音频通道回调

**定义**

```
void onRtcOpenRemoteAudioTrack(String peerId, String userId, String userData);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | RTC服务生成的标识Id (用于标识用户，每次加入对讲组随机生成)
userId | String | 开发者自己平台的用户Id
userData | String | 开发者自己平台的相关信息（昵称，头像等）

**说明**  

**音频模式**下接通 呼叫后回调

### 25. 音频模式下呼叫关闭音频通道回调

**定义**

```
void onRtcCloseRemoteAudioTrack(String peerId, String userId);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String |RTC服务生成的标识Id (用于标识用户，每次加入对讲组随机生成)
userId | String | 开发者自己平台的用户Id

**说明**  

**音频模式**关闭呼叫回调


### 26. 其他人对音视频的操作通知

**定义**

```
void onRTCRemoteAVStatus(String peerId, boolean audio, boolean video);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | RTC服务生成的标识Id (用于标识用户，每次加入对讲组随机生成)
audio | boolean | 音频状态 true 打开 false 关闭
video | boolean | 视频状态 true 打开 false 关闭

**说明**  

与会者关闭/开启了音视频

### 27. 其他人对自己音视频的操作通知

**定义**

```
void onRTCLocalAVStatus(boolean audio, boolean video);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
audio | boolean | 音频状态 true 打开 false 关闭
video | boolean | 视频状态 true 打开 false 关闭

**说明**  

别人对自己音视频的操作

### 28. 音频监测（其他人声音大小回调）

**定义**

```
void onRTCRemoteAudioActive(String peerId, String userId, int level, int time);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
peerId | String | RTC服务生成的标识Id (用于标识用户，每次加入对讲组随机生成)
userId | String | 开发者自己平台的用户Id
level | int | 音量大小
time | int | time时间内不会再回调该方法(毫秒)

### 29. 音频监测（本地声音大小回调）

**定义**

```
void onRTLocalAudioActive(int level, int time);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
level | int | 音量大小
time | int | time时间内不会再回调该方法(毫秒)

### 30. 本地网络状态

**定义**

```
onRTCLocalNetworkStatus(int netSpeed, int packetLost, ARNetQuality netQuality);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
nNetSpeed | int |网络上行
nPacketLost | int | 丢包率(1~100)
netQuality | ARNetQuality | 网络质量

### 31. 远程（其他人）网络状态

**定义**

```
onRTCRemoteNetworkStatus(String rtcpId, int netSpeed, int packetLost, ARNetQuality netQuality);
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
rtcpId |String | 通道Id
nNetSpeed | int |网络上行
nPacketLost | int | 丢包率(1~100)
netQuality | ARNetQuality | 网络质量

### 32. 当前对讲组在线人数

**定义**

```
void onRTCMemberNum(int num)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
num |int | 人数


### 33. 录像地址回调信息

**定义**

```
 void onRTCGotRecordFile(int nRecType, String userData, String filePath)
```

**参数**

参数名 | 类型 | 描述
---|:---:|---
nRecType |int | 录音的类型（0/1/2/3：对讲本地录音/对讲远端录音/强插P2P录音/语音通话呼叫录音）
userData |String | 对讲录音所有者的自定义数据 
filePath |String | 录音文件保存路径  

**说明**  

设置录音路径后录音保存路径的回调接口

### 34. 网络差

**定义**

```
 boolean OnRtcCheckNetSignalIsBad()
```

**说明**  

网络变成2G 3G的时候回调

### 35. 加入语音对讲组成功

**定义**

```
 void OnRtcJoinMaxGroupOk(String groupId)
```
### 36. 临时断开语音对讲组

**定义**

```
 void OnRtcTempLeaveMaxGroup(int nCode)
```
### 37. 加入语音对讲组失败

**定义**

```
 void OnRtcJoinMaxGroupFailed(String strGroupId, int nCode, String strReason)
```
### 38. 离开语音对讲组

**定义**

```
 void OnRtcLeaveMaxGroup(int nCode)
```

### 39. 本地原始音频数据

**定义**

```
 void onRTCLocalAudioPcmData(String peerId, byte[] data, int nLen, int nSampleHz, int nChannel);
```

### 40. 远程用户原始音频数据

**定义**

```
 void onRTCRemoteAudioPcmData(String peerId, byte[] data, int nLen, int nSampleHz, int nChannel);
```

### 四、更新日志

##### V 3.1.2 （2020-03-22）

优化弱网环境语音传输

新增方法：  
new ARMaxKit(arMaxEvent,true)
void setAudioNeedPcm( boolean bEnable)
int setTalkLimitTime( int nLength)
int reportGroupVideo()
int reportAppVideo()
void setOnlyTalkMode(boolean enable)

新增回调：
 boolean OnRtcCheckNetSignalIsBad()
 void OnRtcJoinMaxGroupOk(String groupId)
 void OnRtcTempLeaveMaxGroup(int nCode)
 void OnRtcJoinMaxGroupFailed(String strGroupId, int nCode, String strReason)
 void OnRtcLeaveMaxGroup(int nCode)
 void onRTCLocalAudioPcmData(String peerId, byte[] data, int nLen, int nSampleHz, int nChannel);
 void onRTCRemoteAudioPcmData(String peerId, byte[] data, int nLen, int nSampleHz, int nChannel);

##### V 3.0.0 （2019-05-15）

- SDK版本升级3.0，API接口变更，更加简洁规范
- 更改SDK配置项，配置更简单
- 断网重连优化
- 分辨率选项增加，帧率可配置
- 添加用户服务级token安全认证，服务更安全

