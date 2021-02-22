# Python协程深入理解

从语法上来看，协程和生成器类似，**都是定义体中包含`yield`关键字的函数**。
`yield`在协程中的用法：

**在协程中`yield`通常出现在表达式的右边，例如：`datum = yield`,可以产出值，也可以不产出--如果`yield`关键字后面没有表达式，那么生成器产出`None`**.
**协程可能从调用方接受数据，调用方是通过`send(datum)`的方式把数据提供给协程使用，而不是`next(...)`函数，通常调用方会把值推送给协程。**
**协程可以把控制器让给中心调度程序，从而激活其他的协程**
所以**总体上在协程中把`yield`看做是控制流程的方式**。

### 了解协程的过程

先通过一个简单的协程的例子理解：



```ruby
>>> def simple_coroutine():
...     print('coroutine started..')
...     x=yield
...     print('coroutine received:',x)
...
>>> coro=simple_coroutine()
>>> coro

结果：<generator object simple_coroutine at 0x00000000025544F8>

>>> next(coro)

结果：coroutine started..

>>> coro.send(12)

结果：coroutine received: 12
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

对上述例子的分析：
`yield` 的右边没有表达式，所以这里默认产出的值是`None`
刚开始先调用了`next(...)`是因为这个时候生成器还没有启动，没有停在`yield`那里，这个时候也是无法通过`send`发送数据。所以当我们通过`next(...)`激活协程后，程序就会运行到`x = yield`，这里有个问题我们需要注意，`x = yield`这个表达式的计算过程是先计算等号右边的内容，然后在进行赋值，所以当激活生成器后，**程序会停在yield这里，但并没有给x赋值**。
当我们调用`send`方法后`yield`会收到这个值并赋值给`x`,而当程序运行到协程定义体的末尾时和用生成器的时候一样会抛出`StopIteration`异常

**如果协程没有通过`next(...)`激活(同样我们可以通过`send(None)`的方式激活)，但是我们直接`send`，会提示如下错误**：

![img](https://upload-images.jianshu.io/upload_images/7236178-98d7a93ae4ed336e.png?imageMogr2/auto-orient/strip|imageView2/2/w/982/format/webp)

关于调用`next(...)`函数这一步通常称为”预激(`prime`)“协程，即让协程向前执行到第一个`yield`表达式，准备好作为活跃的协程使用

协程在运行过程中有四个状态：

> `GEN_CREATED`:等待开始执行
> `GEN_RUNNING`:解释器正在执行，这个状态一般看不到
> `GEN_SUSPENDED`:在yield表达式处暂停
> `GEN_CLOSED`:执行结束

#### 通过下面例子来查看协程的状态：



```ruby
>>> def simple_coro2(a):
...     print('->Started:a=',a)
...     b = yield a
...     print('->Received:b=',b)
...     c = yield a+b
...     print('->Received:c=',c)
...
>>> my_coro2 = simple_coro2(666)
>>> from inspect import getgeneratorstate
>>> getgeneratorstate(my_coro2)

'GEN_CREATED'

>>> next(my_coro2)

->Started:a= 666
666

>>> getgeneratorstate(my_coro2)

'GEN_SUSPENDED'

>>> my_coro2.send(121)

->Received:b= 121
787

>>> my_coro2.send(212)

->Received:c= 212
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration

>>> getgeneratorstate(my_coro2)

'GEN_CLOSED'
```

##### 接着再通过一个计算平均值的例子来继续理解：



```python
>>> def averager():
...     total = 0.0
...     count = 0
...     average = None
...     while True:
...             term = yield average
...             total += term
...             count += 1
...             average = total/count
...
>>> coro_avg = averager()
>>> next(coro_avg)
>>> coro_avg.send(10)
10.0
>>> coro_avg.send(30)
20.0
>>> coro_avg.send(40)
26.666666666666668
```

这里是一个死循环，只要不停`send`值给协程，可以一直计算下去。
通过上面的几个例子我们发现，我们如果想要开始使用协程的时候必须通过`next(...)`方式激活协程，如果不预激，这个协程就无法使用，如果哪天在代码中遗忘了那么就出问题了，所以有一种预激协程的装饰器，可以帮助我们干这件事

### 预激协程的装饰器

下面是预激装饰器的演示例子：



```python
from functools import wraps

def coroutine(func):
    @wraps(func)
    def primer(*args,**kwargs):
        gen = func(*args,**kwargs)
        next(gen)
        return gen
    return primer


@coroutine
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count


coro_avg = averager()
from inspect import getgeneratorstate
print(getgeneratorstate(coro_avg))
print(coro_avg.send(10))
print(coro_avg.send(30))
print(coro_avg.send(5))
```

> 关于预激，在使用`yield from`句法调用协程的时候，会自动预激活，这样其实与我们上面定义的`coroutine`装饰器是不兼容的，在`python3.4`里面的`asyncio.coroutine`装饰器不会预激协程，因此兼容`yield from`

#### 终止协程和异常处理

**协程中未处理的异常会向上冒泡,传给`next`函数或`send`函数的调用方(即触发协程的对象)**
拿上面的代码举例子，如果我们发送了一个字符串而不是一个整数的时候就会报错，并且这个时候协程是被终止了



```python
>>> def averager():
...     total = 0.0
...     count = 0
...     average = None
...     while True:
...             term = yield average
...             total += term
...             count += 1
...             average = total/count
...
>>> coro_avg = averager()
>>> next(coro_avg)
>>> coro_avg.send(10)
10.0
>>> coro_avg.send(30)
20.0
>>> coro_avg.send(40)
26.666666666666668
>>> coro_avg.send('a')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 7, in averager
TypeError: unsupported operand type(s) for +=: 'float' and 'str'
>>> getgeneratorstate(coro_avg)
'GEN_CLOSED'
```

从python2.5开始客户端代码在生成器对象上调用两个方法，显示的把异常发送给协程
分别为：`throw`和`close`

> `generator.throw`:会让生成器在暂停的`yield`表达式处抛出指定的异常，如果生成器处理了抛出的异常，代码会向前执行到下一个`yield`表达式，而产出的值会成为调用`generator.throw`方法代码的返回值。如果生成器没有处理抛出的异常，异常会向上冒泡，传到调用方的上下文中。
> `generator.close`:会让生成器在暂停的`yield`表达式处抛出`GeneratorExit`异常。如果生成器没有处理这个异常，或者抛出了`StopIteration`异常，调用方不会报错，如果收到`GeneratorExit`异常，生成器一定不能产出值，否则解释器会抛出`RuntimeError`异常。生成器抛出的异常会向上冒泡，传给调用方。

##### 下面是一个例子：



```ruby
>>> class DemoException(Exception):
...     '''自定义异常'''
...
>>> def demo_exc_handling():
...     print('->coroutine started')
...     while True:
...             try:
...                     x=yield
...             except DemoException:
...                     print('DemoException handled.')
...             else:
...                     print('-> coroutine received:{!r}'.format(x))
...     raise RuntimeError('this line should never run.')
...
>>> exc_coro = demo_exc_handling()
>>> next(exc_coro)
->coroutine started
>>> exc_coro.send(1)
-> coroutine received:1
>>> exc_coro.send(2)
-> coroutine received:2
>>> exc_coro.throw(DemoException)
DemoException handled.
>>> exc_coro.send(3)
-> coroutine received:3
>>> exc_coro.throw(TypeError)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in demo_exc_handling
TypeError
>>> from inspect import getgeneratorstate
>>> getgeneratorstate(exc_coro)
'GEN_CLOSED'
```

> **当传入我们定义的异常时不会影响协程，协程不会停止，可以继续send,但是如果是没有处理的异常的时候，就会报错，并且协程会被终止**

#### 让协程返回值



```python
from collections import namedtuple

Result = namedtuple("Result","colunt average")

def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count+=1
        average = total/count
    return Result(count,average)

coro_avg = averager()
next(coro_avg)
coro_avg.send(10)
coro_avg.send(30)
coro_avg.send(5)
try:
    coro_avg.send(None)
except StopIteration as e:
    result = e.value
    print(result)
```

这样就可以获取到最后的结果：**Result(count=3,average=15.0)**

其实相对来说上面这种方式获取返回值比较麻烦，而**`yield from`结构会自动捕获`StopIteration`异常，这种处理方式与`for`循环处理`StopIteration`异常的方式一样，循环机制使我们更容易理解处理异常，对于`yield from`来说，解释器不仅会捕获`StopIteration`异常，还会把`value`属性的值变成`yield from`表达式的值**

### 关于yield from

**在生成器`gen`中使用`yield from subgen()`时，`subgen`会获得控制权，把产出的值传给`gen`的调用方，即调用方可以直接控制`subgen`,同时，`gen`会阻塞，等待`subgen`终止**

`yield from x`表达式对`x`对象所做的第一件事是，调用`iter(x)`,从中获取迭代器，因此`x`可以是任何可迭代的对象

下面是`yield from`可以简化`yield`表达式的例子：

```python
def gen():
    for c in "AB":
        yield c
    for i in range(1,3):
        yield i

print(list(gen()))

def gen2():
    yield from "AB"
    yield from range(1,3)

print(list(gen2()))
```

这两种的方式的结果是一样的，但是这样看来`yield from`更加简洁，但是`yield from`的作用可不仅仅是替代产出值的嵌套`for`循环。
**`yield from`的主要功能是打开双向通道**，**把最外层的调用方与最内层的子生成器连接起来**，**这样二者可以直接发送和产出值，还可以直接传入异常，而不用再像之前那样在位于中间的协程中添加大量处理异常的代码**

通过`yield from`还可以链接可迭代对象



![img](https://upload-images.jianshu.io/upload_images/7236178-f48f6dd5b80696ad.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)


委派生成器在`yield from` 表达式处暂停时，调用方可以直接把数据发给子生成器，子生成器再把产出产出值发给调用方，子生成器返回之后，解释器会抛出`StopIteration`异常，并把返回值附加到异常对象上，此时委派生成器会恢复。



下面是一个完整的例子代码



```python
from collections import namedtuple

Result = namedtuple('Result', 'count average')
# 子生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)

# 委派生成器
def grouper(result, key):
    while True:
        result[key] = yield from averager()

# 客户端代码，即调用方
def main(data):
    results = {}
    for key,values in data.items():
        group = grouper(results,key)
        next(group)
        for value in values:
            group.send(value)
        group.send(None) #这里表示要终止了
    report(results)

# 输出报告
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print('{:2} {:5} averaging {:.2f}{}'.format(
            result.count, group, result.average, unit
        ))

data = {
    'girls;kg':
        [40.9, 38.5, 44.3, 42.2, 45.2, 41.7, 44.5, 38.0, 40.6, 44.5],
    'girls;m':
        [1.6, 1.51, 1.4, 1.3, 1.41, 1.39, 1.33, 1.46, 1.45, 1.43],
    'boys;kg':
        [39.0, 40.8, 43.2, 40.8, 43.1, 38.6, 41.4, 40.6, 36.3],
    'boys;m':
        [1.38, 1.5, 1.32, 1.25, 1.37, 1.48, 1.25, 1.49, 1.46],
}

if __name__ == '__main__':
    main(data)
```

关于上述代码着重解释一下关于委派生成器部分，这里的循环每次迭代时会新建一个`averager`实例，每个实例都是作为协程使用的生成器对象。

> (委派器中的`while True`的作用：由于拥有`yield from`的委派器 本质还是一个生成器，所以当`yield from`后的表达式执行完毕之后，若后面没有`yield`会报错`StopIteration`则`result[key]`无法接收返回值，所以使用循环或者也可以在`yield from`之后加`yield`)
> 例如：



```python
# 委派生成器
def grouper(result, key):
    #while True:
    result[key] = yield from averager()
    yield 
```

##### 效果一样

`grouper`发送的每个值都会经由`yield from`处理，通过管道传给`averager`实例。`grouper`会在`yield from`表达式处暂停，等待`averager`实例处理客户端发来的值。`averager`实例运行完毕后，返回的值会绑定到`results[key]`上，`while` 循环会不断创建`averager`实例，处理更多的值

并且上述代码中的子生成器可以使用return 返回一个值，而返回的值会成为yield from表达式的值。

#### 关于yield from的意义

> 关于yield from 六点重要的说明：

1.子生成器产出的值都直接传给委派生成器的调用方(即客户端代码)

2.使用`send()`方法发送给委派生成器的值都直接传给子生成器。如果发送的值为`None`,那么会给委派调用子生成器的`__next__()`方法。如果发送的值不是`None`,那么会调用子生成器的`send`方法，如果调用的方法抛出`StopIteration`异常，那么委派生成器恢复运行，任何其他异常都会向上冒泡，传给委派生成器

3.生成器退出时，生成器(或子生成器)中的`return expr`表达式会出发`StopIteration(expr)`异常抛出

4.`yield from`表达式的值是子生成器终止时传给`StopIteration`异常的第一个参数。`yield from` 结构的另外两个特性与异常和终止有关。

5.传入委派生成器的异常，除了`GeneratorExit`之外都传给子生成器的`throw()`方法。如果调用`throw()`方法时抛出`StopIteration`异常，委派生成器恢复运行。`StopIteration`之外的异常会向上冒泡，传给委派生成器

6.如果把`GeneratorExit`异常传入委派生成器，或者在委派生成器上调用`close()`方法，那么在子生成器上调用`clsoe()`方法，如果它有的话。如果调用`close()`方法导致异常抛出，那么异常会向上冒泡，传给委派生成器，否则委派生成器抛出`GeneratorExit`异常