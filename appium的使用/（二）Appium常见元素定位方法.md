# （二）Appium常见元素定位方法

## 前言

appium的核心其实就是一个web服务器，它提供了一套REST的接口。首先它收到客户端的连接、监控命令，之后在移动设备上执行这些命令，最后把执行结果放在HTTP响应中返回给客户端。基于上述原理，appium框架提供了一系列的API供调用。以下简单介绍常见的API

## 分类

![img](https://img-blog.csdnimg.cn/20190103144541766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

## 控件定位

**根据Id定位**

- find_element_by_id（self, id_）
  通过元素的**id定位**，返回含有该属性的元素，在android上 **app上即resource id**
- find_elements_by_id（self, id_）
  通过元素的id定位,返回含有该属性的所有元素

**根据名称name定位**

- find_element_by_name(self, name)
  通过元素**Name定位**，返回含有该属性的元素，对于android，即**text属性**
- find_elements_by_name(self, name)
  通过元素Name定位（元素的名称属性text），含有该属性的所有元素

**根据xpath**

- find_element_by_xpath（self, xpath）
  通过Xpath定位，返回含有该属性的元素
- find_elements_by_xpath（self, xpath）
  通过Xpath定位，返回含有该属性的所有元素

**根据class_name**

- find_element_by_class_name（self,class_name）
  通过元素class name属性定位,返回包含该属性的元素
- find_elements_by_class_name(self,class_name)
  通过元素class name属性定位,返回包含该属性的所有元素

**根据accessibility_id**

- find_element_by_accessibility_id(self,accessibility_id)
- find_elements_by_accessibility_id(self,accessibility_id)
  通过**accessibility id定位**，在android app上相当于**content-description字段**，而在ios app 上**accessibility identifier字段**

## 手势操作

- click（self） 点击
- send_keys(self, *value）在元素中模拟输入（开启appium自带的输入法并配置了appium输入法后，即unicodeKeyboard、resetKeyboard，可以输入中英文
- clear(self) 清除输入的内容
- swipe(self, start_x, start_y, end_x, end_y, duration=None) 滑动，主要用于缓慢滑动，从（start_x, start_y）点滑动到（end_x, end_y）点，可以自定义duration【毫秒】滑动时间
- flick(self, start_x, start_y, end_x, end_y) 快速滑动，主要用于快速滑动,无duration，如View切换，按住（start_x, start_y）点后快速滑动至（end_x, end_y）点
- shake(self) 摇一摇 
- lock(self, seconds) 锁屏一段时间 IOS专有
- scroll(self, origin_el, destination_el) 滚动从元素origin_el滚动至元素destination_el，只有iOS可以使用
- drag_and_drop(self, origin_el, destination_el) 拖放将元素origin_el拖到目标元素destination_el
- zoom(self, element=None, percent=200, steps=50) 放大在元素上执行放大
- pinch(self, element=None, percent=200, steps=50) 缩小在元素上执行缩
- tap(self, positions, duration=None) 点击模拟手指点击（最多五个手指），可设置按住时间长度（单位毫秒），positions参数为单位为元组的列表，如[(x1,y1),(x2,y2)]
- keyevent(self, keycode, metastate=None) 发送按键码发送按键码（安卓仅有），常见的有:
  - KEYCODE_HOME (按键Home) : 3
  - KEYCODE_MENU (菜单键) : 82
  - KEYCODE_BACK (返回键) : 4

```
#用法
element.click()
element.send_keys("测试")
element.clear()
driver.swipe(100,100,100,400)
driver.flick(100,100,100,400)
driver.shake()
driver.lock()
driver.scroll(el1,el2)
driver.drag_and_drop(el1,el2)
driver.zoom(element)#默认分成50步完成,放大量为200%
driver. pinch (element)
driver.tap([(300,500)],10)
driver.keyevent(4) 
```

## 获取设备及控件相关信息

- get_window_size (self, windowHandle='current') 获取当前设备的宽、高
- location 获取元素的左上角坐标
- size 获取元素的宽、高
- current_activity 获取设备当前的activity值
- context 获取当前会话的当前上下文
- app_strings(self, language=None, string_file=None)对于Android，获取app的strings.xml文件内容

```python
#用法
width = driver.get_window_size()['width']
height = driver.get_window_size()['height']
x = element.location['x']
y = element.location['y']
width = element.size['width']
height = element.size['height']
driver.current_activity
driver.context
driver.app_strings
```

## 其他操作

- wait_activity（self, activity, timeout, interval=1） 等待指定activity的出现等待指定activity的出现，返回true or false，默认是轮询间隔是1s，需要值得注意的是这里的参数是current_activity打印出来的值，即不包含包名
- quit(self) 退出退出脚本运行，并关闭每个相关的窗口连接
- background_app(self, seconds) 后台运行，把app放置于后台运行，设置时间seconds，单位s，相当于一段时间后重启app，而不是home将app放置后台
- save_screenshot(self, filename) 截图截图，filename参数为路径，包含文件名称

```python
#用法
driver.wait_activity(activity,timeout,interval)
driver.quit()
driver.background_app(3)
driver.save_screenshot('/Screenshots/inchlifc.png')
```

## 结语

以上是我们在测试中常使用到的一些API，实际上appium提供的API还有很多。本文中可能有一些没有提及，建议大家可以参考appium的API中文文档。