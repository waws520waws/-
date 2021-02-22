# （五）Appium+Python实现简单的自动化登录测试

## 前言

要想让手机app自动登录，也就是让app自己操作。所以在脚本中我们需要对app控件进行操作，那么我们需要获取控件的信息。可以使用..\android-sdk-windows\tools目录下的uiautomatorviewer.bat来获取控件相关信息

## 获取控件相关信息

### 启动uiautomatorviewer.bat

![img](https://img-blog.csdnimg.cn/20190110165103909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

### 打开手机app，例如计算器，USB连接电脑，点击uiautomatorviewer左上角的安卓机器人按钮Devices Screenshot按钮刷新页面

![img](https://img-blog.csdnimg.cn/20190110165419278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

### 定位元素：移动鼠标到需要定位的元素上，如数字7。右下角可以看到元素对应的属性

![img](https://img-blog.csdnimg.cn/20190110165715772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

## 登录脚本实现

```python
# coding=utf-8
__author__ = "Enoch"
# 这是一个app登录的测试
 
from appium import webdriver
from HTMLTestRunner import HTMLTestRunner
import unittest
import time
import warnings
 
 
class LoginTest(unittest.TestCase):
 
    def setUp(self):
        warnings.simplefilter("ignore", ResourceWarning)
        desired_caps = {
            'platformName': 'Android',
            'deviceName': 'SGEEGEHIQ8I7CIKF',
            'platformVersion': '6.0',
            'appPackage': 'com.mengtuiapp.mall',
            'appActivity': '.business.main.MainActivity'
        }
        self.driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)
 
    def testCase(self):
        u"""登录"""
        driver = self.driver
        # time.sleep(2)
        driver.find_element_by_id("bottom_nav").click()
        time.sleep(2)
        driver.find_element_by_name('使用其他方式登录').click()
        driver.find_element_by_name('手机登录').click()
        driver.find_element_by_id("username").send_keys("13100010001")
        driver.find_element_by_name('获取验证码').send_keys("9876")
        driver.find_element_by_id("btn").click()
        driver.quit()
 
 
if __name__ == '__main__':
        print("----------执行---------- ")
        suite = unittest.TestSuite()  # 构造测试集
        suite.addTest(LoginTest('testCase'))
        # 定义自动化报告目录
        filename = "F:\\report.html"
        fp = open(filename, 'wb')
        runner = HTMLTestRunner(
                stream=fp,
                title=u'自动化测试报告',
                description=u'这是登录测试的简单报告'
         )
        runner.run(suite)
        fp.close()
```