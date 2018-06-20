

[TOC]

### 引入 SDK

在 Android Studio 中添加依赖，以 aar 的形式导入。(X.X.X 替换成对应版本号 比如 1.0.1)

```java
api 'com.tencent.livesdk:livesdk:X.X.X' 
```

### 直播流程示例

![pic](http://p3pvnofd3.bkt.clouddn.com/18-6-19/85970877.jpg)

### 接入直播

#### 初始化

| 接口名        | 接口描述                                          |
| ------------- | ------------------------------------------------- |
| initOkLiveSDK | 直播初始化，所有行为的第一步，告知身份 appid 信息 |

| 参数类型 | 说明 |
| -------- | ---- |
| Activity(Context) | 初始化 SDK 的上下文（建议使用ApplicationnContext） |
| int appid | 传入业务方 appid |
| int accountType | 传入业务方 accountType |

**示例**：

```java
// 初始化OkLiveSdk
TencentLiveManager.initLiveSDK(getApplicationContext, appid ,accoutype);
```



#### 账号登录、登出

账号相关的方法，诸如账号登录、登出等方法统一封装到了 ITencentlogin 接口中，使用时需要实例化一个 ITencentlogin 的实现类 TencentloginImpl。

| 接口名           | 接口描述                   |
| ---------------- | -------------------------- |
| ITencentlogin    | 负责直播登录部分的接口     |
| TencentloginImpl | ITencentlogin 接口的实现类 |

**方法**

| 方法名      | 方法描述                                                     |
| ----------- | ------------------------------------------------------------ |
| okLiveLogin | 使用独立模式，在获取到用户的 sig 后，使用登录接口，告知后台音视频模块上线了（包括 avsdk） |

| 参数类型         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| String id        | 用户 id，在直播过程中的唯一标识                              |
| String sig       | 鉴权的密钥 Sig，如果是独立登录方式，是业务方后台计算生成后下发 |
| ITencentCallback | 账号登录回调接口，通知上线是否成功                           |

| 方法名       | 方法描述     |
| ------------ | ------------ |
| okLiveLogout | liveSdk 登出 |

| 参数类型         | 说明                               |
| ---------------- | ---------------------------------- |
| ITencentCallback | 账号登出回调接口，通知登出是否成功 |

**示例**：

```java
ITencentlogin mTencentlogin = new ITenncentloginImpl(TestActivity.this);
mTencentlogin.login(mRoomConnectionParameters.userId, sig, new ITencentCallback() {
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
ITencentlogin mTencentlogin = new ITenncentloginImpl(TestActivity.this);
mTencentlogin.isLogin();
```



### 房间相关

#### 创建房间

与房间相关的方法统一定义在了 ITencentRoom 这个接口中，想要调用诸如创建房间、加入房间、退出房间等方法，需要实例化一个 ITencentRoomImpl 实例。

| 接口名           | 接口描述                                                   |
| ---------------- | ---------------------------------------------------------- |
| ITencentRoom     | 与直播房间相关的接口，包括创建房间内、加入房间、退出房间等 |
| ITencentRoomImpl | ITencentRoom 的接口实现类                                  |

| 方法名    | 方法描述                                             |
| --------- | ---------------------------------------------------- |
| creatRoom | 创建一个直播，只有在初始化和登录成功之后才能创建直播 |

| 参数类型         | 说明                 |
| ---------------- | -------------------- |
| String id        | 学生 id              |
| int roomid       | 房间 ID 房间唯一标识 |
| ITencentCallback | 创建房间的结果回调   |

**示例**：

```java
ITencentRoom mTencentRoom = new TencentRoomImpl(context);
mTencentRoom.creatRoom(userId, roomid,
                       new TencentCallback() {
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



#### 加入、退出房间

| 方法名   | 方法描述               |
| -------- | ---------------------- |
| joinRoom | 观众角色调用加入房间内 |

| 参数类型        | 说明                 |
| --------------- | -------------------- |
| int roomid      | 房间 ID 房间唯一标识 |
| TencentCallback | 加入房间结果的回调   |


| 方法名   | 方法描述         |
| -------- | ---------------- |
| quitRoom | 角色调用退出房间 |

| 参数类型        | 说明               |
| --------------- | ------------------ |
| TencentCallback | 退出房间的结果回调 |

**示例**：

```java
// 加入房间
ITencentRoom mTencentRoom = new TencentRoomImpl(context);
mTencentRoom.joinRoom(roomId, new TencentCallback() {
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
// 退出房间
mTencentRoom.quitRoom(new TencentCallback() {
                    @Override
                    public void onSuccess() {
                        Log.d(TAG, "腾讯云退出成功");
                        ...
                    }

                    @Override
                    public void onFailed(String code, String message) {
                        Log.d(TAG, "腾讯云退出失败, code: " + code + ", 原因: " + message);
                        ...
                    }
                });
```



#### 获取通话质量

| 方法名 | 方法描述 |
| ---|---|
| getQualityData | 获取通话质量 |

**示例**：

```java
ILiveQualityData qualityData = TencentLiveManager.getInstance(mContext).getQualityData();
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

