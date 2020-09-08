# RN 配置热更新+使用文档

## 一、搭建本地`Code Push`服务

配置本地服务端和注册 app 等，参考

[最新 React Native 搭建本地 Code Push 服务(非常全！)](https://www.wddsss.com/main/displayArticle/224)

### 1. 首先全局安装微软提供的 code-push-cli 工具

```bash
npm install code-push-cli@latest -g
```

#### 常用`code-push`命令

-   注册账号: code-push register

-   登陆: code-push login

-   注销: code-push logout

-   添加项目: code-push app add app 名称

-   删除项目: code-push app remove app 名称

-   列出账号下的所有项目: code-push app list

-   显示登陆的 token: code-push access-key ls

-   部署一个环境: code-push deployment add appName deploymentName

-   删除部署: code-push deployment rm appName

-   列出应用的部署: code-push deployment ls appName

-   查询部署环境的 key: code-push deployment ls appName -k

我们会在后面使用此工具创建，发布，更新 App

### 2. 登录`Code Push`服务器

```bash
# code-push login http://localhost:3000
code-push login http://172.16.6.190:3000
```

```
账号：
admin
密码：
123456
```

> 如需要注销，执行下面命令

```bash
code-push logout
```

### 3. 创建应用

#### `Android` 应用

```bash
code-push app add CodePushAndroid android react-native
```

#### `iOS` 应用

```bash
code-push app add CodePushIos iOS react-native
```

然后输入到命令行终端中，执行完毕会生成两个 `key`, 发给原生开发人员

### 4. 更新应用

`react-native` 项目下执行，`test` 用来匹配是否检查安装应用

```bash
code-push release-react CodePushAndroid android --d Production --t 0.1.0 --des [test]生产环境更新1 --m true
```

```bash
code-push release-react CodePushAndroid android --d Staging --t 0.1.0 --des [test]测试环境更新1 --m true
```

```bash
code-push release-react CodePushIos ios --d Production --t 0.1.0 --des [test]生产环境更新1 --m true
```

```bash
code-push release-react CodePushIos ios --d Staging --t 0.1.0 --des [test]测试环境更新1 --m true
```

参数说明：

-   `d`: 发布的环境，`Staging`表示开发环境，`Production`表示生产环境
-   `t`: 版本号，只支持 3 位数，跟原生应用约定好一致
-   `des`: 版本描述：其中的`test`表示发给设定了版本表示为`test`的安装包

#### 静默更新应用(不弹窗，后台下载)

比如在`iOS`平台，执行以下命令：

测试环境

```bash

code-push release-react CodePushIos ios --d Staging --t 1.2.1 --des [test]{SilentUpdate}测试环境静默更新 --m true
```

生产环境

```bash
code-push release-react CodePushIos ios --d Production --t 1.2.1 --des [test]{SilentUpdate}测试环境静默更新 --m true
```

其中, 在`des`中添加`{SilentUpdate}`表示走静默更新，这边是完全匹配，如果匹配的话就在后台自动下载更新，不会再弹框提示

### 5. 获取应用的 `Key`

```bash
code-push deployment ls CodePushAndroid -k
```

```bash
code-push deployment ls CodePushIos -k
```

### 6. 清除推送的更新(终止)

```bash
code-push deployment clear CodePushAndroid staging
```

```bash
code-push deployment clear CodePushIos staging
```

## 二、配置安卓环境（自动 `link` 可能会导致失败）

### 1. `android/settings.gradle file` 添加

```
include ':app', ':react-native-code-push'
project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
```

### 2. `android/app/build.gradle` 添加

```
apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
```

### 3. `MainApplication.java` 添加

```java
import com.microsoft.codepush.react.CodePush;

public class MainApplication extends Application implements ReactApplication {
  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
      ...

      // 2. Override the getJSBundleFile method in order to let
      // the CodePush runtime determine where to get the JS
      // bundle location from on each app start
      @Override
      protected String getJSBundleFile() {
          return CodePush.getJSBundleFile();
      }
  };
}
```

### 4. `strings.xml` 添加

```xml
<resources>
    <string name="app_name">gms</string>
    <string moduleConfig="true" name="CodePushDeploymentKey">you key</string>
    <string moduleConfig="true" translatable="false" name="CodePushServerUrl">serverUrl</string>
</resources>

```

### 5. `bulid.gradle`

配置 `defaultConfig` 中的 `versionName` 为 3 位数，例如 `versionName` "1.0.0"

### 6.安装 `react-native-code-push 6.1.1`

```bash
yarn add react-native-code-push@6.1.1 -S
```

### 7.更新命令

```bash
code-push release-react CodePushDemoAndroid android --d Staging --t 1.0.0 --des 测试环境更新 --m true
```

## 配置 ios 环境（link）

### 1. 运行 cd ios && pod install

`pod` 如果失败, 就切换源，`profile` 文件头部添加

```
source 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'
```

### 2. AppDelegate.m 添加,及替换

```
#import <CodePush/CodePush.h>
...
...
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
  #if DEBUG
    return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
  #else
    return [CodePush bundleURL];
  #endif
}
```

### 3.xcode -> PROJECT -> Info -> Configurations -> 点击+ -> 选泽 Duplicate “Release Configaration ， 输入 Staging。 这表示新建一种打包方式 名称是 Staging。

### 4.xcode -> TARGET -> Build Setting，点击+，Add User-Defined Setting

然后输入 CODEPUSH_KEY，然后在下面的 release 中填写 product deployment key， 在 Staging 下填写 staging deploymeng key, debug 可以不用填写。

同理，新增 CODEPUSH_HOST， 填写各个环境的 code-push 服务器地址，如 Staging 我们可以填写 本机 iP://3000;（此处可能不执行，若失败可不添加）

### 5.ios/<项目>/info.plist 中新增

```
    <key>CodePushDeploymentKey</key>
    <string>$(CODEPUSH_KEY)</string>
    <key>CodePushServerURL</key>
    <string>$(CODEPUSH_HOST)</string>
    // 上一步的CODEPUSH_HOST若添加不成功，此处写死
    // 修改为三位数版本号
    <key>CFBundleShortVersionString</key>
    <string>1.7.0</string>
```

### 更新命令模板

```bash
code-push release-react CodePushDemoAndroid android --d Staging --t 1.0.0 --des 测试环境更新1 --m true
```

```bash
code-push release-react CodePushDemoIos ios --d Staging --t 1.0.0 --des 测试环境更新1 --m true
```
