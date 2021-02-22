# python实现三个线程交替打印1-100的两种方法

### 第一种方法

**使用condition实现**

> 这种方法是网上的普遍解决方案，使用三个锁，以及线程之间的通讯来控制，就是有点绕



```python
# -*- coding:utf-8 -*-
import threading
import time

c1 = threading.Condition()
c2 = threading.Condition()
c3 = threading.Condition()

x = 0

def f1():
    while True:
        with c2:
            with c1:
                time.sleep(1)
                global x
                x+=1
                print('线程1',x)
                c1.notify()
            c2.wait()
            

def f2():
    while True:
        with c3:
            with c2:
                time.sleep(1)
                global x
                x+=1
                print('线程2',x)
                c2.notify()
            c3.wait()

def f3():
    while True:
        with c1:
            with c3:
                time.sleep(1)
                global x
                x+=1
                print('线程3',x)
                c3.notify()
            c1.wait()
            

t=[]
t.append(threading.Thread(target=f1))
t.append(threading.Thread(target=f2))
t.append(threading.Thread(target=f3))
for i in t:
    i.start()
for i in t:
    i.join()
```

### 第二种方法

> 这种方法是我自己琢磨出来的，可能不严谨，但跑出来的结果没有问题，希望大神可以指正我的不足，这种方法的使用一个锁（`Lock`）和三个线程事件对象（`Event`）来实现数据同步和线程通讯



```python
import threading, time

lock = threading.Lock()
e1 = threading.Event()
e2 = threading.Event()
e3 = threading.Event()
x=0

def f1():
    while True :
        with lock:
            # time.sleep(1)
            global x
            if x < 100:
                x+=1
            else:
                e2.set()
                break
            print('线程1', x)
        e1.wait()
        e1.clear()
        e2.set()

def f2():
    while True:
        with lock:
            # time.sleep(1)
            global x
            if x < 100:
                x+=1
            else:
                e3.set()
                break
            print('线程2', x)
        e2.wait()
        e2.clear()
        e3.set()

def f3():
    while True:
        with lock:
            # time.sleep(1)
            global x
            if x < 100:
                x+=1
            else:
                e1.set()
                break
            print('线程3', x)
        e1.set()
        e3.wait()
        e3.clear()
        

t=[]
t.append(threading.Thread(target=f1))
t.append(threading.Thread(target=f2))
t.append(threading.Thread(target=f3))
for i in t:
    i.start()
for i in t:
    i.join()
```