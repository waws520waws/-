# （十一）Appium自动化测试断言的实现

## 前言

首先我们要思考的一点是，什么是断言？百度百科是这样描述的“断言是编程术语，表示为一些布尔表达式，程序员相信在程序中的某个特定点该表达式值为真，可以在任何时候启用和禁用断言验证。”

那么我们在做自动化测试的时候也需要用到断言，自动化测试中寻找元素并进行操作，如果在元素好找的情况下，相信大家都可以较熟练地编写用例脚本，但是如果只是进行操作还不够，有时候还需要对预期结果进行判断。这时就要用到断言了。本文主要简单介绍Python的断言。在前面的文章有分享到[Python的unittest单元测试框架](https://blog.csdn.net/cx243698/article/details/93487217)，这里介绍断言也是unittest的TestCase类提供的常用assert方法。

## 正文

### 语法

> assert expression [, arguments]
> expression是一个表达式，其值应该为True或者False

| 常用方法                  | 用法                                 |
| ------------------------- | ------------------------------------ |
| assertEqual(a, b)         | 判断a==b                             |
| assertNotEqual(a, b)      | 判断a!=b                             |
| assertTrue(x)             | bool(x) is True                      |
| assertFalse(x)            | bool(x) is False                     |
| assertIs(a, b)            | a is b                               |
| assertIsNot(a, b)         | a is not b                           |
| assertIsNone(x)           | x is None                            |
| assertIsNotNone(x)        | x is not None                        |
| assertIn(a, b)            | a in b                               |
| assertNotIn(a, b)         | a not in b                           |
| assertIsInstance(a, b)    | isinstance(a, b)断言a是b的一个实例   |
| assertNotIsInstance(a, b) | not isinstance(a, b)断言a不是b的实例 |

> **1.assertEqual（first，second，msg=None）**
>
> 该方法是测试first和second是否相等。如果值不相等，测试将失败。

> **2.assertNotEqual(first,second,msg=None)**
>
> 该方法是测试first与second不相等，如果值相等，测试将失败。

> **3.assertTrue（expr,msg=None）和assertFalse（expr,msg=None）**
>
> 测试**expr是true**（或false）但当有更具体的方法可以使用时，应避免使用这种方法，例如，assertEqual（a,b）可判断a与b是否相等时，应使用assertEqual（）方法，而不是assertTrue（a==b）因为他们在出现故障时提供更好的错误信息。

## 栗子

> **对用例是否执行成功断言**

```python
# 断言
def assertCheck(self, testName):
    print(testName + '结果为：')
    if 'NG' in GL.flag:
        print('失败')
        return 'NG'
    else:
        print('成功')
        return 'OK'
```

```python
def testcase_001(self):
    u"""测试"""
    RunYaml(self.driver, self._testMethodName).run_yaml(GV.FRONT_DIR + '/testModule/yaml/' + self.ob_name + self._testMethodName + '.yaml')
    self.assertEqual(self.check.assertCheck(self._testMethodName), 'OK')
```

> **对元素是否存在断言**

```python
# 断言 当text == 'true'时，报错不可忽略，否则显示为正确
def check_ele(self, mOperate, cts,i, case_name):
    if mOperate['text'] == 'false':
        try:
            elements_by(mOperate, cts)
            # 当元素存在的时候
            print(mOperate['element_info'] + '存在')
            self.ch.error_shot(case_name,i)
            flag = 'NG'
        except:
            flag = 'OK'
            logging.info(mOperate['element_info'] + '不存在')
    else:
        try:
            elements_by(mOperate, cts)
            flag = 'OK'
            logging.info(mOperate['element_info'] + '存在')
        except:
            flag = 'NG'
            self.ch.error_shot(case_name,i)
            print(mOperate['element_info'] + '应在本页面显示，Fail！')
    GL.flag.append(flag)
```

