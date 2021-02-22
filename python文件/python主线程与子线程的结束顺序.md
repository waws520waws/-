# python主线程与子线程的结束顺序

> **对于程序来说，如果主进程在子进程还未结束时就已经退出，那么`Linux`内核会将子进程的父进程`ID`改为`1`（也就是init进程），当子进程结束后会由`init`进程来回收该子进程。**

> **主线程退出后子线程的状态依赖于它所在的进程，如果进程没有退出的话子线程依然正常运转。如果进程退出了，那么它所有的线程都会退出，所以子线程也就退出了。**

### 1.**主线程退出，进程等待所有子线程执行完毕后才结束**

(子线程全是非守护线程的情况下)
**进程启动后会默认产生一个主线程，默认情况下主线程创建的子线程都不是守护线程**（`setDaemon(False)`）。**因此主线程结束后，子线程会继续执行，进程会等待所有子线程执行完毕后才结束**

**所有线程共享一个终端输出（线程所属进程的终端）**



```python
import threading
import time


def child_thread1():
    for i in range(100):
        time.sleep(1)
        print('child_thread1_running...')


def parent_thread():
    print('parent_thread_running...')
    thread1 = threading.Thread(target=child_thread1)
    thread1.start()
    print('parent_thread_exit...')


if __name__ == "__main__":
    parent_thread()
```

输出为：



```undefined
parent_thread_running...
parent_thread_exit...
child_thread1_running...
child_thread1_running...
child_thread1_running...
child_thread1_running...
...
```

可见父线程结束后，子线程仍在运行，此时结束进程，子线程才会被终止

### 2.**主线程结束后进程不会等待守护线程完成，进程立即结束**

**当设置一个线程为守护线程时，此线程所属进程不会等待此线程运行结束，进程将立即结束**



```python
import threading
import time


def child_thread1():
    for i in range(100):
        time.sleep(1)
        print('child_thread1_running...')


def child_thread2():
    for i in range(5):
        time.sleep(1)
        print('child_thread2_running...')


def parent_thread():
    print('parent_thread_running...')
    thread1 = threading.Thread(target=child_thread1)
    thread2 = threading.Thread(target=child_thread2)
    thread1.setDaemon(True)
    thread1.start()
    thread2.start()
    print('parent_thread_exit...')


if __name__ == "__main__":
    parent_thread()
```

输出：



```css
parent_thread_running...
parent_thread_exit...
child_thread1_running...child_thread2_running...

child_thread1_running...child_thread2_running...

child_thread1_running...child_thread2_running...

child_thread1_running...child_thread2_running...

child_thread2_running...child_thread1_running...

Process finished with exit code 0
```

`thread1`是守护线程，`thread2`非守护线程，因此，进程会等待`thread2`完成后结束，而不会等待`thread1`完成

**注意：子线程会继承父线程中`daemon`的值，即守护线程开启的子线程仍是守护线程**

### 3.主线程等待子线程完成后结束

在线程`A`中使用`B.join()`表示线程`A`在调用`join()`处被阻塞，且要等待线程`B`的完成才能继续执行



```python
import threading
import time


def child_thread1():
    for i in range(10):
        time.sleep(1)
        print('child_thread1_running...')


def child_thread2():
    for i in range(5):
        time.sleep(1)
        print('child_thread2_running...')


def parent_thread():
    print('parent_thread_running...')
    thread1 = threading.Thread(target=child_thread1)
    thread2 = threading.Thread(target=child_thread2)
    thread1.setDaemon(True)
    thread2.setDaemon(True)
    thread1.start()
    thread2.start()
    thread2.join()
    1/0
    thread1.join()
    print('parent_thread_exit...')


if __name__ == "__main__":
    parent_thread()
```

输出：



```ruby
parent_thread_running...
child_thread1_running...
child_thread2_running...
child_thread1_running...
child_thread2_running...
child_thread1_running...
child_thread2_running...
child_thread1_running...
child_thread2_running...
child_thread1_running...
child_thread2_running...
Traceback (most recent call last):
  File "E:/test_thread.py", line 31, in <module>
    parent_thread()
  File "E:/test_thread.py", line 25, in parent_thread
    1/0
ZeroDivisionError: integer division or modulo by zero
```

主线程在执行到`thread2.join()`时被阻塞，等待`thread2`结束后才会执行下一句

`1/0`会使主线程报错退出，且`thread1`设置了`daemon=True`，**因此主线程意外退出时`thread1`也会立即结束。`thread1.join()`没有被主线程执行**