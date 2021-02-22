# Python通过future处理并发

### future初识

通过下面代码来对`future`进行一个初步了解：
例子1：普通通过循环的方式



```python
import os
import time
import sys

import requests

POP20_CC = (
    "CN IN US ID BR PK NG BD RU JP MX PH VN ET EG DE IR TR CD FR"
).split()

BASE_URL = 'http://flupy.org/data/flags'

DEST_DIR = 'downloads/'


def save_flag(img,filename):
    path = os.path.join(DEST_DIR,filename)
    with open(path,'wb') as fp:
        fp.write(img)


def get_flag(cc):
    url = "{}/{cc}/{cc}.gif".format(BASE_URL,cc=cc.lower())
    resp = requests.get(url)
    return resp.content


def show(text):
    print(text,end=" ")
    sys.stdout.flush()


def download_many(cc_list):
    for cc in sorted(cc_list):
        image = get_flag(cc)
        show(cc)
        save_flag(image,cc.lower()+".gif")

    return len(cc_list)


def main(download_many):
    t0 = time.time()
    count = download_many(POP20_CC)
    elapsed = time.time()-t0
    msg = "\n{} flags downloaded in {:.2f}s"
    print(msg.format(count,elapsed))


if __name__ == '__main__':
    main(download_many)
```

例子2：通过future方式实现，这里对上面的部分代码进行了复用

```python
from concurrent import futures
import os
import time
import sys

import requests

POP20_CC = (
    "CN IN US ID BR PK NG BD RU JP MX PH VN ET EG DE IR TR CD FR"
).split()

BASE_URL = 'http://flupy.org/data/flags'

DEST_DIR = 'downloads/'


def save_flag(img,filename):
    path = os.path.join(DEST_DIR,filename)
    with open(path,'wb') as fp:
        fp.write(img)

def get_flag(cc):
    url = "{}/{cc}/{cc}.gif".format(BASE_URL,cc=cc.lower())
    resp = requests.get(url)
    return resp.content

def show(text):
    print(text,end=" ")
    sys.stdout.flush()

def main(download_many):
    t0 = time.time()
    count = download_many(POP20_CC)
    elapsed = time.time()-t0
    msg = "\n{} flags downloaded in {:.2f}s"
    print(msg.format(count,elapsed))

MAX_WORKERS = 20

def download_one(cc):
    image = get_flag(cc)
    show(cc)
    save_flag(image, cc.lower()+".gif")
    return cc

def download_many(cc_list):
    workers = min(MAX_WORKERS,len(cc_list))
    with futures.ThreadPoolExecutor(workers) as executor:
        res = executor.map(download_one, sorted(cc_list))

    return len(list(res))


if __name__ == '__main__':
    main(download_many)
```

分别运行三次，两者的平均速度：8.67和1.32s，可以看到差别还是非常大的。

### future

`future`是`concurrent.futures`模块和`asyncio`模块的重要组件
从`python3.4`开始标准库中有两个名为`Future`的类：`concurrent.futures.Future`和`asyncio.Future`
这两个类的作用相同：两个`Future`类的实例都表示可能完成或者尚未完成的延迟计算。与`Twisted`中的`Deferred`类、`Tornado`框架中的`Future`类的功能类似

- 注意：通常情况下自己不应该创建`future`，而是由并发框架(`concurrent.futures`或`asyncio`)实例化
- 原因：`future`表示终将发生的事情，而确定某件事情会发生的唯一方式是执行的时间已经安排好，因此只有把某件事情交给`concurrent.futures.Executor`子类处理时，才会创建`concurrent.futures.Future`实例。
  如：`Executor.submit()`方法的参数是一个可调用的对象，调用这个方法后会为传入的可调用对象排定时间，并返回一个`future`

客户端代码不能应该改变`future`的状态，并发框架在`future`表示的延迟计算结束后会改变期物的状态，我们无法控制计算何时结束。

这两种`future`都有`.done()`方法，这个方法不阻塞，返回值是布尔值，指明`future`链接的可调用对象是否已经执行。客户端代码通常不会询问`future`是否运行结束，而是会等待通知。因此两个`Future`类都有`.add_done_callback()`方法，这个方法只有一个参数，类型是可调用的对象，`future`运行结束后会调用指定的可调用对象。

`.result()`方法是在两个`Future`类中的作用相同：返回可调用对象的结果，或者重新抛出执行可调用的对象时抛出的异常。但是如果`future`没有运行结束，`result`方法在两个`Futrue`类中的行为差别非常大。
对`concurrent.futures.Future`实例来说，调用`.result()`方法会阻塞调用方所在的线程，直到有结果可返回，此时，`result`方法可以接收可选的`timeout`参数，如果在指定的时间内`future`没有运行完毕，会抛出`TimeoutError`异常。
而`asyncio.Future.result`方法不支持设定超时时间，在获取`future`结果最好使用`yield from`结构，但是`concurrent.futures.Future`不能这样做

不管是`asyncio`还是`concurrent.futures.Future`都会有几个函数是返回`future`，其他函数则是使用`future`,在最开始的例子中我们使用的`Executor.map`就是在使用`future`，返回值是一个迭代器，迭代器的`__next__`方法调用各个`future`的`result`方法，因此我们得到的是各个`futrue`的结果，而不是`future`本身

关于`future.as_completed`函数的使用，这里我们用了两个循环，一个用于创建并排定`future`,另外一个用于获取`future`的结果



```python
from concurrent import futures
from flags import save_flag, get_flag, show, main
MAX_WORKERS = 20

def download_one(cc):
    image = get_flag(cc)
    show(cc)
    save_flag(image, cc.lower()+".gif")
    return cc


def download_many(cc_list):
    cc_list = cc_list[:5]
    with futures.ThreadPoolExecutor(max_workers=3) as executor:
        to_do = []
        for cc in sorted(cc_list):
            future = executor.submit(download_one,cc)
            to_do.append(future)
            msg = "Secheduled for {}:{}"
            print(msg.format(cc,future))

        results = []
        for future in futures.as_completed(to_do):
            res = future.result()
            msg = "{}result:{!r}"
            print(msg.format(future,res))
            results.append(res)

    return len(results)


if __name__ == '__main__':
    main(download_many)
```

![img](https://upload-images.jianshu.io/upload_images/7236178-c1db8281cb615da4.png?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

注意：**`Python`代码是无法控制`GIL`，标准库中所有执行阻塞型`IO`操作的函数，在等待操作系统返回结果时都会释放`GIL`.运行其他线程执行，也正是因为这样，`Python`线程可以在`IO`密集型应用中发挥作用**

以上都是`concurrent.futures`启动线程，下面通过它启动进程

### concurrent.futures启动进程

`concurrent.futures`中的`ProcessPoolExecutor`类把工作分配给多个`Python`进程处理，因此，如果需要做`CPU`密集型处理，使用这个模块能绕开`GIL`，利用所有的`CPU`核心。
**其原理是一个`ProcessPoolExecutor`创建了N个独立的`Python`解释器，N是系统上面可用的`CPU`核数**。
使用方法和`ThreadPoolExecutor`方法一样