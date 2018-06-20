

[TOC]

### 官方文档及 Demo

[互动直播ILVB](https://cloud.tencent.com/product/ilvb#wiki)

[官方 Demo-随心播**iLiveSDK_Android_Suixinbo**](https://github.com/zhaoyang21cn/iLiveSDK_Android_Suixinbo)

[简单直播](https://github.com/zhaoyang21cn/iLiveSDK_Android_LiveDemo)



### 引入 SDK

在 Android Studio 中添加依赖，以 aar 的形式导入。(X.X.X 替换成对应版本号 比如 1.0.1)

```java
// 直播业务功能
api 'com.tencent.livesdk:livesdk:X.X.X' 
// 核心功能     
api 'com.tencent.ilivesdk:ilivesdk:X.X.X'
```

#### 添加混淆

```java
-keep class com.tencent.**{*;}
-dontwarn com.tencent.**

-keep class tencent.**{*;}
-dontwarn tencent.**

-keep class qalsdk.**{*;}
-dontwarn qalsdk.**
```



#### 配置 service 和 receiver

```java
<!--TLS Qal 一些服务 -->
        <service
            android:name="com.tencent.qalsdk.service.QalService"
            android:exported="false"
            android:process=":QALSERVICE"></service>

        <receiver
            android:name="com.tencent.qalsdk.QALBroadcastReceiver"
            android:exported="false">
            <intent-filter>
                <action android:name="com.tencent.qalsdk.broadcast.qal"/>
            </intent-filter>
        </receiver>

        <receiver
            android:name="com.tencent.qalsdk.core.NetConnInfoCenter"
            android:process=":QALSERVICE">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.TIME_SET"/>
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.TIMEZONE_CHANGED"/>
            </intent-filter>
        </receiver>
```

#### 配置权限

```java
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
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
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.BROADCAST_STICKY"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>

<uses-feature android:name="android.hardware.camera"/>
<uses-feature android:name="android.hardware.camera.autofocus"/>
```

#### 删除非 armeabi 架构 so

由于目前只支持 armeabi 和 armeabi-v7a 架构，如果工程(或依赖库)中有多架构，需要在 build.gradle 中添加以下配置（如果包含子工程子工程也要加）。

```java
android{
    defaultConfig{
        ndk{
            abiFilters 'armeabi', 'armeabi-v7a'
        }
    }
}
```



### 直播流程示例

![pic](http://mc.qcloudimg.com/static/img/e6632b362fbc90745505823b1dc295bd/image.png)

### 一些类的说明

```java
ILiveConstants 常量定义 (视频权限等等)
ILiveLoginManager 直播登录类 (通过 sig 独立登录，以托管方式登录、注册)
ILVLiveRoomOption 房间配置类 (加入房间、创建房间时的配置信息)
ILVLiveManager 直播接口类 (初始化、创建、加入房间、发消息、连麦等)
ILiveRoomManager 房间管理类 (开启、关闭屏幕分享、切换角色)

// ILVLiveManager 和 ILiveRoomManager 有一定的关联，在 ILVLiveManager 的实现类 LiveMgr 中，如创建房间、加入房间等操作实际都是调用的 ILiveRoomManager 中的方法。
@Override
public int init(ILVLiveConfig config) {
    //使用SDK消息过滤机制 否则用他们自己的
    if (config.getRoomMessageListener() == null)
        config.messageListener(this);
    liveConfig = config;

    ILiveLog.ki(TAG, "init", new ILiveLog.LogExts().put("version", getVersion()));
    // 初始化 ILVLiveManager 的同时，初始化了 ILiveRoomManager类
    return ILiveRoomManager.getInstance().init(config);
}
```

### 接入直播

#### 初始化

| 接口名      | 接口描述                                                     |
| ----------- | ------------------------------------------------------------ |
| **initSDK** | iLiveSDK 的部分类的预初始化，是所有行为的第一步，告知身份 appId |

| 参数类型 | 说明 |
| -------- | ---- |
| Conext | 初始化 SDK 的上下文（建议使用ApplicationnContext） |
| int appid | 传入业务方 appid |
| int accountType | 传入业务方 accountType |

**示例**：

```java
// init sdk
if (MsfSdkUtils.isMainProcess(activity)) {
    //iLiveSDK初始化
    ILiveSDK.getInstance().initSdk(activity, appId, accountType);

    //初始化直播场景
    ILVLiveConfig liveConfig = new ILVLiveConfig();
    ILVLiveManager.getInstance().init(liveConfig);
} else {
    //需要在UI线程初始化
    throw new RuntimeException();
}
```



#### 账号登录

| 接口名           | 接口描述                                                     |
| ---------------- | ------------------------------------------------------------ |
| **iLiveLogin**   | 使用托管方式或独立模式，在获取到用户的 sig 后，使用登录接口，告知后台音视频模块上线了（包括 avsdk） |


**方法**

| 参数类型      | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| String id     | 用户 id，在直播过程中的唯一标识                              |
| String sig    | 鉴权的密钥 Sig，如果是独立登录方式，是业务方后台计算生成后下发 |
| ILiveCallBack | 帐号登录回调接口。通知上线是否成功                           |

**示例**：

```java
// 登录
ILiveLoginManager.getInstance().iLiveLogin(id, sig, new ITencentCallback() {
        @Override
        public void onSuccess() {
            Toast.makeText(TestActivity.this, "login success !", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onFaild(final int eCode, final String eMsg) {
            Toast.makeText(TestActivity.this, "|login fail " + eCode + " " + eMsg, Toast.LENGTH_SHORT).show();
        }
});
```



#### 判断是否是登录状态

| 方法名  | 方法描述     |
| ------- | ------------ |
| isLogin | 当前登录状态 |

**示例**：

```java
ILiveLoginManager.getInstance().isLogin()
```



### 房间相关

#### 创建房间

Tencent 的直播接口类是 ILVLiveManager 类，此类中负责加入、退出或者是切换房间等操作，发送消息、信令，上下麦、设置界面渲染等方法的封装。此类在 `com.tencent.livesdk` 包下，对房间的配置，参照 ILVLiveRoomOption 中的配置进行设置。

| 方法名    | 方法描述                                             |
| --------- | ---------------------------------------------------- |
| creatRoom | 创建一个直播，只有在初始化和登录成功之后才能创建直播 |

| 参数类型          | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| int               | 房间 ID 房间唯一标识 建议由业务方后台统一分配                |
| ILVLiveRoomOption | 房间配置项 可以设置角色 权限 主播 ID 摄像头参数等 具体参考类 ILVLiveRoomOption |
| ILiveCallBack     | 创建房间回调接口。通知创建房间是否成功                       |

**示例**：

```java
//创建房间配置项
ILVLiveRoomOption hostOption = new ILVLiveRoomOption(id)
                .controlRole(role)// 限定加入房间的角色为老师
                .autoCamera(false)//自动开启摄像头
                .authBits(AVRoomMulti.AUTH_BITS_DEFAULT) // 权限设置
                .videoMode(ILiveConstants.VIDEOMODE_NORMAL)
                .videoRecvMode(AVRoomMulti.VIDEO_RECV_MODE_SEMI_AUTO_RECV_CAMERA_VIDEO)  // 是否开启半自动接收(开启后不需要手动渲染)
                .autoFocus(true)
                .imsupport(false) // im 支持
                .roomDisconnectListener(disconnectListener);
//创建房间
ILVLiveManager.getInstance().createRoom(roomId, hostOption, callBack);
```



#### 加入、退出房间

| 方法名   | 方法描述               |
| -------- | ---------------------- |
| joinRoom | 观众角色调用加入房间内 |

| 参数类型        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| int             | 房间 ID 房间唯一标识 建议由业务方后台统一分配                |
| ILiveRoomOption | 房间配置项 可以设置角色 权限 主播 ID 摄像头参数等 具体参考类 ILiveRoomOption |
| ILiveCallBack   | 加入房间回调接口。通知加入房间是否成功                       |


| 方法名   | 方法描述         |
| -------- | ---------------- |
| quitRoom | 角色调用退出房间 |

| 参数类型      | 说明               |
| ------------- | ------------------ |
| ILiveCallBack | 退出房间的结果回调 |

**示例**：

```java
long authBits;//权限设置

if (TencentLiveRole.ROLE_MASTER.equals(role)) {
    authBits = AVRoomMulti.AUTH_BITS_DEFAULT;//默认所有权限
} else {
    authBits = AVRoomMulti.AUTH_BITS_JOIN_ROOM | AVRoomMulti.AUTH_BITS_RECV_CAMERA_VIDEO //加入房间权限
            | AVRoomMulti.AUTH_BITS_RECV_AUDIO //接受语音权限
            | AVRoomMulti.AUTH_BITS_RECV_SCREEN_VIDEO;//接收屏幕视频权限
}
//加入房间配置项
ILVLiveRoomOption memberOption = new ILVLiveRoomOption("")
                .autoCamera(false) //是否自动打开摄像头
                .controlRole(role) //角色设置
                .authBits(authBits)
                .setRoomMemberStatusLisenter(this) // 房间内成员状态回调接口
                .imsupport(false)//目前业务不需要IM功能，默认关闭掉
                .videoMode(ILiveConstants.VIDEOMODE_BSUPPORT)//后台静默
                .screenRecvMode(CommonConstants.Const_AutoRecv_Screen)//是否开始半自动接收
                .autoMic(false)//默认关闭mic
                .autoRender(true)//自动渲染
                .videoRecvMode(AVRoomMulti.SCREEN_RECV_MODE_SEMI_AUTO_RECV_SCREEN_VIDEO)
                .roomDisconnectListener(disconnectListener);//是否自动打开mic
//加入房间，成功返回 ILiveConstants.NO_ERR
int ret = ILVLiveManager.getInstance().joinRoom(roomId, memberOption, callBack);

...
// 退出房间
ILVLiveManager.getInstance().quitRoom(new ILiveCallBack() {
            @Override
            public void onSuccess(Object data) {
                ...
            }

            @Override
            public void onError(String module, int errCode, String errMsg) {
			  ...
            }
        });
```



#### 获取通话质量

必须在房间建立好之后调用，并且仅限在主线程调用

| 方法名 | 方法描述 |
| ---|---|
| getQualityData | 获取质量数据（通话质量） |

**示例**：

```java
ILiveQualityData qualityData = ILiveRoomManager.getInstance().getQualityData();
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

 互动直播引入角色的概率，用于实现在单一平台上也可以配置不同的参数。可以将角色理解为终端进入房间的配置集。用户可以在腾讯云后台，配置角色、定制角色。

| 方法名     | 方法描述     |
| ---------- | ------------ |
| changeRole | 修改房间角色 |

| 参数类型         | 说明           |
| ---------------- | -------------- |
| String role      | 要切换到的角色 |
| ITencentCallback | 回调           |

**示例**：

```java
ITencentRoom mTencentRoom = new TencentRoomImpl(context);
mTencentRoom.changeRole(TencentLiveRole.ROLE_LIVEGUEST, new ITencentCallback() {
          @Override
          public void onSuccess() {
              
          }
    
          @Override
          public void onFaild(int code, String message) {

          }
});
```



#### 修改权限（暂时无用）

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



#### 设置界面渲染

建议在进入房间内之前设置界面渲染。

| 方法名         | 方法描述      |
| -------------- | ------------- |
| setAvVideoView | 设置渲染 View |

| 参数类型   | 说明      |
| ---------- | --------- |
| AVRootView | 渲染 View |



#### 资源相关方法(在Activity对应事件中调用)（暂时无用）

| 方法名    | 方法描述 |
| --------- | -------- |
| onPause   | 暂停直播 |
| onResume  | 恢复直播 |
| onDestroy | 销毁直播 |



### 设置渲染层

渲染层级示例图，在界面层 XML 插入一个 AVRootView，音视频数据最终是通过 AVRootView 渲染出来。考虑多屏互动情况，**AVRootView 实际上不是一层 View 而是多层 AVVideoView 的叠加**。**AVRootView 中维护了一个 AVVideoView 集合，默认创建 10 个 AVVideoView 在里面**。直播业务默认主播在第 0 层默认最大，其他互动观众分别在 1，2，3 层。每层大小都可以动态调节。

```java
private AVVideoView[] mVideoArr = new AVVideoView[ILiveConstants.MAX_AV_VIDEO_NUM];

// 初始化用户视频
public AVVideoView[] initVideoGroup() {
    ILiveLog.di(TAG, "initVideoGroup", new ILiveLog.LogExts().put("width", getWidth())
           .put("height", getHeight()).put("initLandScape", mInitLandScape));

    for (int i = 0; i < ILiveConstants.MAX_AV_VIDEO_NUM; i++) {
        // 初始化小屏参数
        mVideoArr[i] = new AVVideoView(getContext(), mGraphicRenderMgr);
        mVideoArr[i].setLocalRotationFix(localRotationFix);
        mVideoArr[i].setRemoteRotationFix(remoteRotationFix);
        mVideoArr[i].setLocalFullScreen(bLocalFullScreen);
        mVideoArr[i].setVisibility(GLView.INVISIBLE);
    }

    resetVideoViewPos();

    return mVideoArr;
}
```



![pic](http://mc.qcloudimg.com/static/img/d063a1980cc046cafa0444df0b609d02/image.png)

**示例**：

```java
<com.tencent.ilivesdk.view.AVRootView
        android:id="@+id/av_root_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/white" />

mAVRootView = (AVRootView) findViewById(R.id.av_root_view);
mAVRootView.setAutoOrientation(false);// 关闭腾讯 AVRootView 内部的自动旋转（
mAVRootView.setSubCreatedListener(new AVRootView.onSubViewCreatedListener() {
    // onSubViewCreated 是在小屏初始化完毕后的回调
    @Override
    public void onSubViewCreated() {
        mAVRootView.setDeviceRotation(270);
    }
});
ILVLiveManager.getInstance().setAvVideoView(mAVRootView);
```





### 直播互动

与直播功能性相关的方法，诸如举手、上下麦等操作统一定义在了 ITencentInteraction 接口中，开发者使用时，需要实例化一个实现类 ITencentInteractionImpl。



#### 开启、关闭屏幕分享

| 方法名        | 方法描述     |
| ------------- | ------------ |
| enableScreen  | 开启屏幕分享 |
| disableScreen | 关闭屏幕分享 |

| 参数类型        | 说明                   |
| --------------- | ---------------------- |
| TencentCallback | 开启屏幕分享结果的回调 |

**示例**：

```java
// 开启屏幕分享
ITencentInteraction mTencentInteraction = new TencentInteractionImpl();
mTencentInteraction.enableScreen(new TencentCallback() {
                @Override
                public void onSuccess() {
                    Log.d(TAG, "分享屏幕开启成功");
                }

                @Override
                protected void onFailed(String code, String message) {
                    Log.d(TAG, "分享屏幕开启失败");
                }
            });
...
//关闭屏幕分享
mTencentInteraction.disableScreen(new TencentCallback() {
                @Override
                public void onSuccess() {
                    Log.d(TAG, "分享屏幕关闭成功");
                }

                @Override
                protected void onFailed(String code, String message) {
                    Log.d(TAG, "分享屏幕关闭失败");
                }
            });
```



#### 上麦、下麦

| 方法名    | 方法描述 |
| --------- | -------- |
| upToMic   | 控制上麦 |
| downToMic | 控制下麦 |

| 参数类型        | 说明                 |
| --------------- | -------------------- |
| TencentCallback | 控制上下麦结果的回调 |

**示例**：

```java
// 上麦
ITencentInteraction mTencentInteraction = new TencentInteractionImpl();
mTencentInteraction.upToMic(new TencentCallback() {
                    @Override
                    public void onSuccess() {
                       ...
                    }

                    @Override
                    protected void onFailed(String code, String message) {
                        ...
                    }
                });
...
// 下麦
mTencentInteraction.downToMic(new TencentCallback() {
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



### 常见问题

* so 文件冲突

如若出现 libstlport_shared.so 冲突，这是 Xlog 中和直播 SDK 中的 so 冲突，需要在项目 gradle 的 android 下添加如下代码：

```java
packagingOptions {
     pickFirst '**/libstlport_shared.so'
} 
```

* Jenkins 打包报错

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

