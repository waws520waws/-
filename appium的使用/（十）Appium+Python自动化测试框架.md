# （十）Appium+Python自动化测试框架

## 前言

前面几篇文章介绍了相关的技术实现，比如自动化登录、appium安装、错误重跑、生成报告等等。接下来我们看看如何搭建一个比较真、善、美的结构，实现特定的功能，每个模块分工明确，思路清晰。直接上图：

![img](https://img-blog.csdnimg.cn/20190731102204902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

##  模块介绍

 Appium+Python 自动化结构主要包括 6 个部分。

- appPerf 结构下主要存放文件是 app 运行时的性能监控，包括内存，cpu，电量等。
- common 结构作为公共文件，主要存放的文件如获取 yaml 文件内容、与报告相关的HTMLTestRunner、截图文件等。
- config 结构作为配置文件，该目录下主要存放与手机的配置信息有关文件。
- initdata 结构用作数据初始化。主要是一些接口文件，把账号的相关信息诸如创建订单、地址等在运行前初始化完成。
- testModule 结构下主要是用于存放测试用例。测试用例的脚本在 yaml 目录下编写，且每个 yaml 文件对应一个用例。
- testResult 结构下主要存放项目运行产生的报告、截图、log 等。
- testRunner 结构下主要是存放项目运行时需要的文件。
- utils 顾名思义工具，此目录下主要存放一些工具，比如针对这个项目来所主要存放于上传文件到阿里云相关的工具类。