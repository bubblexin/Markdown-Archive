

[TOC]
### 引入 SDK

1. 在 Android Studio 中添加依赖，以 aar 的形式导入。(X.X.X 替换成对应版本号 比如 1.0.1)

```java
api 'com.okay.client_app:okay_lib_live:X.X.X' 
```
2. 添加 AndroidManifest 中需要的权限
```java
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.GET_TASKS"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
    <uses-permission android:name="android.permission.READ_LOGS"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

### 直播流程示例

![pic](http://p3pvnofd3.bkt.clouddn.com/18-6-19/85970877.jpg)

### 接入直播

#### 初始化

| 接口名 | 接口描述                                                     |
| ------ | ------------------------------------------------------------ |
| init   | 直播初始化，所有行为的第一步，告知身份 appid 信息(需要在 Activity 中初始化) |

| 参数类型 | 说明 |
| -------- | ---- |
| Context context | 初始化 SDK 的上下文（建议使用ApplicationnContext） |
| int appid | 传入业务方 appid |
| int accountType | 传入业务方 accountType |

**示例**：

```java
// 初始化OkLiveSdk
ITencentLiveManager.getInstance().init(context, appid ,accoutype);
```



#### 账号登录

| 方法名 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| login  | 使用独立模式，在获取到用户的 sig 后，使用登录接口，告知后台音视频模块上线了（包括 avsdk） |

| 参数类型                              | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| String id                             | 用户 id，在直播过程中的唯一标识                              |
| String sig                            | 鉴权的密钥 Sig，如果是独立登录方式，是业务方后台计算生成后下发 |
| TencentOkLiveCallback                 | 账号登录回调接口，通知上线是否成功                           |
| ILiveLoginManager.TILVBStatusListener | 用户状态回调接口                                             |
#### 账号登出
| 方法名 | 方法描述 |
| ------ | -------- |
| logout | 登出     |

| 参数类型              | 说明                               |
| --------------------- | ---------------------------------- |
| TencentOkLiveCallback | 账号登出回调接口，通知登出是否成功 |

**示例**：

```java
ITencentLiveManager.getInstance().login(String id, String sig, ILiveCallBack callBack, ILiveLoginManager.TILVBStatusListener statusListener)
```



#### 判断是否是登录状态

| 方法名  | 方法描述         |
| ------- | ---------------- |
| isLogin | 返回当前登录状态 |

**示例**：

```java
ITencentLiveManager.getInstance().isLogin();
```



### 房间相关

#### 创建房间/加入房间

| 方法名      | 方法描述                                                     |
| ----------- | ------------------------------------------------------------ |
| joinChannel | 根据传入的角色，创建/加入一个直播，只有在初始化和登录成功之后才能创建直播 |

| 参数类型              | 说明                 |
| --------------------- | -------------------- |
| String id             | 学生 id              |
| int roomid            | 房间 ID 房间唯一标识 |
| String role           | 创建房间的结果回调   |
| TencentOkLiveCallback | 加入/创建房间回调    |

**示例**：

```java
ITencentLiveManager.getInstance().joinChannel(String id, int roomId, String role, TencentOkLiveCallback callBack)
```



#### 退出房间

| 方法名       | 方法描述 |
| ------------ | -------- |
| leaveChannel | 退出房间 |

| 参数类型              | 说明               |
| --------------------- | ------------------ |
| TencentOkLiveCallback | 退出房间的结果回调 |

**示例**：

```java
// 退出房间
ITencentLiveManager.getInstance().leaveChannel(TencentOkLiveCallback callback)
```



#### 获取通话质量

| 方法名 | 方法描述 |
| ---|---|
| getQualityData | 获取通话质量(主线程调用) |

**示例**：

```java
ILiveQualityData qualityData = ITencentLiveManager.getInstance().getQualityData();
if(qualityData != null) {
      JSONObject jo = new JSONObject();
      try {
          jo.put("start_time", qualityData.getStartTime());
          jo.put("end_time", qualityData.getEndTime());
          jo.put("app_cpu_rate", qualityData.getAppCPURate());
          jo.put("sys_app_rate", qualityData.getSysCPURate());
          // 接收丢包率
          jo.put("recv_loss_rate", qualityData.getRecvLossRate());
          // 发送丢包率
          jo.put("send_loss_rate", qualityData.getSendLossRate());
          jo.put("recp_kbps", qualityData.getRecvKbps());
          jo.put("send_kbps", qualityData.getSendKbps());
          jo.put("up_fps", qualityData.getUpFPS());
      } catch (JSONException e) {
          e.printStackTrace();
      }
}
```



#### 切换角色

| 方法名     | 方法描述     |
| ---------- | ------------ |
| changeRole | 修改房间角色 |

| 参数类型              | 说明           |
| --------------------- | -------------- |
| String role           | 要切换到的角色 |
| TencentOkLiveCallback | 回调           |

**示例**：

```java
ITencentLiveManager.getInstance().changeRole(String role, TencentOkLiveCallback callBack）
```



#### 设置界面渲染

#### [~~手动渲染~~](https://cloud.tencent.com/document/product/647/17432)

建议在进入房间内之前设置界面渲染。

| 方法名         | 方法描述      |
| -------------- | ------------- |
| setAvVideoView | 设置渲染 View |

| 参数类型   | 说明      |
| ---------- | --------- |
| AVRootView | 渲染 View |

#### 资源相关方法(在 Activity 对应事件中调用)

| 方法名    | 方法描述 |
| --------- | -------- |
| onPause   | 暂停直播 |
| onResume  | 恢复直播 |
| onDestroy | 销毁直播 |



### 直播互动


#### 开启、关闭屏幕分享

| 方法名        | 方法描述     |
| ------------- | ------------ |
| enableScreen  | 开启屏幕分享 |
| disableScreen | 关闭屏幕分享 |

| 参数类型            | 说明              |
| ------------------- | ----------------- |
| int mode            | 模式              |
| boolean vertical    | 是否竖屏（false） |
| TencentLiveCallback | 回调              |

**示例**：

```java
// 开启屏幕分享
ITencentLiveManager.getInstance().enableScreen(int mode, false, TencentLiveCallback callBack)
...
//关闭屏幕分享
ITencentLiveManager.getInstance().disableScreen(TencentLiveCallback callBack)
```



#### 上麦、下麦

| 方法名     | 方法描述 |
| ---------- | -------- |
| enableMic  | 控制上麦 |
| disableMic | 控制下麦 |

| 参数类型            | 说明                 |
| ------------------- | -------------------- |
| TencentLiveCallback | 控制上下麦结果的回调 |

**示例**：

```java
// 上麦
ITencentLiveManager.getInstance().enableMic(TencentLiveCallback callback)
...
// 下麦
ITencentLiveManager.getInstance().disableMic(TencentLiveCallback callback)
```



### 可选方法

#### 配置日志类

> 端上可以选择获取库里打印的相关日志，通过 LiveLog 类设置实现 LogListener 接口，***需要在加入房间之前设置***

| 方法名         | 描述         |
| -------------- | ------------ |
| setLogListener | 设置日志监听 |

```java
// e.g
LiveLog.INSTANCE.setLogListener(new LiveLog.LogListener() {
            @Override
            public void d(@Nullable String tag, @Nullable String msg) {
                
            }

            @Override
            public void v(@Nullable String tag, @Nullable String msg) {

            }

            @Override
            public void i(@Nullable String tag, @Nullable String msg) {

            }

            @Override
            public void w(@Nullable String tag, @Nullable String msg) {

            }

            @Override
            public void e(@Nullable String tag, @Nullable String msg) {

            }
        });
```



#### 配置 RoomOption

>  默认可以不设置，加入房间的时候库里会设置一个默认的属性，若要自己设置，具体参数信息见 ILVLiveRoomOption 类，请在  joinChannel 之前配置好相关配置信息

| 方法名            | 描述             |
| ----------------- | ---------------- |
| setChannelProfile | 设置房间配置属性 |

| 参数类型                                    | 说明             |
| ------------------------------------------- | ---------------- |
| ILVLiveRoomOption optionn                   | 腾讯房间配置信息 |
| OkLiveRoomOption option（设置为 null 即可） | OkLiveRoomOption |

#### 获取/生成 房间创建者/加入房间者 的配置信息

| 方法名                          | 描述                             |
| ------------------------------- | -------------------------------- |
| generateDefaultMasterRoomOption | 生成/获取房间创建者的 RoomOption |

| 参数类型            | 说明    |
| ------------------- | ------- |
| String id           | 主播 id |
| String role         | 角色    |
| TencentLiveCallback | 回调    |

| 方法名                          | 描述                             |
| ------------------------------- | -------------------------------- |
| generateDefaultMemberRoomOption | 生成/获取房间加入这的 RoomOption |

| 参数类型            | 说明 |
| ------------------- | ---- |
| String role         | 角色 |
| TencentLiveCallback | 回调 |



### 保留方法

以下方法为保留方法，暂未实现，忽略即可

| 方法名               | 描述 |
| -------------------- | ---- |
| enableCamera         |      |
| disableCamera        |      |
| switchCamera         |      |
| enableAudio          |      |
| disableAudio         |      |
| enableVideo          |      |
| disableVideo         |      |
| setAudioConfigration |      |
| setVideoConfigration |      |

### 废弃方法

#### ~~修改权限（废弃）~~

用于核心机房和旁路机房通道之间的切换，之前与腾讯技术沟通，暂不支持切换，所以此方法暂时无用。

| 方法名              | 方法描述  |
| ------------------- | --------- |
| changeAuthorityToOc | 切换到 OC |
| changeAuthorityToDc | 切换到 DC |

| 参数类型         | 说明 |
| ---------------- | ---- |
| ITencentCallback | 回调 |

**示例**：

```java
// ChangeToOc
TencentLiveManager.getInstance(mContext).changeAuthorityToOc(new TencentCallback() {
            @Override
            public void onSuccess() {
                ...
            }

            @Override
            protected void onFailed(String code, String message) {
                ...
            }
        });
// ChangeToDc
TencentLiveManager.getInstance(mContext).changeAuthorityToDc(new TencentCallback() {
            @Override
            public void onSuccess() {
                ...
            }

            @Override
            protected void onFailed(String code, String message) {
                ...
            }
        });
```



### Ps：

* 所有回调接口，请使用 **TencentOkLiveCallback** 来实现

### 常见问题

#### [腾讯互动直播 SDK 官方文档](https://cloud.tencent.com/document/product/268/7658)

#### [~~声网互动直播 SDK 官方文档~~](https://docs.agora.io/cn/2.0/product/Interactive%20Broadcast/API%20Reference/live_video_android?platform=Android)

#### 错误码表

[直播项目错误码](http://wiki.okjiaoyu.cn/pages/viewpage.action?pageId=22366143)

若上述位置没有找到对应错误码，在[此位置](https://cloud.tencent.com/document/product/268/8423)寻找

#### 编译错误 - so 文件冲突

如若出现 libstlport_shared.so 冲突，这是 Xlog 中和直播 SDK 中引入的 so 库冲突，需要在「项目 gradle」 的 android 下添加如下代码：

```java
packagingOptions {
     pickFirst '**/libstlport_shared.so'
} 
```
#### Jenkins 打包错误

在项目的 build.gradle.low 中加入如下代码，1.8.5.3.2 是用到的腾讯直播库版本号

```java
import com.android.build.gradle.internal.pipeline.TransformTask

def deleteDuplicateJniFiles() {
    def files = fileTree("${buildDir}/intermediates/exploded-aar/com.tencent.ilivesdk/ilivesdk/1.8.5.3.2/jni/") {
        include "**/libstlport_shared.so"
    }
    files.each { it.delete() }
}

tasks.withType(TransformTask) { pkgTask ->
    pkgTask.doFirst { deleteDuplicateJniFiles() }
}
```

