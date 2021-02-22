# Python中的os.fork()详解

> **额，这里先说明一下，os.fork()只能在Linux系统中使用，在windows中会报错**

### python中使用os模块中的fork创建新的进程

在我们运行`python`程序的时候，系统会生成一个新的`python`进程。

下面为测试代码：



```python
# -*- coding: utf-8 -*-
import time 
time.sleep(20)
```

因为在`python`程序执行完毕后，进程会被销毁，所以这里设置休眠时间20秒，方面看到效果。在`Linux`下执行命令：`python test.py &`

在命令后面加`&`符号，时为了让程序在后台运行。然后输入`ps -l`查看进程，在终端显示如下：

![img](https://upload-images.jianshu.io/upload_images/7236178-2ab5c0c0f63bcb38?imageMogr2/auto-orient/strip|imageView2/2/w/533/format/webp)



##### 使用`fork`方法来创建一个新进程

使用`fork`创建一个新的进程后，新进程是原进程的子进程，原进程为父进程。如果发生错误，则会抛出`OSError`异常。



```python
# -*- coding: utf-8 -*-
import time
import os

try:
    pid = os.fork()
except OSError, e:
    pass

time.sleep(20)
```

运行代码，查看进程，在终端输出如下：



![img](https://upload-images.jianshu.io/upload_images/7236178-3419409cee621ae6?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

可以看出第二条`python`进程就是第一条的子进程。

##### fork出进程后的程序流程

使用`fork`创建子进程后，子进程会复制父进程的数据信息，而后程序就分两个进程继续运行后面的程序，这也是`fork`（分叉）名字的含义了。在子进程内，这个方法会返回`0`；在父进程内，这个方法会返回子进程的编号`PID`。
可以使用`PID`来区分两个进程：



```python
# -*- coding: utf-8 -*-
import time
import os

#创建子进程前声明的变量
number = 7
try:
    pid = os.fork()

    if pid == 0:
        print "this is child process"
        number = number - 1
        time.sleep(5)
        print number
    else:
        print "this is parent process"
except OSError, e:
    pass
```

上面代码中，在子进程创建前，声明了一个变量`number`，然后在子进程中自减`1`，最后打印出`number`的值,显然父进程打印出来的值为`7`，子进程打印出来的值为`6`。为了明显区分父进程和子进程，让子进程睡`3`秒，这样效果就比较明显了。

既然子进程是父进程创建的，那么父进程退出之后，子进程会被`PID`为`1`的进程接管，就是`init`进程了。这样子进程就不会受终端退出影响了，使用这个特性就可以创建在后台执行的程序，俗称守护进程（`daemon`）。

