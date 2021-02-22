# python编程（gevent入门）

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[费晓行](https://feixiaoxing.blog.csdn.net/) 2018-01-14 14:34:38 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 10117 ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) 收藏 14

分类专栏： [python编程](https://blog.csdn.net/feixiaoxing/category_6064878.html)

版权

【 声明：版权所有，欢迎转载，请勿用于商业用途。 联系信箱：feixiaoxing @163.com】

  大家都知道python脚本执行的时候不是很快，特别是python下面的多线程机制，长久以来一直被大家所诟病。所以，很多同学都在思考python下面有没有什么方法可以让python执行地更快一些。其中这些方法包括：1、将复杂的代码转由c完成；2、多进程并发执行；3、用多线程完成io操作等等。另外，这几年，大家讨论最多的大概还是gevent协程机制。

#### 1、协程的基本原理

  gevent的基本原理来自于libevent&libev。熟悉c语言的同学对这么一个lib应该不陌生。本质上libevent或者说libev都是一种事件驱动模型。这种模型对于提高cpu的运行效率，增强用户的并发访问非常有效。但是因为它本身是一种事件机制，所以写起来有点绕，不是很直观。所以，为了修正这个问题，有心人引入了用户侧上下文切换的机制。这就是说，==**如果代码中引入了带io阻塞的代码时，lib本身会自动完成上下文的切换，全程用户都是没有觉察的。这就是gevent的由来。**==

#### 2、gevent安装

  ubuntu下面可以直接用apt-get安装gevent库。

```c
sudo apt-get install python-gevent1
```

#### 3、gevent入门

  为了说明gevent的使用方法，我们可以看一段简单的代码，

```c
import gevent

def f1():
    for i in range(5):
        print 'this is ' + str(i)
        gevent.sleep(0)

def f2():
    for i in range(5):
        print 'that is ' + str(i)
        gevent.sleep(0)

t1 = gevent.spawn(f1)
t2 = gevent.spawn(f2)
gevent.joinall([t1, t2])
12345678910111213141516
```

  通过打印输出，可以看出f1和f2是交叉打印信息的，因为在代码执行的过程中，由用户自己主动调用了切换函数。

#### 4、延时操作

  关于延时，我们只需要将上面的代码修改一下，即将sleep时间变长即可，

```c
import gevent

def f1():
    for i in range(5):
        print 'this is ' + str(i)
        gevent.sleep(3)

def f2():
    for i in range(5):
        print 'that is ' + str(i)
        gevent.sleep(3)

t1 = gevent.spawn(f1)
t2 = gevent.spawn(f2)
gevent.joinall([t1, t2])123456789101112131415
```

#### 5、锁操作

  虽然是协程，但是在里面添加锁增加对共享资源的互斥访问也是非常重要的，此外锁本身的添加也是很简单的，

```c
import gevent
from gevent.lock import Semaphore

sem = Semaphore(1)

def f1():
    for i in range(5):
        sem.acquire()
        print 'this is ' + str(i)
        sem.release()

def f2():
    for i in range(5):
        sem.acquire()
        print 'that is ' + str(i)
        sem.release()

t1 = gevent.spawn(f1)
t2 = gevent.spawn(f2)
gevent.joinall([t1, t2])1234567891011121314151617181920
```

#### 6、延时和锁

  大家可以看一下如果锁和延时一起使用，会怎么样？

```c
import gevent
from gevent.lock import Semaphore

sem = Semaphore(1)

def f1():
    for i in range(5):
        sem.acquire()
        print 'this is ' + str(i)
        sem.release()
        gevent.sleep(2)

def f2():
    for i in range(5):
        sem.acquire()
        print 'that is ' + str(i)
        sem.release()

t1 = gevent.spawn(f1)
t2 = gevent.spawn(f2)
gevent.joinall([t1, t2])123456789101112131415161718192021
```

#### 7、gevent下的monkey机制

  要是gevent说到这里，只能算的上还行。我个人觉得gevent另外一个特别厉害的功能就是它的monkey机制。简单来说，假设你不愿意修改原来已经写好的python代码，但是又想充分利用gevent机制，那么你就可以用monkey来做到这一点。你所要做的就是在文件开头打一个patch，那么它就会自动替换你原来的thread、socket、time、multiprocessing等代码，全部变成gevent框架。这一切都是由gevent自动完成的。注意这个patch是在所有module都import了之后再打，否则没有效果。

```c
from gevent import monkey; monkey.patch_all()
import gevent12
```

#### 8、其他资料

  关于gevent，建议大家多看看英文第一手资料。比如[官网](http://www.gevent.org/contents.html)、还有[这里](http://sdiehl.github.io/gevent-tutorial/#events)。