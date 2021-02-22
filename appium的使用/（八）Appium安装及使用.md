# （八）Appium安装及使用

## 前言

在介绍appium的安装与使用之前，我们需要再次来回顾一下appium的特点，下面以总结的两个图为开始。

![img](https://img-blog.csdnimg.cn/20190520165716822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190520165735463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

## 安装及使用

**1、安装appium server**

 方法一:npm install -g appium

   •安装node.js,官方网站：[https://nodejs.org](https://nodejs.org/)[/](https://nodejs.org/)

   •Windows 命令提示符，npm命令回车,验证是否安装成功

   •npm install -g appium命令安装

 方法二:安装appium desktop（推荐）官网下载成功后点击安装。

**2、安装Android  SDK 主要是依赖里面的库**

  •安装Android Studio(安装慢,文件大)建议安装2.3.3版本

  •网址：https://developer.android.google.cn/studio/

  •配置Android_HOME变量,将Android SDK的目录创建为变量并将变量

   加到path中，%Android_HOME%\tools;%Android_HOME%\platform-tools;

**3、安装Java JDK,并配置环境变量**

**4、appium-desktop使用**

   连接好测试机,点击appium server图标，启动appium-desktop

![img](https://img-blog.csdnimg.cn/20190520170525931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190520170545767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)