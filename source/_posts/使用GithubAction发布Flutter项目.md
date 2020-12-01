---
title: 使用GithubAction发布Flutter项目-Android
tags: 
- Github
- Flutter
comments: true
categories: 
  - technology
toc: true
keywords: Github Action,Flutter,Release,发布,Android
date: 2020-11-01 08:40:51
---

`CI`目标: 使用`GithubAction`发布`Flutter`项目
- `Android` - `PlayStore`/`GithubRelease`
- `IOS` - `AppStore`
- `Macos` - `GithubRelease`
- `Linux` - `GithubRelease`
- `Windows` - `GithubRelease`

![](https://images.di1shuai.com/flutter_ci.jpg)

那就先来看看如何`CI`发布`Flutter`-`Android`吧


## Android - 本地

### App Logo 修改

1. 修改`pubspec.yaml`下的:

```yml
dev_dependencies:
  flutter_launcher_icons: ^0.8.1

flutter_icons:
  android: "launcher_icon"
  ios: true
  image_path: "assets/logo/logo.png"
```

2. 运行

```bash
flutter pub get
flutter pub run flutter_launcher_icons:main
```

### App Name 修改

1. 修改`pubspec.yaml`下的:

```yml
dev_dependencies:
  flutter_launcher_name: ^0.0.1

flutter_launcher_name:
  name: "APPName"
```

2. 运行

```bash
flutter pub get
flutter pub run flutter_launcher_name:main
```

Check

`Android` - `android/app/src/main/AndroidManifest.xml`

`IOS` - `ios/Runner/Info.plist`

### 权限

文件路径 : `android/app/src/main/AndroidManifest.xml`

`manifest`之后加入`uses-permission`标签，e.g.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.di1shuai.xxx">
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>

```

具体权限名称参考 [Manifest.permission](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn)

### 签名

1. 创建`key.jks`,已有可忽略

- Mac/Linux

```bash
keytool -genkey -v -keystore ~/key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias key
```

- Windows

```bash
keytool -genkey -v -keystore c:\Users\USER_NAME\key.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias key
```

2. 配置`Gradle`

路径: `android/app/build.gradle`

- 设置 `signingConfigs` 以及 `buildTypes`

```gradle
android {
  
    ...
    
    defaultConfig {
       ...
    }

    <!-- import start -->
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            signingConfig signingConfigs.release
        }
    }
    <!-- import end -->

}
```

- 设置`keystoreProperties`

```gradle
...
apply plugin: 'com.android.application'
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

<!-- import start -->
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
} else {
    keystoreProperties.setProperty('storePassword', System.getenv('KEY_STORE_PASSWORD'));
    keystoreProperties.setProperty('keyPassword', System.getenv('KEY_PASSWORD'));
    keystoreProperties.setProperty('keyAlias', System.getenv('KEY_ALIAS'));
    keystoreProperties.setProperty('storeFile', System.getenv('KEY_PATH'));
}
<!-- import end -->

android {
    compileSdkVersion 29
...
```

3. 设置环境变量
  
path: `macos`/`linux` 下 `~/.zshrc` or `~/.bashrc`

```bash
## Flutter - Android
export KEY_STORE_PASSWORD=XXXXX
export KEY_PASSWORD=XXXXX
export KEY_ALIAS=key
export KEY_PATH=~/key.jks
```

### 本地 Build

- `APK` Build

```bash
flutter build apk --obfuscate --split-per-abi --split-debug-info=./info
```

`.apk` 路径

```bash
build/app/outputs/apk/release/app-x86_64-release.apk
build/app/outputs/apk/release/app-armeabi-v7a-release.apk
build/app/outputs/apk/release/app-arm64-v8a-release.apk
```

- `AAB` Build

```bash
flutter build appbundle --obfuscate --split-debug-info=./info
```

`.aab` 路径 `build/app/outputs/bundle/release/app-release.aab`


## Github Action CI

### Secrets

```bash
KEY_ALIAS           # e.g. key
KEY_PASSWORD        # e.g. xxxxxx
KEY_PATH            # e.g. key.jks
KEY_STORE_PASSWORD  # e.g. xxxxxx
PACKAGE_NAME        # 包名 e.g. com.di1shuai.xxxxx
PLAY_JSON           # e.g. {xxxxxx}
SIGNING_KEY         # 加密key后得到
```

- `SIGNING_KEY` 加密`key`得到

```bash
openssl base64 -A -in ~/key.jks
```

### 配置 Action

路径 `.github/workflows/build.yml`

```yml
name: beta-build

on:
  push:
    tags:
      - '*-beta*'

jobs:
  android-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: '11'
      - name: Cache Flutter Dependencies
        uses: actions/cache@v1
        with:
          path: /opt/hostedtoolcache/flutter
          key: ${{ runner.os }}-flutter
      - uses: subosito/flutter-action@v1
        with:
          channel: 'dev'
      - name: Env Init
        run: |
          echo "${{ secrets.SIGNING_KEY }}" | base64 --decode > android/app/${{secrets.KEY_PATH}}
      - name: AAB/APK Build
        run: |
          flutter build appbundle --obfuscate --split-debug-info=./info
          flutter build apk --obfuscate --split-per-abi --split-debug-info=./info
        env:
          KEY_STORE_PASSWORD: ${{ secrets.KEY_STORE_PASSWORD }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PATH: ${{ secrets.KEY_PATH }}
      - name: Push AAB/APK Github Releases
        uses: ncipollo/release-action@v1
        with:
          artifacts: "build/app/outputs/bundle/release/app-release.aab,build/app/outputs/apk/release/*.apk"
          token: ${{ secrets.GITHUB_TOKEN }}
      - name: Upload AAB to Artifact
        uses: actions/upload-artifact@v2
        with:
          name: appbundle
          path: build/app/outputs/bundle/release/app-release.aab

  aab-release:
    name: Release app to beta track
    needs: [ android-build ]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Get appbundle from artifacts
        uses: actions/download-artifact@v2
        with:
          name: appbundle
      - name: Release app to beta track
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.PLAY_JSON }}
          packageName: ${{secrets.PACKAGE_NAME}}
          releaseFiles: app-release.aab
          track: beta
          whatsNewDirectory: distribution/whatsnew

```

### Push Tag

```bash
git tag 1.0.0-beta1
git push origin 1.0.0-beta1
```
