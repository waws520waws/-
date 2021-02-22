# （四）Appium使用HTMLTestRunner生成测试报告

## 介绍

HTMLTestRunner是一个单元测试运行器，它是 Python 标准库的 unittest 模块的一个扩展。它生成易于使用的 HTML 测试报告。在使用前我们首先需要下载HTMLTestRunner.py文件。[传送门](http://tungwaiyip.info/software/HTMLTestRunner.html)

## 如何使用

- Windows平台：将下载的文件放入...\Python27\Lib 目录下。
- Linux平台：需要先确定 python 的安装目录，打开终端，输入 python 命令进入 python 交互模式，通过 sys.path 可以查看本机 python 文件目录，以管理员身份将 HTMLTestRunner.py 文件考本到/usr/lib/python2.7/dist-packages/ 目录下

## 脚本实例

```python
# coding =utf-8
__author__ = "waws"
from appium import webdriver
from HTMLTestRunner import HTMLTestRunner
import unittest
class SearchTest(unittest.TestCase):
    def setUp(self):
        # 启动服务端时的参数设置
        desired_caps = {'platformName': 'Android',
                        'deviceName': 'SGEEGEHIQ8I7CIKF',
                        'platformVersion': '6.0',
                        'unicodeKeyboard': True,
                        'resetKeyboard': True,
                        'appPackage': 'com.mengtuiapp.mall',
                        'appActivity': '.business.main.MainActivity'
                        }
        self.driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)

    def testCase(self):
        u"""搜索测试"""
        driver = self.driver
        # 点击搜索按钮
        driver.find_element_by_id('tv_search').click()
        # 搜索框
        driver.find_element_by_id('search_text').send_keys(u"杯子")
        driver.find_element_by_id('search_tv').click()
        driver.quit()


if __name__ == '__main__':
    # 构造测试集
    suite = unittest.TestSuite()  # 构造测试集
    suite.addTest(SearchTest('testCase'))
    # 定义自动化报告目录
    filename = 'F:\\sousuo.html'
    fp = open(filename, 'wb')
    runner = HTMLTestRunner(
            stream=fp,
            title=u'自动化测试报告',
            description=u'这是一个简单的测试报告',
            verbosity=2
     )
    runner.run(suite)
    fp.close()
```

## 运行结果

![img](https://img-blog.csdnimg.cn/20190110140020535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)