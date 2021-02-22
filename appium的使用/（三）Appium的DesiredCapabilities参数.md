# （三）Appium的DesiredCapabilities参数

## 前言

这个参数是干嘛的，有啥子用，下面我们一一道来。首先Desired Capabilities 是由 keys 和 values 组成的 JSON 对象。它负责启动服务端时的参数设置，启动session的时候是必须提供的。它告诉appium server这样一些事情，比如：

- 本次测试是启动浏览器还是启动移动设备？
- 是启动andorid还是启动ios？
- 启动android时，app的package是什么？
- 启动android时，app的activity是什么？

说白了，我们做app自动化测试的时候，这个是**配置连接手机的相关信息**。那么有哪些配置呢。

## 基本参数

| 参数                | 描述                                                         | 实例                                                  |
| :------------------ | :----------------------------------------------------------- | :---------------------------------------------------- |
| `automationName`    | 自动化测试引擎                                               | `Appium`或 `Selendroid`                               |
| `platformName`      | 手机操作系统                                                 | `iOS`, `Android`, 或 `FirefoxOS`                      |
| `platformVersion`   | 手机操作系统版本                                             | 如： `7.1`, `4.4`；ios的 `9.0`                        |
| `deviceName`        | 手机或模拟器设备名称                                         | android的忽略，ios如`iPhone Simulator`                |
| `app`               | `.ipa` `.apk`文件路径                                        | 比如`/abs/path/to/my.apk`或`http://myapp.com/app.ipa` |
| `browserName`       | 启动手机浏览器                                               | iOS如:`Safari，Android`如:`Chrome,Chromium,Browser`   |
| `newCommandTimeout` | 设置命令超时时间，单位：秒。                                 | 比如 `60`                                             |
| `autoLaunch`        | Appium是否需要自动安装和启动应用。默认值`true`               | `true`, `false`                                       |
| `language`          | (Sim/Emu-only) 设定模拟器 ( simulator / emulator ) 的语言。  | 如： `fr`                                             |
| `locale`            | (Sim/Emu-only) 设定模拟器 ( simulator / emulator ) 的区域设置。 | 如： `fr_CA`                                          |
| `udid`              | ios真机的唯一设备标识                                        | 如： `1ae203187fc012g`                                |
| `orientation`       | 设置横屏或竖屏                                               | `LANDSCAPE` (横向) 或 `PORTRAIT` (纵向)               |
| `autoWebview`       | 直接转换到 WebView 上下文。 默认值 `false`、                 | `true`, `false`                                       |
| `noReset`           | 不要在会话前重置应用状态。默认值`false`。                    | `true`, `false`                                       |
| `fullReset`         | (iOS) 删除整个模拟器目录。(Android)通过卸载默认值 `false`    | `true`, `false`                                       |

## Android特有

| 关键字                      | 描述                                                         | 实例                                                         |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `appActivity`               | 启动app包,一般点开头                                         | 如：`.MainActivity`, `.Settings`                             |
| `appPackage`                | Android应用的包名                                            | 比如`com.example.android.myApp`                              |
| `appWaitActivity`           | 等待启动的Activity名称                                       | `SplashActivity`                                             |
| `deviceReadyTimeout`        | 设置超时时间                                                 | `5`                                                          |
| `androidCoverage`           | 用于执行测试的 instrumentation类                             | `com.my.Pkg/com.my.Pkg.instrumentation.MyInstrumentation`    |
| `enablePerformanceLogging`  | (仅适用于 Chrome 和 webview) 开启 Chromedriver 的性能日志。(默认 `false`) | `true`, `false`                                              |
| `androidDeviceReadyTimeout` | 等待设备在启动应用后超时时间，单位秒                         | 如 `30`                                                      |
| `androidDeviceSocket`       | 开发工具的 socket 名称。Chromedriver 把它作为开发者工具来进行连接。 | 如 `chrome_devtools_remote`                                  |
| `avd`                       | 需要启动的 AVD (安卓模拟器设备) 名称。                       | 如 `api19`                                                   |
| `avdLaunchTimeout`          | 以毫秒为单位，等待 AVD 启动并连接到 ADB的超时时间。(默认值 `120000`) | `300000`                                                     |
| `avdReadyTimeout`           | 以毫秒为单位，等待 AVD 完成启动动画的超时时间。(默认值 `120000`) | `300000`                                                     |
| `avdArgs`                   | 启动 AVD 时需要加入的额外的参数。                            | 如 `-netfast`                                                |
| `useKeystore`               | 使用一个自定义的 keystore 来对 apk 进行重签名。默认值 `false` | `true` or `false`                                            |
| `keystorePath`              | 自定义keystore路径。默认~/.android/debug.keystore            | 如 `/path/to.keystore`                                       |
| `keystorePassword`          | 自定义 keystore 的密码。                                     | 如 `foo`                                                     |
| `keyAlias`                  | key 的别名                                                   | 如 `androiddebugkey`                                         |
| `keyPassword`               | key 的密码                                                   | 如 `foo`                                                     |
| `chromedriverExecutable`    | webdriver可执行文件的绝对路径 应该用它代替Appium 自带的 webdriver) | `/abs/path/to/webdriver`                                     |
| `autoWebviewTimeout`        | 毫秒为单位，Webview上下文激活的时间。默认`2000`              | 如 `4`                                                       |
| `intentAction`              | 用于启动activity的intent action。(默认值 `android.intent.action.MAIN`) | 如 `android.intent.action.MAIN`, `android.intent.action.VIEW` |
| `intentCategory`            | 用于启动 activity 的 intent category。 (默认值 `android.intent.category.LAUNCHER`) | 如 `android.intent.category.LAUNCHER`, `android.intent.category.APP_CONTACTS` |
| `intentFlags`               | 用于启动activity的标识(flags) (默认值 `0x10200000`)          | 如 `0x10200000`                                              |
| `optionalIntentArguments`   | 用于启动 activity 的额外 intent 参数。请查看 [Intent 参数](http://developer.android.com/tools/help/adb.html#IntentSpec) | 如 `--esn <EXTRA_KEY>`, `--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE>` |
| `dontStopAppOnReset`        | 在使用 adb 启动应用时不要停止被测应用的进程。默认值： `false` | `true` 或 `false`                                            |
| `unicodeKeyboard`           | 使用 Unicode 输入法。默认值 `false`                          | `true` 或 `false`                                            |
| `resetKeyboard`             | 重置输入法到原有状态，默认值 `false`                         | `true` 或 `false`                                            |
| `noSign`                    | 跳过检查和对应用进行 debug 签名的步骤。默认值 `false`        | `true` 或 `false`                                            |
| `ignoreUnimportantViews`    | 调用 uiautomator 的函数这个关键字能加快测试执行的速度。默认值 `false` | `true` 或 `false`                                            |
| `disableAndroidWatchers`    | 关闭 android 监测应用无响ANR和崩溃crash的监视器默认值： `false`。 | `true` 或者 `false`                                          |
| `chromeOptions`             | 允许传入 chrome driver 使用的 chromeOptions 参数。请查阅 [chromeOptions](https://sites.google.com/a/chromium.org/chromedriver/capabilities) 了解更多信息。 | `chromeOptions: {args: [‘--disable-popup-blocking‘]}`        |

## iOS特有

| 关键字                        | 描述                                                         | 实例                                                         |
| :---------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `calendarFormat`              | (Sim-only) 为iOS的模拟器设置日历格式                         | 如 `gregorian` (公历)                                        |
| `bundleId`                    | **被测应用的bundle ID，真机上执行测试时，你可以不提供 `app` 关键字，但你必须提供udid** | 如 `io.appium.TestApp`                                       |
| `udid`                        | **连接真机的唯一设备编号 ( Unique device identifier )**      | 如 `1ae203187fc012g`                                         |
| `launchTimeout`               | 以毫秒为单位，在Appium运行失败之前设置一个等待 instruments的时间 | 比如： `20000`                                               |
| `locationServicesEnabled`     | (Sim-only) 强制打开或关闭定位服务。默认值是保持当前模拟器的设定 | `true` 或 `false`                                            |
| `locationServicesAuthorized`  | 使用这个关键字时，你同时需要使用 `bundleId` 关键字来发送你的应用的 bundle ID。 | `true` 或者 `false`                                          |
| `autoAcceptAlerts`            | 当 iOS 的个人信息访问警告 (如 位置、联系人、图片) 出现时，自动选择接受( Accept )。默认值 `false`。 | `true` 或者 `false`                                          |
| `autoDismissAlerts`           | 当 iOS 的个人信息访问警告 (如 位置、联系人、图片) 出现时，自动选择不接受( Dismiss )。默认值 `false`。 | `true` 或者 `false`                                          |
| `nativeInstrumentsLib`        | 使用原生 intruments 库 (即关闭 instruments-without-delay )   | `true` 或者 `false`                                          |
| `nativeWebTap`                | (Sim-only) 在Safari中允许"真实的"，默认值： `false`。注意：取决于 viewport 大小/比例， 点击操作不一定能精确地点中对应的元素。 | `true` 或者 `false`                                          |
| `safariInitialUrl`            | (Sim-only) (>= 8.1) Safari 的初始地址。默认值是一个本地的欢迎页面 | 例如： `https://www.github.com`                              |
| `safariAllowPopups`           | (Sim-only) 允许 javascript 在 Safari 中创建新窗口。默认保持模拟器当前设置。 | `true` 或者 `false`                                          |
| `safariIgnoreFraudWarning`    | (Sim-only) 阻止 Safari 显示此网站可能存在风险的警告。默认保持浏览器当前设置。 | `true` 或者 `false`                                          |
| `safariOpenLinksInBackground` | (Sim-only) Safari 是否允许链接在新窗口打开。默认保持浏览器当前设置。 | `true` 或者 `false`                                          |
| `keepKeyChains`               | (Sim-only) 当 Appium 会话开始/结束时是否保留存放密码存放记录 (keychains) (库(Library)/钥匙串(Keychains)) | `true` 或者 `false`                                          |
| `localizableStringsDir`       | 从哪里查找本地化字符串。默认值 `en.lproj`                    | `en.lproj`                                                   |
| `processArguments`            | 通过 instruments 传递到 AUT 的参数                           | 如 `-myflag`                                                 |
| `interKeyDelay`               | 以毫秒为单位，按下每一个按键之间的延迟时间。                 | 如 `100`                                                     |
| `showIOSLog`                  | 是否在 Appium 的日志中显示设备的日志。默认值 `false`         | `true` 或者 `false`                                          |
| `sendKeyStrategy`             | 输入文字到文字框的策略。模拟器默认值：`oneByOne` (一个接着一个) 。真实设备默认值：`grouped` (分组输入) | `oneByOne`, `grouped` 或 `setValue`                          |
| `screenshotWaitTimeout`       | 以秒为单位，生成屏幕截图的最长等待时间。默认值： 10。        | 如 `5`                                                       |
| `waitForAppScript`            | 用于判断 "应用是否被启动” 的 iOS 自动化脚本代码。默认情况下系统等待直到页面内容非空。结果必须是布尔类型。 | 例如 `true;`, `target.elements().length > 0;`, `$.delay(5000); true;` |

## 附录

**获取设备名称`deviceName`**

**![img](https://img-blog.csdnimg.cn/20190104181153587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)**

**获取appPackage和appActivity（打开app，命令行输入）**

**![img](https://img-blog.csdnimg.cn/20190104181540625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)**

**实例**

```python
# 启动服务端时的参数设置
desired_caps = {'platformName': 'Android',
                'deviceName': 'SGEEGEHIQ8I7CIKF',
                'platformVersion': '6.0',
                'unicodeKeyboard': True,
                'resetKeyboard': True,
                'appPackage': 'com.mengtuiapp.mall',
                'appActivity': '.business.main.MainActivity'
               }
```

