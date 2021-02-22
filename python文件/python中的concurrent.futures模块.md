# python中的concurrent.futures模块

### 一、concurrent.futures模块简介

`concurrent.futures` 模块提供了并发执行调用的高级接口

并发可以使用`threads`执行，使用`ThreadPoolExecutor`或 分离的`processes`，使用`ProcessPoolExecutor`。都实现了同一个接口，这个接口在抽象类`Executor`定义

### 二、类的属性和方法

> ```
> concurrent.futures.wait(fs, timeout=None return_when=ALL_COMPLETED)
> ```

```
`wait`等待`fs`里面所有的`Future`实例（由不同的`Executors`实例创建的）完成。返回两个命名元祖，第一个元祖名为`done`，存放完成的`futures`对象，第二个元祖名为`not_done`，存放未完成的`futures`。
`return_when`参数必须是`concurrent.futures`里面定义的常量：`FIRST_COMPLETED`,`FIRST_EXCEPTION`,`ALL_COMPLETED
```

> ```
> concurrent.futures.as_completed(fs, timeout=None)
> ```

返回一个迭代器，`yield`那些完成的`futures`对象。`fs`里面有重复的也只可能返回一次。任何`futures`在调用`as_completed()`调用之前完成首先被`yield`。

### 三、Future对象

`Future()`**封装了可调用对象的异步执行**。**`Future`实例可以被`Executor.submit()`方法创建**。除了测试之外不应该直接创建。`Future`对象可以和异步执行的任务进行交互

- Future方法



```rust
cancel()：尝试去取消调用。如果调用当前正在执行，不能被取消。
          这个方法将返回False，否则调用将会被取消，方法将返回True

cancelled()：如果调用被成功取消返回True

running()：如果当前正在被执行不能被取消返回True

done()：如果调用被成功取消或者完成running返回True

result(Timeout = None)：拿到调用返回的结果。如果没有执行完毕就会去等待

exception(timeout=None)：捕获程序执行过程中的异常

add_done_callback(fn)：将fn绑定到future对象上。当future对象被取消或完成运行时，
                       fn函数将会被调用

'以下的方法是在unitest中'

set_running_or_notify_cancel()

set_result(result)

set_exception(exception) 
```

### 四、Executor对象

1、抽象类，提供异步调用的方法。不能被直接使用，而是通过构建子类。

2、方法

- 提交任务方式一：`submit(fn, *args, **kwargs)`
  调度函数`fn(*args **kwargs)`返回一个`Future`对象代表调用的执行。
- 提交任务方式二：`map(func, *iterables, timeout=None, chunksize=1)`
  和`[map(func, *iterables)]`相似。但是该`map`方法的执行是异步的。多个`func`的调用可以同时执行。当`Executor`对象是 `[ProcessPoolExecutor]`，才可以使用`chunksize`,将`iterable`对象切成块，将其作为分开的任务提交给`pool`，默认为`1`。对于很大的`iterables`，设置较大`chunksize`可以提高性能（切记）。

`shutdown(wait=True)`
给`executor`发信号，使其释放资源，当`futures`完成执行时。已经`shutdown`再调用`submit()`或`map()`会抛出`RuntimeError`。使用`with`语句，就可以避免必须调用本函数

### 五、ThreadPoolExecutor对象

`ThreadPoolExecutor`是`Executor`的子类使用线程池来异步执行调用

如果使用不正确可能会造成死锁，所以`submit`的`task`尽量不要调用`executor`和`futures`，否则很容易出现死锁

- 相互等待的死锁



```python
import time
def wait_on_b():
    time.sleep(5)
    print(b.result())  # b will never complete because it is waiting on a.
    return 5

def wait_on_a():
    time.sleep(5)
    print(a.result())  # a will never complete because it is waiting on b.
    return 6


executor = ThreadPoolExecutor(max_workers=2)
a = executor.submit(wait_on_b)
b = executor.submit(wait_on_a)
```

- 等待自己的结果的死锁



```python
def wait_on_future():
    f = executor.submit(pow, 5, 2)
    # This will never complete because there is only one worker thread and
    # it is executing this function.
    print(f.result())

executor = ThreadPoolExecutor(max_workers=1)
executor.submit(wait_on_future)
```

默认的max_workers是设备的处理器数目*5

### 六、ProcessPoolExecutor对象

`ProcessPoolExecutor`使用`multiprocessing`模块，不受`GIL`锁的约束，意味着只有可以`pickle`的对象才可以执行和返回

`__main__`必须能够被工作子进程导入。所以意味着`ProcessPoolExecutor`在交互式解释器下不能工作。

提交给`ProcessPoolExecutor`的可调用方法里面调用`Executor`或`Future`将会形成死锁。

```
class concurrent.futures.ProcessPoolExecutor(max_workers=None)
```

`max_workers`默认是处理器的个数

- 例子



```python
import concurrent.futures
import math

PRIMES = [
    112272535095293,
    112582705942171,
    112272535095293,
    115280095190773,
    115797848077099,
    115797848077098,
    1099726899285419]


def is_prime(n):
    """
    to judge the input number is prime or not
    :param n: input number
    :return: True or False
    """
    if n % 2 == 0:
        return False

    sqrt_n = int(math.(math.sqrt(n)))
    for i in range(3,sqrt_n + 1, 2):
        if n % i == 0:
            return False
        return True


def main():
    """
    create Process Pool to judge the numbers is prime or not
    :return: None
    """
    with concurrent.futures.ProcessPoolExecutor() as executor:
        for number, prime in zip(PRIMES, executor.map(is_prime, PRIMES)):
            print(number,prime)

if __name__ == '__main__':
    main()
```

### 七、Exception类

```
exception concurrent.futures.CancelledError
exception concurrent.futures.TimeoutError
exception concurrent.futures.process.BrokenProcessPool
```