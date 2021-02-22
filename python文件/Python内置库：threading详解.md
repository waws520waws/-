# Python内置库：threading详解

`Python`的线程操作在旧版本中使用的是`thread`模块，在`Python2.7`和`Python3`中引入了`threading`模块，同时`thread`模块在`Python3`中改名为`_thread`模块，`threading`模块相较于`thread`模块，对于线程的操作更加的丰富，而且`threading`模块本身也是相当于对`thread`模块的进一步封装而成，`thread`模块有的功能`threading`模块也都有，所以涉及到对线程的操作，推荐使用`threading`模块。

`threading`模块中包含了关于线程操作的丰富功能，包括：

> 常用线程函数，线程对象，锁对象，递归锁对象，事件对象，条件变量对象，信号量对象，定时器对象，栅栏对象。

注：本文使用的`Python`版本是`Python3.6`

### 一、with语法

这个模块中所有带有`acquire()`和`release()`方法的对象，都可以使用`with`语句。当进入`with`语句块时，`acquire()`方法被自动调用，当离开`with`语句块时，`release()`语句块被自动调用。包括`Lock`、`RLock`、`Condition`、`Semaphore`。

以下语句：



```python
with some_lock:
    # do something
    pass
```

相当于：



```python
some_lock.acquire()
try:
    # do something
    pass
finally:
    some_lock.release()
```

### 二、threading函数

在`Python3`中方法名和函数名统一成了以字母小写加下划线的命令方式，但是`Python2.x`中`threading`模块的某些以驼峰命名的方法和函数仍然可用，如`threading.active_count()`和`threading.activeCount()`是一样的。

通常情况下，`Python`程序启动时，`Python`解释器会启动一个继承自`threading.Thread`的`threading._MainThread`线程对象作为主线程，所以涉及到`threading.Thread`的方法和函数时通常都算上了这个主线程的，比如在启动程序时打印`threading.active_count()`的结果就已经是1了。

> - `threading.active_count()`：返回当前存活的`threading.Thread`线程对象数量，等同于`len(threading.enumerate())`。

> - `threading.current_thread()`：返回此函数的调用者控制的`threading.Thread`线程对象。如果当前调用者控制的线程不是通过`threading.Thread`创建的，则返回一个功能受限的虚拟线程对象。

> - `threading.get_ident()`：返回当前线程的线程标识符。注意当一个线程退出时，它的线程标识符可能会被之后新创建的线程复用。
>   `threading.enumerate()`：返回当前存活的`threading.Thread`线程对象列表。

> - `threading.main_thread()`：返回主线程对象，通常情况下，就是程序启动时`Python`解释器创建的`threading._MainThread`线程对象。

> - `threading.stack_size([size])`：返回创建线程时使用的堆栈大小。也可以使用可选参数`size`指定之后创建线程时的堆栈大小，`size`可以是0或者一个不小于32KiB的正整数。如果参数没有指定，则默认为0。如果系统或者其他原因不支持改变堆栈大小，则会报`RuntimeError`错误；如果指定的堆栈大小不合法，则会报`ValueError`，但并不会修改这个堆栈的大小。32KiB是保证能解释器运行的最小堆栈大小，当然这个值会因为系统或者其他原因有限制，比如它要求的值是大于32KiB的某个值，只需根据要求修改即可。

### 三、threading常量

`threading.TIMEOUT_MAX`：指定阻塞函数（如`Lock.acquire()`、`RLock.acquire()`、`Condition.wait()`等）中参数`timeout`的最大值，在给这些阻塞函数传参时如果超过了这个指定的最大值会抛出`OverflowError`错误。

### 四、线程对象：threading.Thread

`threading.Thread`目前还没有优先级和线程组的功能，而且创建的线程也不能被销毁、停止、暂定、恢复或中断。

##### 守护线程：

> 只有所有守护线程都结束，整个`Python`程序才会退出，但并不是说`Python`程序会等待守护线程运行完毕，相反，当程序退出时，如果还有守护线程在运行，程序会去强制终结所有守护线程，当守所有护线程都终结后，程序才会真正退出。可以通过修改`daemon`属性或者初始化线程时指定`daemon`参数来指定某个线程为守护线程。

##### 非守护线程：

> 一般创建的线程默认就是非守护线程，包括主线程也是，即在`Python`程序退出时，如果还有非守护线程在运行，程序会等待直到所有非守护线程都结束后才会退出。
>
> - 注：守护线程会在程序关闭时突然关闭（如果守护线程在程序关闭时还在运行），它们占用的资源可能没有被正确释放，比如正在修改文档内容等，需要谨慎使用。

#### Thread对象主要属性和方法

> ### `threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)`
>
> 如果这个类的初始化方法被重写，请确保在重写的初始化方法中做任何事之前先调用`threading.Thread`类的`__init__`方法。
>
> - `group`：应该设为`None`，即不用设置，使用默认值就好，因为这个参数是为了以后实现`ThreadGroup`类而保留的。
> - `target`：在`run`方法中调用的可调用对象，即需要开启线程的可调用对象，比如函数或方法。
> - `name`：线程名称，默认为“`Thread-N`”形式的名称，`N`为较小的十进制数。
> - `args`：在参数`target`中传入的可调用对象的参数元组，默认为空元组`()`。
> - `kwargs`：在参数`target`中传入的可调用对象的关键字参数字典，默认为空字典`{}`。
> - `daemon`：默认为`None`，即继承当前调用者线程（即开启线程的线程，一般就是主线程）的守护模式属性，如果不为`None`，则无论该线程是否为守护模式，都会被设置为“守护模式”。
> - `start()`：开启线程活动。它将使得`run()`方法在一个独立的控制线程中被调用，**需要注意的是同一个线程对象的`start()`方法只能被调用一次，如果调用多次，则会报`RuntimeError`错误**。
> - `run()`：此方法代表线程活动。
> - `join(timeout=None)`：让当前调用者线程（即开启线程的线程，一般就是主线程）等待，直到线程结束（无论它是什么原因结束的），`timeout`参数是以秒为单位的浮点数，用于设置操作超时的时间，返回值为`None`。如果想要判断线程是否超时，只能通过线程的`is_alive`方法来进行判断。`join`方法可以被调用多次。**如果对当前线程使用`join`方法（即线程在内部调用自己的`join`方法），或者在线程没有开始前使用`join`方法，都会报`RuntimeError`错误**。
> - `name`：线程的名称字符串，并没有什么实际含义，多个线程可以赋予相同的名称，初始值由初始化方法来设置。
> - `ident`：线程的标识符，如果线程还没有启动，则为`None`。`ident`是一个非零整数，参见`threading.get_ident()`函数。当线程结束后，它的`ident`可能被其他新创建的线程复用，当然就算该线程结束了，它的`ident`依旧是可用的。
> - `is_alive()`：线程是否存活，返回`True`或者`False`。在线程的`run()`运行之后直到`run()`结束，该方法返回`True`。
> - `daemon`：表示该线程是否是守护线程，`True`或者`False`。设置一个线程的`daemon`必须在线程的`start()`方法之前，否则会报`RuntimeError`错误。这个值默认继承自创建它的线程，主线程默认是非守护线程的，所以在主线程中创建的线程默认都是非守护线程的，即`daemon=False`。

##### 使用threading.Thread类创建线程简单示例：



```python
"""
通过实例化threading.Thread类创建线程
"""
import time
import threading


def test_thread(para='hi', sleep=3):
    """线程运行函数"""
    time.sleep(sleep)
    print(para)


def main():
    # 创建线程
    thread_hi = threading.Thread(target=test_thread)
    thread_hello = threading.Thread(target=test_thread, args=('hello', 1))
    # 启动线程
    thread_hi.start()
    thread_hello.start()
    print('Main thread has ended!')


if __name__ == '__main__':
    main()
```

结果：



```undefined
Main thread has ended!
hello
hi
```

##### 使用threading.Thread类的子类创建线程简单示例：



```python
"""
通过继承threading.Thread的子类创建线程
"""
import time
import threading


class TestThread(threading.Thread):
    def __init__(self, para='hi', sleep=3):
        # 重写threading.Thread的__init__方法时，确保在所有操作之前先调用threading.Thread.__init__方法
        super().__init__()
        self.para = para
        self.sleep = sleep

    def run(self):
        """线程内容"""
        time.sleep(self.sleep)
        print(self.para)


def main():
    # 创建线程
    thread_hi = TestThread()
    thread_hello = TestThread('hello', 1)
    # 启动线程
    thread_hi.start()
    thread_hello.start()
    print('Main thread has ended!')


if __name__ == '__main__':
    main()
```

结果：



```undefined
Main thread has ended!
hello
hi
```

##### join方法简单示例：



```python
"""
使用join方法阻塞主线程
"""
import time
import threading


def test_thread(para='hi', sleep=5):
    """线程运行函数"""
    time.sleep(sleep)
    print(para)


def main():
    # 创建线程
    thread_hi = threading.Thread(target=test_thread)
    thread_hello = threading.Thread(target=test_thread, args=('hello', 1))
    # 启动线程
    thread_hi.start()
    thread_hello.start()
    time.sleep(2)
    print('马上执行join方法了')
    # 执行join方法会阻塞调用线程（主线程），直到调用join方法的线程（thread_hi）结束
    thread_hi.join()
    print('线程thread_hi已结束')
    # 这里不会阻塞主线程，因为运行到这里的时候，线程thread_hello已经运行结束了
    thread_hello.join()
    print('Main thread has ended!')

    # 以上代码只是为了展示join方法的效果
    # 如果想要等所有线程都运行完成后再做其他操作，可以使用for循环
    # for thd in (thread_hi, thread_hello):
    #     thd.join()
    #
    # print('所有线程执行结束后的其他操作')


if __name__ == '__main__':
    main()
```

结果：



```csharp
hello
马上执行join方法了
hi
线程thread_hi已结束
Main thread has ended!
```

### 五、锁对象：threading.Lock

`threading.Lock`是直接通过`_thread`模块扩展实现的。

当锁在被锁定时，它并不属于某一个特定的线程。

锁只有“锁定”和“非锁定”两种状态，当锁被创建时，是处于“非锁定”状态的。当锁已经被锁定时，其他线程再次调用`acquire()`方法会被阻塞执行，直到锁被获得锁的线程调用`release()`方法释放掉锁并将其状态改为“非锁定”。

同一个线程获取锁后，如果在释放锁之前再次获取锁会导致当前线程阻塞，除非有另外的线程来释放锁，如果只有一个线程，并且发生了这种情况，会导致这个线程一直阻塞下去，即形成了死锁。所以在获取锁时需要保证锁已经被释放掉了，或者使用递归锁(`RLock`)来解决这种情况。

- `acquire(blocking=True, timeout=-1)`：
  获取锁，并将锁的状态改为“锁定”，成功返回`True`，失败返回`False`。当一个线程获得锁时，会阻塞其他尝试获取锁的线程，直到这个锁被释放掉。`timeout`默认值为`-1`，即将无限阻塞等待直到获得锁，如果设为其他的值时（单位为秒的浮点数），将最多阻塞等待`timeout`指定的秒数。当`blocking`为`False`时，`timeout`参数被忽略，即没有获得锁也不进行阻塞。
- `release()`：
  释放一个锁，并将其状态改为“非锁定”，需要注意的是任何线程都可以释放锁，不只是获得锁的线程（因为锁不属于特定的线程）。`release()`方法只能在锁处于“锁定”状态时调用，如果在“非锁定”状态时调用则会报`RuntimeError`错误。

##### 使用锁实现线程同步的简单示例：



```python
"""
使用锁实现线程同步
"""
import time
import threading

# 创建锁
lock = threading.Lock()

# 全局变量
global_resource = [None] * 5


def change_resource(para, sleep):
    # 请求锁
    lock.acquire()

    # 这段代码如果不加锁，第一个线程运行结束后global_resource中是乱的，输出为：修改全局变量为： ['hello', 'hi', 'hi', 'hello', 'hello']
    # 第二个线程运行结束后，global_resource中还是乱的，输出为：修改全局变量为： ['hello', 'hi', 'hi', 'hi', 'hi']
    global global_resource
    for i in range(len(global_resource)):
        global_resource[i] = para
        time.sleep(sleep)
    print("修改全局变量为：", global_resource)

    # 释放锁
    lock.release()


def main():
    thread_hi = threading.Thread(target=change_resource, args=('hi', 2))
    thread_hello = threading.Thread(target=change_resource, args=('hello', 1))
    thread_hi.start()
    thread_hello.start()


if __name__ == '__main__':
    main()
```

结果：



```bash
修改全局变量为： ['hi', 'hi', 'hi', 'hi', 'hi']
修改全局变量为： ['hello', 'hello', 'hello', 'hello', 'hello']
```

### 六、递归锁对象：threading.RLock

递归锁和普通锁的差别在于加入了“所属线程”和“递归等级”的概念，释放锁必须有获取锁的线程来进行释放，同时，同一个线程在释放锁之前再次获取锁将不会阻塞当前线程，只是在锁的递归等级上加了1（获得锁时的初始递归等级为1）。

使用普通锁时，对于一些可能造成死锁的情况，可以考虑使用递归锁来解决。

- `acquire(blocking=True, timeout=-1)`：
  与普通锁的不同之处在于：当使用默认值时，如果这个线程已经拥有锁，那么锁的递归等级加1。线程获得锁时，该锁的递归等级被初始化为1。当多个线程被阻塞时，只有一个线程能在锁被解时获得锁，这种情况下，`acquire()`是没有返回值的。
- `release()`：
  没有返回值，调用一次则递归等级减1，递归等级为零时表示这个线程的锁已经被释放掉，其他线程可以获取锁了。可能在一个线程中调用了多次`acquire()`，导致锁的递归等级大于了1，那么就需要调用对应次数的`release()`来完全释放锁，并将它的递归等级减到零，其他的线程才能获取锁，不然就会一直被阻塞着。

##### 递归锁使用简单示例：



```python
"""
在普通锁中可能造成死锁的情况，可以考虑使用递归锁解决
"""
import time
import threading


# 如果是使用的两个普通锁，那么就会造成死锁的情况，程序一直阻塞而不会退出
# rlock_hi = threading.Lock()
# rlock_hello = threading.Lock()

# 使用成一个递归锁就可以解决当前这种死锁情况
rlock_hi = rlock_hello = threading.RLock()


def test_thread_hi():
    # 初始时锁内部的递归等级为1
    rlock_hi.acquire()
    print('线程test_thread_hi获得了锁rlock_hi')
    time.sleep(2)
    # 如果再次获取同样一把锁，则不会阻塞，只是内部的递归等级加1
    rlock_hello.acquire()
    print('线程test_thread_hi获得了锁rlock_hello')
    # 释放一次锁，内部递归等级减1
    rlock_hello.release()
    # 这里再次减，当递归等级为0时，其他线程才可获取到此锁
    rlock_hi.release()


def test_thread_hello():
    rlock_hello.acquire()
    print('线程test_thread_hello获得了锁rlock_hello')
    time.sleep(2)
    rlock_hi.acquire()
    print('线程test_thread_hello获得了锁rlock_hi')
    rlock_hi.release()
    rlock_hello.release()


def main():
    thread_hi = threading.Thread(target=test_thread_hi)
    thread_hello = threading.Thread(target=test_thread_hello)
    thread_hi.start()
    thread_hello.start()


if __name__ == '__main__':
    main()
```

结果:



```undefined
线程test_thread_hi获得了锁rlock_hi
线程test_thread_hi获得了锁rlock_hello
线程test_thread_hello获得了锁rlock_hello
线程test_thread_hello获得了锁rlock_hi
```

### 七、条件变量对象：threading.Condition

它的`wait()`方法释放锁，并阻塞程序直到其他线程调用`notify()`或者`notify_all()`方法唤醒，然后`wait()`方法重新获取锁，这个方法也可以指定`timeout`超时时间。
它的`notify()`方法唤醒一个正在等待的线程，`notify_all()`则是唤醒所有正在等待的线程。`notify()`或者`notify_all()`并不会释放锁，所以被唤醒的线程并不会立即从它们的`wait()`方法出返回并执行，只有在调用了`notify()`或者`notify_all()`方法的线程放弃了锁的所有权后才会返回对应的线程并执行，即先通知再释放锁。

`threading.Condition(lock=None)`：一个条件变量对象允许一个或多个线程等待，直到被另一个线程通知。`lock`参数必须是一个`Lock`对象或者`RLock`对象，并且会作为底层锁使用，默认使用`RLock`。

- `acquire(*args)`：
  请求底层锁。此方法调用底层锁对应的方法和返回对应方法的返回值。
- `release()`：
  释放底层锁。此方法调用底层所对应的方法，没有返回值。
- `wait(timeout=None)`：
  释放锁，等待直到被通知（再获取锁）或者发生超时事件。如果线程在调用此方法时本身并没有锁（即线程首先得有锁），则会报`RuntimeError`错误。这个方法释放底层锁，然后阻塞线程，直到另一个线程中的同一个条件变量使用`notify()`或`notify_all()`唤醒，或者超时事件发生，一旦被唤醒或者超时，则会重新去获取锁并返回（成功返回`True`，否则返回`False`）。`timeout`参数为浮点类型的秒数。在`RLock`中使用一次`release`方法，可能并不能释放锁，因为锁可能被`acquire()`了多次，但是在条件变量对象中，它调用了`RLock`类的内部方法，可以一次就完全释放锁，重新获取锁时也会重置锁的递归等级。
- `wait_for(predicate, timeout=None)`：
  与wait方法相似，等待，直到条件计算为`True`，返回最后一次的`predicate`的返回值。`predicate`参数为一个返回值为布尔值的可调用对象。调用此方法的时候会先调用`predicate`对象，如果返回的就是`True`，则不会释放锁，直接往后执行。另一个线程通知后，在它释放锁时，才会触发`wait_for`方法等待事件，这时如果`predicate`结果为`True`，则尝试获取锁，获取成功后则继续往后执行，如果为`False`，则会一直阻塞下去。此方法如果忽略`timeout`参数，就相当于：



```bash
while not predicate(): 
    condition_lock.wait()
```

- `notify(n=1)`：
  唤醒一个等待这个条件的线程，如果调用这个方法的线程在没有获得锁的情况下调用这个方法，会报`RuntimeError`错误。默认唤醒一个线程，可以通过参数`n`设置唤醒`n`个正在等待这个条件变量的线程，如果没有线程在等待，调用这个方法不会发生任何事。如果等待的线程中正好有`n`个线程，那么这个方法可以准确的唤醒这`n`个线程，但是等待的线程超过指定的`n`个，有时候可能会唤醒超过`n`个的线程，所以依赖参数`n`是不安全的行为。
- `notify_all()`：
  唤醒所有等待这个条件的线程。这个方法与`notify()`不同之处在于它唤醒所有线程，而不是特定`n`个。

##### 条件变量简单示例：



```python
"""
让一个线程等待，直到另一个线程通知
"""
import time
import threading


# 创建条件变量对象
condition_lock = threading.Condition()

PRE = 0


# predicate可调用函数
def pre():
    print(PRE)
    return PRE


def test_thread_hi():
    # 在使用wait/wait_for之前必须先获得锁
    condition_lock.acquire()

    print('等待线程test_thread_hello的通知')
    # 先执行一次pre，返回False后释放掉锁，等另一个线程释放掉锁后再次执行pre，返回True后再次获取锁
    # wait_for的返回值不是True和False，而是predicate参数的返回值
    condition_lock.wait_for(pre)
    # condition_lock.wait()
    print('继续执行')

    # 不要忘记使用wait/wait_for之后要释放锁
    condition_lock.release()


def test_thread_hello():
    time.sleep(1)
    condition_lock.acquire()

    global PRE
    PRE = 1
    print('修改PRE值为1')

    print('通知线程test_thread_hi可以准备获取锁了')
    condition_lock.notify()
    
    # 先notify/notify_all之后在释放锁
    condition_lock.release()
    print('你获取锁吧')


def main():
    thread_hi = threading.Thread(target=test_thread_hi)
    thread_hello = threading.Thread(target=test_thread_hello)
    thread_hi.start()
    thread_hello.start()


if __name__ == '__main__':
    main()
```

结果：



```undefined
等待线程test_thread_hello的通知
0
修改PRE值为1
通知线程test_thread_hi可以准备获取锁了
你获取锁吧
1
继续执行
```

### 八、信号量对象：threading.Semaphore

一个信号量管理一个内部计数器，`acquire()`方法会减少计数器，`release()`方法则增加计数器，计数器的值永远不会小于零，当调用`acquire()`时，如果发现该计数器为零，则阻塞线程，直到调用`release()`方法使计数器增加。

- `threading.Semaphore(value=1)`：
  value参数默认值为1，如果指定的值小于0，则会报`ValueError`错误。一个信号量对象管理一个原子性的计数器，代表`release()`方法调用的次数减去`acquire()`方法的调用次数，再加上一个初始值。
- `acquire(blocking=True, timeout=None)`：
  默认情况下，在进入时，如果计数器大于0，则减1并返回`True`，如果等于0，则阻塞直到使用`release()`方法唤醒，然后减1并返回`True`。被唤醒的线程顺序是不确定的。如果`blocking`设置为`False`，调用这个方法将不会发生阻塞。`timeout`用于设置超时的时间，在`timeout`秒的时间内没有获取到信号量，则返回`False`，否则返回`True`。
- `release()`：释放一个信号量，将内部计数器增加1。当计数器的值为0，且有其他线程正在等待它大于0时，唤醒这个线程。

##### 信号量对象简单示例：



```python
"""
通过信号量对象管理一次性运行的线程数量
"""
import time
import threading

# 创建信号量对象，初始化计数器值为3
semaphore3 = threading.Semaphore(3)


def thread_semaphore(index):
    # 信号量计数器减1
    semaphore3.acquire()
    time.sleep(2)
    print('thread_%s is running...' % index)
    # 信号量计数器加1
    semaphore3.release()


def main():
    # 虽然会有9个线程运行，但是通过信号量控制同时只能有3个线程运行
    # 第4个线程启动时，调用acquire发现计数器为0了，所以就会阻塞等待计数器大于0的时候
    for index in range(9):
        threading.Thread(target=thread_semaphore, args=(index, )).start()


if __name__ == '__main__':
    main()
```

### 九、事件对象：threading.Event

一个事件对象管理一个内部标志，初始状态默认为`False`，`set()`方法可将它设置为`True`，`clear()`方法可将它设置为`False`，`wait()`方法将线程阻塞直到内部标志的值为`True`。

如果一个或多个线程需要知道另一个线程的某个状态才能进行下一步的操作，就可以使用线程的`event`事件对象来处理。

- `is_set()`：当内部标志为`True`时返回`True`。
- `set()`：设置内部标志为`True`。此时所有等待中的线程将被唤醒，调用`wait()`方法的线程将不会被阻塞。
- `clear()`：将内部标志设置为`False`。所有调用`wait()`方法的线程将被阻塞，直到调用`set()`方法将内部标志设置为`True`。
- `wait(timeout=None)`：阻塞线程直到内部标志为`True`，或者发生超时事件。如果调用时内部标志就是`True`，那么不会被阻塞，否则将被阻塞。`timeout`为浮点类型的秒数。

##### 事件对象简单示例：



```python
"""
事件对象使用实例
"""
import time
import threading

# 创建事件对象，内部标志默认为False
event = threading.Event()


def student_exam(student_id):
    print('学生%s等监考老师发卷。。。' % student_id)
    event.wait()
    print('开始考试了！')


def invigilate_teacher():
    time.sleep(5)
    print('考试时间到，学生们可以开始考试了！')
    # 设置内部标志为True，并唤醒所有等待的线程
    event.set()


def main():
    for student_id in range(3):
        threading.Thread(target=student_exam, args=(student_id, )).start()

    threading.Thread(target=invigilate_teacher).start()


if __name__ == '__main__':
    main()
```

结果：



```undefined
学生0等监考老师发卷。。。
学生1等监考老师发卷。。。
学生2等监考老师发卷。。。
考试时间到，学生们可以开始考试了！
开始考试了！
开始考试了！
开始考试了！
```

### 十、定时器对象：threading.Timer

表示一个操作需要在等待一定时间之后执行，相当于一个定时器。`Timer`类是`threading.Thread`的子类，所以它可以像一个自定义线程一样工作。
和线程一样，可以通过`start()`方法启动定时器，在定时器计时结束之前（线程开启之前）可以使用`cancel()`方法停止计时器。计时器等待的时间可能与用户设置的时间不完全一样。

#### `threading.Timer(interval, function, args=None, kwargs=None)`

- `interval`：间隔时间，即定时器秒数。
- `function`：执行的函数。
- `args`：传入`function`的参数，如果为`None`，则会传入一个空列表。
- `kwargs`：传入`function`的关键字参数，如果为`None`，则会传入一个空字典。
- `cancel()`：停止计时器，并取消对应函数的执行，这个方法只有在计时器还没有计时结束之前才会生效，如果已经开始执行函数，则不会生效。

### 十一、栅栏对象：threading.Barrier

栅栏对象用于一个固定数量的线程，而这些线程需要等待彼此的情况。这些线程中的每个线程都会尝试调用`wait()`方法，然后阻塞，直到所有线程都调用了`wait()`方法，然后所有线程会被同时释放。

#### `threading.Barrier(parties, action=None, timeout=None)`

- `parties`：指定需要创建的栅栏对象的线程数。
- `action`：一个可调用对象，当所有线程都被释放时，在释放前，其中一个线程（随机）会自动调用这个对象。
- `timeout`：设置wait()方法的超时时间。
- `wait(timeout=None)`：通过栅栏。当所有线程都调用了这个方法，则所有线程会被同时释放。如果提供了`timeout`参数，那么此参数是优先于初始化方法中的`timeout`参数的。返回值为`range(parties)`中的一个整数，每个线程的返回值都不同。如果提供了`action`参数，那么在所有线程释放前，其中一个线程（随机）会调用这个`action`参数对象，并且如果这个`action`调用发生了异常，则栅栏对象将进入破损状态。如果`wait()`方法发生了超时事件，那么栅栏对象也将进入破损状态。如果栅栏对象进入破损状态或者重置栅栏对象时仍有线程在等待释放，则会报`BrokenBarrierError`异常。
- `reset()`：将一个栅栏对象重置为默认的初始态。如果此时有任何线程正在等待释放，那么将会报`BrokenBarrierError`异常。如果`barrier`中有线程的状态是未知的，那么可能需要外部的某种同步来确保线程已被释放。如果栅栏对象已经破损，那么最好是丢弃它并重新创建一个新的栅栏对象。
- `abort()`：使栅栏对象进入破损状态。这将导致所有已经调用和未调用的`wait()`方法引发`BrokenBarrierError`异常。比如，需要放弃一个线程，但又不想引发死锁的情况，就可以调用这个方法。一个更好的办法是提供一个合理的`timeout`参数值，来自动避免某个线程出错。
- `parties`：通过栅栏的线程数量。
- `n_waiting`：在栅栏中正在等待的线程数量。
- `broken`：如果栅栏对象为破损状态则返回`True`。

##### 栅栏对象简单示例：



```python
"""
栅栏对象使用示例
"""
import time
import threading


def test_action():
    print('所有栅栏线程释放前调用此函数！')


# 创建线程数为3的栅栏对象，当“拦住”3个线程的wait后放行，然后又继续“拦”（如果有的话）
barrier = threading.Barrier(3, test_action)


def barrier_thread(sleep):
    time.sleep(sleep)
    print('barrier thread-%s wait...' % sleep)
    # 阻塞线程，直到阻塞线程数达到栅栏指定数量
    barrier.wait()
    print('barrier thread-%s end!' % sleep)


def main():
    # 这里开启了6个线程，则一次会拦截3个
    for sleep in range(6):
        threading.Thread(target=barrier_thread, args=(sleep, )).start()


if __name__ == '__main__':
    main()
```

结果：



```bash
barrier thread-0 wait...
barrier thread-1 wait...
barrier thread-2 wait...
所有栅栏线程释放前调用此函数！
barrier thread-2 end!
barrier thread-0 end!
barrier thread-1 end!
barrier thread-3 wait...
barrier thread-4 wait...
barrier thread-5 wait...
所有栅栏线程释放前调用此函数！
barrier thread-5 end!
barrier thread-3 end!
barrier thread-4 end!
```

### 十二、多线程与多进程的理解

在网上查多线程资料的时候，很多文章讲到了对Python的多线程与多进程的理解，包括相同之处和它们之间的区别，我就整理了一些点放在这里：

- ##### 进程ID：

> **多线程**的主进程和它的子线程的进程ID，即`os.getpid()`，都是相同的，都是主进程的进程ID。
> **多进程**则是主进程和它的子进程都有各自的进程ID，都不相同。

- ##### 共享数据：

多线程可以共享主进程内的数据，但是多进程用的都是各自的数据，无法共享。

- ##### 主线程：

> 由`Python`解释器运行主`py`时，也就是开启了一个`Python`进程，而这个`py`是这个进程内的一个线程，不过不同于其他线程，它是主线程，同时这个进程内还有其他的比如垃圾回收等解释器级别的线程，所以进程就等于主线程这种理解是有误的。

- ##### CPU多核利用：

> `Python`解释器的线程只能在`CPU`单核上运行，开销小，但是这也是缺点，因为没有利用`CPU`多核的特点。`Python`的多进程是可以利用多个`CPU`核心的，但也有其他语言的多线程是可以利用多核的。

- ##### 单核与多核：

> 一个`CPU`的主要作用是用来做计算的，多个`CPU`核心如果都用来做计算，那么效率肯定会提高很多，但是对于`IO`来说，多个`CPU`核心也没有太大用处，因为没有输入，后面的动作也无法执行。所以如果一个程序是计算密集型的，那么就该利用多核的优势（比如使用`Python`的多进程），如果是`IO`密集型的，那么使用单核的多线程就完全够了。

- ##### 线程或进程间的切换：

> 线程间的切换是要快于进程间的切换的。

- ##### 死锁：

> 指的是两个或两个以上的线程或进程在请求锁的时候形成了互相等待阻塞的情况，导致这些线程或进程无法继续执行下去，这时候称系统处于死锁状态或者系统产生了死锁，这些线程或进程就称为死锁线程或死锁进程。解决死锁的办法可以使用递归锁，即`threading.RLock`，然后线程或进程就可以随意请求和释放锁了，而不用担心别的线程或进程也在请求锁而产生死锁的情况。

- ##### 信号量与进程池：

> 进程池`Pool(n)`只能是“池”中的`n`个进程运行，不能有新的进程，信号量只要保证最大线程数就行，而不是只有这几个线程，旧的线程运行结束，就可以继续来新的线程。