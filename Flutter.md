# Flutter

[toc]

## 环境配置
具体环境配置可参考 [Flutter 中文网](https://flutterchina.club/setup-macos/)的教程

### 主要命令及问题解决
```
export PATH="$PATH:`pwd`/flutter/bin"
```

```
flutter doctor
```

```
  ╔════════════════════════════════════════════════════════════════════════════╗
  ║                 Welcome to Flutter! - https://flutter.io                   ║
  ║                                                                            ║
  ║ The Flutter tool anonymously reports feature usage statistics and crash    ║
  ║ reports to Google in order to help Google contribute improvements to       ║
  ║ Flutter over time.                                                         ║
  ║                                                                            ║
  ║ Read about data we send with crash reports:                                ║
  ║ https://github.com/flutter/flutter/wiki/Flutter-CLI-crash-reporting        ║
  ║                                                                            ║
  ║ See Google's privacy policy:                                               ║
  ║ https://www.google.com/intl/en/policies/privacy/                           ║
  ║                                                                            ║
  ║ Use "flutter config --no-analytics" to disable analytics and crash         ║
  ║ reporting.                                                                 ║
  ╚════════════════════════════════════════════════════════════════════════════╝


Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, v1.0.0, on Mac OS X 10.14.3 18D42, locale zh-Hans-CN)
[✗] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from: https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK components.
      (or visit https://flutter.io/setup/#android-setup for detailed instructions).
      If Android SDK has been installed to a custom location, set $ANDROID_HOME to that location.
      You may also want to add it to your PATH environment variable.

[!] iOS toolchain - develop for iOS devices (Xcode 10.1)
    ✗ libimobiledevice and ideviceinstaller are not installed. To install with Brew, run:
        brew update
        brew install --HEAD usbmuxd
        brew link usbmuxd
        brew install --HEAD libimobiledevice
        brew install ideviceinstaller
    ✗ ios-deploy not installed. To install with Brew:
        brew install ios-deploy
[!] Android Studio (not installed)
[!] Connected device
    ! No devices available

! Doctor found issues in 4 categories.

```

### 重新安装 Homebrew
由于 install link usbmuxd 时依赖库问题，需要重新安装 Homebrew

先卸载已安装的homebrew，命令如下：

``` 
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```
然后重新安装：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 更新环境变量


```
vi $HOME/.bash_profile
```


```
export PUB_HOSTED_URL=https://pub.flutter-io.cn //国内用户需要设置
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn //国内用户需要设置
export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin" //如下
//export PATH="$PATH:/Users/zhangke/Documents/development/flutter/bin"
```

```
source $HOME/.bash_profile //$HOME 当前用户目录
```

```
echo $PATH //查看环境变量
```


要删除第三处的冗余路径
export PATH=`echo $PATH | cut -d":" -f1,2,4-`


### Flutter卡在Running "flutter packages get"
应该是缺少镜像的环境变量

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```