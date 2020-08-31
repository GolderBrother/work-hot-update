# RN 配置热更新+使用文档

## 一、搭建本地`Code Push`服务

配置本地服务端和注册 app 等，参考

[最新 React Native 搭建本地 Code Push 服务(非常全！)](https://www.wddsss.com/main/displayArticle/224)

### 1. 登录`Code Push`服务器

```bash
code-push login http://localhost:3000
```

下面的命令参数已经跟原生约定好

### 2. 创建应用

```bash
code-push app add CodePushAndroid android react-native
```

执行完毕会生成两个 `key`, 发给原生开发人员

### 3. 更新应用

`react-native` 项目下执行，`test` 用来匹配是否检查安装应用

```bash
code-push release-react CodePushAndroid android --d Production --t 0.1.0 --des [test]测试环境更新 1 --m true
```

```bash
code-push release-react CodePushAndroid android --d Staging --t 0.1.0 --des [test]测试环境更新 1 --m true
```

```bash
code-push release-react CodePushIos ios --d Production --t 0.1.0 --des [test]测试环境更新 1 --m true
```

```bash
code-push release-react CodePushIos ios --d Staging --t 0.1.0 --des [test]测试环境更新 1 --m true
```

### 4. 获取应用的 `Key`

```bash
code-push deployment ls CodePushAndroid -k
```

```bash
code-push deployment ls CodePushIos -k
```

### 5. 清除推送的更新(终止)

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
