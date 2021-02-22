# （九）Appium自动化测试之实现错误重跑机制

### **背景**

当我们在使用appium进行自动化测试的时候，报告中必然会出现错误或失败。由于appium经常出现中断，这时我们就需要再次验证用例的正确性。打开项目，连上手机，再次对错误的脚本重新验证，很麻烦是不是？所以我们想，是否可以在运行过程中，出现错误的时候（毕竟appium经常由于识别不到元素而中断）重新运行一次。

## 实现

在之前的一篇文章[传送地址](https://blog.csdn.net/cx243698/article/details/86228874)，我们提到HTMLTestRunner是一个单元测试运行器，它是 Python 标准库的 unittest 模块的一个扩展。它生成易于使用的 HTML 测试报告。所以接下来，我们对HTMLTestRunner进行改造。

**1、根据unittest的运行机制，在stopTest 中判断测试结果，如果失败或出错status为1，判断是否需要重试：**

```python
def stopTest(self, test):
    # Usually one of addSuccess, addError or addFailure would have been called.
    # But there are some path in unittest that would bypass this.
    # We must disconnect stdout in stopTest(), which is guaranteed to be called.
    if self.retry:
        if self.status == 1:
            self.trys += 1
            if self.trys <= self.retry:
                if self.save_last_try:
                    t = self.result.pop(-1)
                    if t[0] == 2:
                        self.error_count = self.error_count - 1
                    else:
                        self.failure_count -= 1
                test = copy.copy(test)
                sys.stderr.write("Retesting... ")
                sys.stderr.write(str(test))
                sys.stderr.write('..%d \n' % self.trys)
                doc = test._testMethodDoc or ''
                if doc.find('_retry') != -1:
                    doc = doc[:doc.find('_retry')]
                desc = "%s_retry:%d" % (doc, self.trys)
                if not PY3K:
                    if isinstance(desc, str):
                        desc = desc.decode("utf-8")
                test._testMethodDoc = desc
                test(self)
             else:
                self.status = 0
                self.trys = 0
    self.complete_output()
```

**2、在实例化HTMLTestRunner 对象时追加参数，retry，指定重试次数**

```python
runner = HTMLTestRunner.HTMLTestRunner(title="APP自动化测试报告", 
                                               description="报告详情", 
                                               tester="ceshi", 
                                               stream=open(self.filename,"wb"),
                                               verbosity=2, 
                                               retry=1)
```

**问题**：加入重跑机制后，报告中会生成case的几条记录数据，类似于下图，这样的报告不利于我们统计分析。

![img](https://img-blog.csdnimg.cn/20190730175610296.png)

 3**、在实例化HTMLTestRunner 对象时追加参数，save_last_try即只保存最后一次报告结果**

```python
runner = HTMLTestRunner.HTMLTestRunner(title="APP自动化测试报告", 
                                               description="报告详情", 
                                               tester="ceshi", 
                                               stream=open(self.filename,"wb"),
                                               verbosity=2, 
                                               retry=1, 
                                               save_last_try=True)
```

## 结果展示

![img](https://img-blog.csdnimg.cn/20190730180412103.png)