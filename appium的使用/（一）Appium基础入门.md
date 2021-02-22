# （一）Appium基础入门

## 介绍

Appium是一个自动化测试开源工具，支持IOS和Android平台上的移动原生应用、移动Web应用和混合应用。Appium是一个跨平台工具，它允许测试人员使用同样的接口、基于 不同的平台写自动化测试代码，大大增加了测试套件间代码的复用性。

- 移动原生应用：是指那些用ios或android sdk写的应用
- 移动web应用：是指那些使用移动浏览器访问的应用，appium支持ios的safari和android的chrome
- 混合应用：是指原生代码封装在网页视图（原生代码和web内容交互）

## 理念

1. 无需为了自动化，而重新编译或者修改我们的应用；
2. 不必局限于某种语言或者框架来写和运行测试脚本；
3. 一个移动自动化的框架不应该在接口上重复造轮子；
4. 无论精神上，还是名义上，都必须要开源。

## 设计

- Appium的真正的工作引擎是第三方自动化框架，不需要在本身应用里植入appium特定或者第三方代码。 
- Appium把这些第三方框架封装成一套API，即WebDriver API。
- Appium扩充了WebDriver的协议，在原有的基础上添加自动化相关的API方法。

## 架构原理

Appium是在手机操作系统自带的测试框架基础上实现的，Android和IOS的系统上使用的工具分别如下：

- Android(版本>4.3):**UIAutomator,Android4.3之后系统自带的UI自动化测试工具**
- Android(版本<=4.3):Selendroid,基于Android Instrumentation框架实现的自动化测试工具
- IOS:UIAutomation (instruments框架里面的一个模版)，IOS系统自带的UI自动化测试工具

![img](https://img-blog.csdnimg.cn/20190118101750542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

注：图片来自百度

**运行原理**

我们的电脑（client）上运行自动化测试脚本，调用的是webdriver的接口，appium server（默认监听 4723 端口 ）接收到我们client上发送过来的命令后appium server 会把请求转发给中间件Bootstrap.jar ，它是用java写的，安装在 手机上 ，Bootstrap监听 4723端口并接收appium 的命令，最终通过调 的命令过调用UIAutomator 的命令来实现。Bootstrap将执行的结果返回给 将执行的结果返回给 appium server 。Appiumserver再将结果返回给 client端。

**Appium服务器**

Appium服务器是Appium框架的核心，它是一个基于Node.js实现的HTTP服务器。Appium服务器的主要功能是接受从Appium客户端发起的连接，监听从客户端发送来的命令，将命令发送发bootstrap.jar（IOS手机为bootstrap.js）执行，并将命令的执行结果通过HTTP应答反馈给Appium客户端。

**Bootstrap.jar**

bootstrap.jar是在Android手机上运行的一个应用程序，它在手机上扮演TCP服务器的角色。当Appium服务器需要运行命令时，Appium服务器会与Bootstrap.jar建立TCP通信，并把命令发送给Bootstrap.jar;Bootstrap.jar负责运行测试命令。

**Appium客户端**

它主要是指实现了Appium功能的webDriver协议的客户端Library,它负责与Appium服务器建立连接，并将测试脚本的指令发送到Appium服务器。现有的客户端Library由多种语言实现，包括Ruby，Python，Java等等。Appium的测试是在这些Library的基础上进行开发的。

## 相关概念

### C/S架构 

Appium的核心是一个Web服务器，它提供了一套REST的接口。它接收到客户端的连接、监听的命令，接着在移动设备上执行这些命令，然后将执行的结果放在HTTP响应中返还给客户端。

### Session 

**自动化总是在一个session的上下文中运行，客户端初始化一个和服务端交互的session。客户端发送一个附有desired capabilities的JSON对象参数的POST请求“/session”给服务器，服务端就会开始一个自动话的session，然后返回一个session ID，客户端拿到这个ID后就用这个ID发送后续的命令。**

### Desired Capabilities 

**Desired Capabilities是一些键值对。客户端将这些键值对发给服务端，告诉服务端我们想要启动怎样的自动化session。**

### Appium server 

Appium server是用Nodejs写的，我们可以源码编译或者从NPM直接安装。

### Appium服务端 

Appium服务端有很多语言库Java，Ruby，Python，PHP，JavaScript，C#等，这些库实现了对WebDriver协议的扩展。使用Appium的时候，只需使用这些库代替常规的WebDriver库就可以了。

### Appium.app，Appium.exe 

Appium提供了GUI封装的Appium server下载，它封装了运行Appium server的所有依赖元素。而且这个封装包含了一个Inspector工具，可以让使用者检查应用的界面元素层级。