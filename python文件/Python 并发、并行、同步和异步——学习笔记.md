# Python 并发、并行、同步和异步——学习笔记

- 串行：同一个时间段只干一件事

- 并行：同一个时间段可以干多件事

- 并发 V.S. 并行

  - 并发是指一个时间段内，有几个程序在同一个CPU上运行，但是任意时刻只有一个程序在CPU上运行。
  - 并行是指任意时刻点上，有多个程序同时运行在多个CPU上，即每个CPU独立运行一个程序。
  - 并行的最大数量和CPU的数量是一致的。

- 同步 V.S. 异步：

  - 同步是指代码调用IO操作时，必须等待IO操作完成返回才调用的方式
  - 异步是指代码调用IO操作时，不必等待IO操作完成返回才调用的方式

- 多线程：交替执行，另一种意义上的串行 ![\Rightarrow](https://math.jianshu.com/math?formula=%5CRightarrow) 比进程的概念小，减少了进程切换过程中的消耗

- 多进程：并行执行，真正意义上的并行 ![\Rightarrow](https://math.jianshu.com/math?formula=%5CRightarrow) 通过切换不同的进程来实现多任务的并行

- 多进程 V.S. 多线程：

  - 由于全局解释锁GIL的存在，Python的多线程无法利用多核优势，所以不适合计算密集型任务，适合IO密集型任务。
  - CPU密集型任务适合使用多进程
  - 进程切换代价要高于线程
  - 线程之间可以通过全局变量来进行通信，但是进程不行。进程之间的数据是完全隔离的。

- threading类：

   `threading.Lock()`：同步锁（互斥锁），解决数据安全问题

   `threading.RLock()`：递归锁，解决线程死锁问题

   `threading.Semaphore()`：信号量，最多允许同时有n个线程，超过信号量的线程会被堵塞，直到有线程的信号量被释放。

   `threading.Event()`：事件，让不同的线程保持同步而不是独立的运行，彼此交互：你给我发一个信号，我接到了执行一个操作，再给你发一个信号，你接到了执行操作并继续发信号……

## 锁



```python
import threading
#生成锁对象，全局唯一
lock = threading.Lock()
#获取锁
lock.acquire()
#释放锁
lock.release()
```

- 可以使用上下文管理协议来管理锁



```python
lock = threading.Lock()
with lock:
    pass
#等价于：
lock.acquire()
try:
    pass
finally:
    lock.release()
```

- 加锁会影响性能，获取锁和释放锁都需要时间。
- 使用锁可能会引起死锁

## 多线程



```python
import threading
import time

def sleeper(n,name):
    print('Hi, I`m {}. I`m going to sleep 5 seconds'.format(name))
    time.sleep(n)
    print('{} has woken up from sleep'.format(name))
```



```python
e.g.1
def main():
    t = threading.Thread(target = sleeper, args = (5,'thread1'))
    t.start()

    #调用.join（）方法来阻塞当前线程，直到该线程结束后，才跳转执行其余的线程
    #t.join() 如果加上这个语句，程序需要等整个sleeper执行完毕后才会跳回执行后面的两次print

    print('hello')
    print('hello')
>>
Hi, I`m thread1. I`m going to sleep 5 seconds.
hello
hello
thread1 has woken up from sleep.
```



```python
e.g.2
def main():
    start = time.time()
    thread_list = []
    for i in range(1,6):
        t = threading.thread(target=sleeper, args=(5,'thread{}'.format(i)))
        thread_list.append(t)
        t.start()
        print('{} has started.'.format(t.name)
    
    for t in thread_list:
        t.join()
    end = time.time()
    print('total time: {}'.format(end - start))
    print('aaa')  
              
>>
Hi, I`m thread1. I`m going to sleep 5 seconds
Thread-1 has started.
Hi, I`m thread2. I`m going to sleep 5 seconds
Thread-2 has started.
Hi, I`m thread3. I`m going to sleep 5 seconds
Thread-3 has started.
Hi, I`m thread4. I`m going to sleep 5 seconds
Thread-4 has started.
thread2 has woken up from sleep
thread1 has woken up from sleep
thread4 has woken up from sleep
thread3 has woken up from sleep
total time: 5.001285791397095
aaa              
```



```python
e.g.3
def main():
    start = time.time()
    thread_list = []
    for i in range(1,6):
        t = threading.thread(target=sleeper, args=(5,'thread{}'.format(i)))
        thread_list.append(t)
        t.start()
        t.join() #阻塞进程
        print('{} has started.'.format(t.name)

    end = time.time()
    print('total time: {}'.format(end - start))
    print('aaa')  
              
>>
Hi, I`m thread1. I`m going to sleep 5 seconds
thread1 has woken up from sleep
Thread-1 has started.
Hi, I`m thread2. I`m going to sleep 5 seconds
thread2 has woken up from sleep
Thread-2 has started.
Hi, I`m thread3. I`m going to sleep 5 seconds
thread3 has woken up from sleep
Thread-3 has started.
Hi, I`m thread4. I`m going to sleep 5 seconds
thread4 has woken up from sleep
Thread-4 has started.
total time: 20.00114393234253
aaa              
```

### 线程之间通信的方法

#### queue:

- queue是线程安全的，其源码中已经使用了锁
- queue也是共享变量的一种，只是它自身已经具有线程安全性。如果换成其他变量可能会出现不同步的问题。
- 常用方法：
  - `from queue import Queue`
  - `put(self, item, block=True, timeout=None)`：将元素加入队列，队列为满时为阻塞，可以设定timeout参数
  - `get(self, block=True, timeout=None)`：从队列中取出元素，队列为空时阻塞，可以设定timeout参数
  - `qsize()`：返回队列的大小
  - `empty()`：判断是否为空
  - `full()`：判断是否为满
  - `join()`：阻塞直到队列中的所有元素都被取出执行。在调用该方法前要使用`task_done()`方法，否则队列将一直处在阻塞状态
  - `task_done()`：指示之前的队列任务已经完成
  - `put_nowait(self, item)`：非阻塞模式，队列满时直接抛出异常
  - `get_nowait(self)`：非阻塞模式，队列空时直接抛出异常

#### condition

```
from threading import Condition
```

- 条件变量，用于复杂的线程间的同步。

- threading中的`Queue`和`semaphore`的实现都使用了condition

- `Condition`中定义了`__enter__`和`__exit__`方法，所以可以使用`with`语句实现上下文管理

- 常用方法：

  - Condition中实际上有两层锁:

    - 初始化时会上一把默认的递归锁锁住condition，这把锁会在调用`wait()`方法时被解除，称为底层锁
    - 在调用`wait()`方法时还会分配一把锁将当前进程锁住，并将这把锁添加到一个双端队列中去。等到调用`notify()`方法时该锁会被释放掉

    

    ```python
        def __init__(self, lock=None):
            if lock is None:
                lock = RLock()
            self._lock = lock
            # Export the lock's acquire() and release() methods
            self.acquire = lock.acquire
            self.release = lock.release
            
        def wait(self, timeout=None):
             if not self._is_owned():
                raise RuntimeError("cannot wait on un-acquired lock")
            waiter = _allocate_lock()
            waiter.acquire() #分配一把锁锁住当前进程
            self._waiters.append(waiter) #将锁加入到一个双端队列中去
            saved_state = self._release_save() #释放掉初始化conditon时加上的底层锁
            
        def notify(self, n=1):
           if not self._is_owned():
              raise RuntimeError("cannot notify on un-acquired lock")
            all_waiters = self._waiters
            waiters_to_notify = _deque(_islice(all_waiters, n))
            if not waiters_to_notify:
                return
            for waiter in waiters_to_notify:
                waiter.release() #释放掉双端队列中的锁
                try:
                    all_waiters.remove(waiter)
                except ValueError:
                    pass
    ```

  - `wait(self, predicate, timeout=None)`：

    - > Wait until notified or until a timeout occurs.
      >
      > This method releases the underlying lock, and then blocks until it is awakened by a notify() or notify_all() for the same condition variable in another thread, or until the optional timeout occurs.

    - 释放掉当前线程的底层锁，使得其他线程可以访问当前线程。同时为当前线程加上一把锁，在收到`notify()`的通知之前阻塞当前线程剩余的部分继续执行。

    - 即：在调用了`wait()`方法后，需要等待`notify()`的通知，才能启动。

  - `notify(self, n=1)`：

    - > Wake up one or more threads waiting on this condition.

    - 释放掉当前线程的锁，使得其余线程的`wait()`方法可以获得锁

- 

  ```python
  class PersonA(threading.Thread):
      def __init__(self,cond):
          super().__init__(name="A")
          self.cond = cond
          
      def run(self):
          with self.cond: #为PersonA加上一把底层锁
              print('{}: Hello B，你在吗？'.format(self.name))
              self.cond.notify() #给B发出通知，释放掉B中wait加上的锁
              self.cond.wait() #释放掉A的底层锁，确保下一次收到notify()通知后线程能够切入。同时给A加上一把锁，阻塞之后程序的运行
              
   class PersonB(threading.Thread):
      def __init__(self,cond):
          super().__init__(name="B")
          self.cond = cond
          
      def run(self):
          with self.cond: #为PersonB加上一把底层锁
              self.cond.wait() #使用wait方法阻塞后续程序，需要等待A发出通知，同时释放掉B的底层锁，保证线程可以切入进来
              print('{}: Hello A，我在'.format(self.name))
              self.cond.notify() #给A发出一个通知，同时释放掉A中wait加上的锁
              
   if __name__ == '__main__':
      cond = threading.Condition()
      A = PersonA(cond)
      B = PersonB(cond)
      B.start() 
      A.start()
      #如果先调用A.start()，程序会阻塞，无法继续进行，原因在于先启动A时，notify()方法已经被发出，而此时B的wait()方法还没有启动，也就没法接受到notify()的通知，所以程序无法再继续
      
  >> 
  A: Hello B，你在吗？
  B: Hello A，我在
  ```

  - 注意`start()`的顺序很重要，错误的`start()`顺序会导致阻塞，程序无法继续进行。

### 信号量（semaphore）

- 用于控制进入数量的锁
- 文件的读写，通常情况下读可以由多个线程来读，而写仅由一个线程来写
- `threading.semaphore(n)`：可以同时允许n个线程执行，大于n的部分必须等之前有线程释放后才能加入继续执行。

## concurrent.futures

- 封装好的方法，为多线程和多进程提供了统一接口，方便调用。

- 优势：

  - 主线程中可以获取某一个线程的状态或者某一任务的状态，以及返回值
  - 当一个线程完成的时候我们主线程能立即知道

- 

  ```python
  from concurrent.futures import ThreadPoolExecutor
  from concurrent.futures import Future
  
  def get_html(times):
      time.sleep(times)
      print('get page {} success'.format(times))
      return times
  
  executor = ThreadPoolExecutor(max_workers=2) #最大线程数设定为2
  
  task1 = executor.submit(get_html,(3)) #使用submit方法来提交线程
  task2 = executor.submit(get_html,(2)) #executor返回的是一个Future对象
  
  task1.done() #用于判定某个任务是否完成，该方法没有阻塞，是Future的方法
  task2.cancel() #取消线程，只能在线程还没有开始前取消，若线程已经开始执行，则无法取消
  task1.result() #返回线程执行的结果，有阻塞，只有在线程结束后才有返回值
  ```

- 常用方法：

  - `as_completed(fs, timeout=None)`：

    - 生成器，返回已经完成的`Future`
    - 返回结果不一定与参数的传入顺序相同，谁先完成就先返回谁

    

    ```python
    #上例中若只获取已经成功了的task的结果
    from concurrent.futures import as_completed
    urls = [3,4,2]
    all_task = [executor.submit(get_html, (url)) for url in urls]
    for future in as_completed(all_task): 
        #无阻塞的方法，一旦子线程完成执行，主线程中即可立马返回相应的结果
        data = future.result()
        print("get {} page success".format(data))
    ```

  - `.map()`：

    - 类似于python中的`map()`函数，将可迭代的参数列表作用在同一函数上，其本质依旧是一个生成器，直接返回`future.result()`
    - 返回的顺序与调用可迭代参数的顺序一致

    

    ```python
    #只获取已经成功了的task的结果的另一种方法
    urls = [3,4,2]
    for future in executor.mao(get_html, urls):
        print('get {} page success'.format(data))
    ```

  - `wait(fs, timeout=None, return_when=ALL_COMPLETED)`：阻塞主线程，直到指定的某些子线程完成后才能继续执行主线程（在return_when参数中设定）

  - `with ThreadPoolExecutor(n) as executor:`使用with语句来进行调用

### Future

- `submit(self, fn, *args, **kwargs)`：

  

  ```python
  class ThreadPoolExecutor(_base.Executor):
  
      # Used to assign unique thread names when thread_name_prefix is not supplied.
      _counter = itertools.count().__next__
  
      def __init__(self, max_workers=None, thread_name_prefix=''):
          """Initializes a new ThreadPoolExecutor instance.
  
          Args:
              max_workers: The maximum number of threads that can be used to
                  execute the given calls.
              thread_name_prefix: An optional name prefix to give our threads.
          """
          if max_workers is None:
              # Use this number because ThreadPoolExecutor is often
              # used to overlap I/O instead of CPU work.
              max_workers = (os.cpu_count() or 1) * 5
          if max_workers <= 0:
              raise ValueError("max_workers must be greater than 0")
  
          self._max_workers = max_workers
          self._work_queue = queue.Queue()
          self._threads = set()
          self._shutdown = False
          self._shutdown_lock = threading.Lock()
          self._thread_name_prefix = (thread_name_prefix or
                                      ("ThreadPoolExecutor-%d" % self._counter()))
      
      
      def submit(self, fn, *args, **kwargs):
          with self._shutdown_lock: #添加锁,为了锁住后面的一段代码
              if self._shutdown:
                  raise RuntimeError('cannot schedule new futures after shutdown')
  
              f = _base.Future() #生成一个Future对象，最后这个对象会被返回
              w = _WorkItem(f, fn, args, kwargs)
              #WorkItem才是整个线程池的真正执行单元，Future对象会被传入WorkItem中
  
              self._work_queue.put(w) #整个WorkItem实例被放入到_work_queue中
              self._adjust_thread_count()
              return f
      submit.__doc__ = _base.Executor.submit.__doc__
      
      def _adjust_thread_count(self):
          # When the executor gets lost, the weakref callback will wake up
          # the worker threads.
          def weakref_cb(_, q=self._work_queue):
              q.put(None)
          # TODO(bquinlan): Should avoid creating new threads if there are more
          # idle threads than items in the work queue.
          num_threads = len(self._threads)
          if num_threads < self._max_workers: #判断启动了多少线程
              thread_name = '%s_%d' % (self._thread_name_prefix or self,
                                       num_threads)
              t = threading.Thread(name=thread_name, target=_worker,
                                   args=(weakref.ref(self, weakref_cb),
                                         self._work_queue)) 
              #线程执行的函数是_worker,而_worker中执行的函数来自self._work_queue
              #_worker函数的作用：运行线程，不停的从_work_queue中取出_WorkItem
              t.daemon = True
              t.start()
              self._threads.add(t) #将启动了的线程数量加入到_threads中
              _threads_queues[t] = self._work_queue
              
   class _WorkItem(object):
      #设定这个类的目的其实就是执行函数，并将执行的结果设置到Future中
      def __init__(self, future, fn, args, kwargs):
          self.future = future
          self.fn = fn #需要执行的函数在这里传入
          self.args = args
          self.kwargs = kwargs
  
      def run(self):
          if not self.future.set_running_or_notify_cancel():
              return
  
          try:
              result = self.fn(*self.args, **self.kwargs)
          except BaseException as exc:
              self.future.set_exception(exc)
              # Break a reference cycle with the exception 'exc'
              self = None
          else:
              self.future.set_result(result) #将函数执行的结果设置到Future中
  ```

- `Future`的常用方法：

  - `cancel(self)`：取消future

    

    ```python
        def cancel(self):
            """Cancel the future if possible.
    
            Returns True if the future was cancelled, False otherwise. A future
            cannot be cancelled if it is running or has already completed.
            """
            with self._condition: #使用条件变量
                if self._state in [RUNNING, FINISHED]: 
                    #判断future的状态使否是在执行或已完成，若是则无法取消
                    return False
    
                if self._state in [CANCELLED, CANCELLED_AND_NOTIFIED]:
                    return True
    
                self._state = CANCELLED
                self._condition.notify_all()
    
            self._invoke_callbacks()
            return True
    ```

  - `cancelled(self)`：状态判断，是否已经取消

  - `running(self)`：状态判断，是否在执行

  - `done(self)`：状态判断，是否已完成或已取消

  - `result(self, timeout=None)`：

    - 返回调用的结果，是有阻塞的方法，依旧是使用condition来实现
    - 其中调用了condition的`wait()`方法

  - `set_result(self, result)`：

    - > set the return value of work associated with the future

    - 调用了condition的`notify_all()`方法

## 多进程

**在windows下使用多进程，不管是`ProcessPoolExecutor`还是`multiprocessing`，代码要放到`if __ name__ == '__ main __'`模块中，否则会报错**



```python
from concurrent.futures import ProcessPoolExecutor, as_completed
def fib(b):
    if n <= 2:
        return 1
    return fib(n-1)+fib(n-2)
if __name__ == '__main__':
    with ProcessPoolExecutor(3) as executor:
        all_task = [executor.submit(fib,(num)) for num in range(15,45)]
        for future in as_completed(all_task):
            data = future.result()
            print('result: {}'.format(data))
```

- 在处理CPU密集型问题时，使用多进程会比使用多线程快，但是很难做到翻倍，因为进程的切换比线程的切换更耗时。

### multiprocessing



```python
import time
import multiprocessing

def get_html(n):
    time.sleep(n)
    return n

if __name__ == '__main__':
    progress = multiprocessing.Process(target=get_html, args=(2,))
    progress.start()
    progress.join()
    print(progress.pid) #打印进程的ID
```

#### multiprocessing 的进程池



```python
pool = multiprocessing.Pool(multiprocessing.cpu_count())
result = pool.apply_async(get_html, args=(3,))#进程的提交
pool.close() #在调用.join之前要先调用.close，关闭进程池使其不再接受进程，否则.join方法会出错
pool.join()
print(result.get())
```

- 常用方法：

  - `apply_async(self, func, args=(), kwds={}, callback=None, error_callback=None)`：`apply()`方法的异步版本

  - `apply(self, fuc, args=(), kwds={})`：添加进程

  - `imap(self, func, iterable, chunksize=1)`：对应到之前executor里的`map`方法

    

    ```python
    pool = multiprocessing.Pool(multiprocessing.cpu_count())
    for result in pool.imap(get_html, [1,5,3]):
        print('{} sleep success'.format(result))
    >>
    1 sleep success
    5 sleep success
    3 sleep success
    ```

  - `imap_unordered(self, func, iterable, chunksize=1)`：先执行完的先返回

    

    ```python
    for result in pool.imap_unordered(get_html, [1,5,3]):
        print('{} sleep success'.format(result))
    >>
    1 sleep success
    3 sleep success
    5 sleep success
    ```

### 进程之间的通信

- 多进程中的`Queue`与多线程中的不一样，这里要使用来自`maltiprocessing`里的`Queue`

  - 

    ```python
    from multiprocessing import Process,Queue
    import multiprocessing
    
    def producer(queue):
        queue.put("a")
        time.sleep(2)
    
    def consumer(queue):
        time.sleep(2)
        data = queue.get()
        print(data)
    
    if __name__ == "__main__":
        queue = Queue(10)
        my_producer = Process(target=producer, args=(queue,))
        my_consumer = Process(target=consumer, args=(queue,))
        my_producer.start()
        my_consumer.start()
        my_producer.join()
        my_consumer.join()
    ```

- `maltiprocessing`里创建的`Queue`不能用在`Pool`创建的进程池中

  - 

    ```python
    from multiprocessing import Pool,Queue
    if __name__ == "__main__":
        queue = Queue(10)
        pool = Pool(2)
    
        pool.apply_async(producer, args=(queue,))
        pool.apply_async(consumer, args=(queue,))
    
        pool.close()
        pool.join()
    #没有任何结果返回
    ```

- 进程池中进程的通信需要用来自`Manager`里的`Queue`

  - 

    ```python
    from maltiprocessing import Manager
    if __name__ == "__main__":
        queue = Manager().Queue(10)
        pool = Pool(2)
    
        pool.apply_async(producer, args=(queue,))
        pool.apply_async(consumer, args=(queue,))
    
        pool.close()
        pool.join()
    ```

- 使用pipe进行通信

  - pipe只能适用于两个进程间的通信

  - pipe返回的是`Connection`的实例，`Connection`的常用方法有：`send(self, obj)`和`recv(self)`

    - 

      ```python
      def Pipe(duplex=True):
          return Connection(), Connection()
      
      class Connection(object):
          def send(self, obj):
              pass
      
          def recv(self):
              pass
      ```

  - pipe的性能比queue要高

  - 

    ```python
    from maltiprocessing import Pipe
    def producer(pipe):
        pipe.send("bobby")
    
    def consumer(pipe):
        print(pipe.recv())
    
    if __name__ == "__main__":
        recevie_pipe, send_pipe=Pipe() #管道的两端，发送端和接收端
    
        my_producer= Process(target=producer, args=(send_pipe, ))
        my_consumer = Process(target=consumer, args=(recevie_pipe,))
    
        my_producer.start()
        my_consumer.start()
        my_producer.join()
        my_consumer.join()
    ```

- 共享全局变量在多进程中是不适用的



```python
def producer(a):
    a += 100
    time.sleep(2)
def consumer(a):
    time.sleep(2)
    print(a)
if __name__ == '__main__':
    a = 1
    my_producer = Process(target=producer,args=(a,))
    my_consumer = Process(target=consumer,args=(a,))
    my_producer.start()
    my_consumer.start()
    my_producer.join()
    my_consumer.join()
>> 1 #返回值还是1，不是101
```

- 进程间的同步（维护一个公共的内存/变量）使用`Manager()`

  - 

    ```python
    def Manager():
        return multiprocessing.SyncManager()
    
    class SyncManager(multiprocessing.managers.BaseManager):
        def Condition(self, lock=None):
            return threading.Condition(lock)
    
        def Event(self):
            return threading.Event()
    
        def Lock(self):
            return threading.Lock()
    
        def Namespace(self):
            pass
    
        def Queue(self, maxsize=None):
            return queue.Queue()
    
        def RLock(self):
            return threading.RLock()
    
        def Semaphore(self, value=None):
            return threading.Semaphore(value)
    
        def Array(self, typecode, sequence):
            pass
    
        def Value(self, typecode, value):
            pass
    
        def dict(self, mapping_or_sequence):
            pass
    
        def list(self, sequence):
            pass
    ```

  - 

    ```python
    def add_data(p_dict, key, value):
        p_dict[key] = value
        
    if __name__ == '__main__':
        progress_dict = Manager().dict()
        
        first = Process(target=add_data, args=(progress_dict, 'Bobby1',22))
        second = Process(target=add_data, args=(progress_dict, 'Bobby2',23))
        
        first.start()
        second.start()
        first.join()
        second.join()
        
        print(progress_dict)
    >> {'bobby1': 22, 'bobby2': 23}
    ```

  - 使用时注意数据的同步，必要时需要加锁

## 文档阅读

### Thread Objects

- Thread类：在单独的控制线程中运行的活动（activity），有两种指定活动的方法：

  - 将可调用对象传递给构造器
  - 在子类中重写 `run()` 方法

- 通过调用thread对象的`start()`方法来激活对象，`is_alive()`方法用来判断线程是否处在激活状态。

- 其他线程可以调用`join()`方法来阻塞正在调用的线程，直到调用`join()`方法的线程结束。

- ```
  threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
  ```

  - target：传入被`run()`方法调用的可调用对象
  - name：线程的名字
  - args：目标调用的参数的元组
  - kwargs：目标调用的参数的字典

------

### Lock Objects

- `threading.Lock()` 生成一个锁对象。一旦某个线程获取了锁，之后的所有获取锁的请求都会被阻塞，直到锁被释放。任何线程都可以释放锁，不一定是获得锁的线程。

------

### RLock Objects

- 可重入锁允许同一线程多次获取同一把锁。

------

### Event Objects

- 一个事件对象管理线程内的一个内部标记，可以使用`set()`方法将标记设置为`true`，使用`clear()`方法将标记重置为`false`。标记为`false`时，`wait()`方法被阻塞，直到标记被设定为`true`。

- `threading.Event()`构造一个事件对象，内部标记初始化为`false`。

- `is_set()`：检查内部标记的状态，只有在状态为为`true`时返回`true`。

- `set()`：将内部标记设定为`true`。完成设定后，所有调用了`wait()`方法的线程都将不再受到阻塞。

- `clear()`：将内部标记重置为`false`。

- ```
  wait(timeout=None)
  ```

  ：阻塞进程直到内部标记设定为

  ```
  true
  ```

  或满足选定的timeout。

  - timeout：a floating point number，以秒为单位设定超时时间

------

### queue

- 同步队列：队列模块实现了多生产者、多消费者队列。当必须在多个线程之间安全地交换信息时，它在线程编程中特别有用。
- 该模块实现了三种队列：
  - FIFO队列：先添加的任务先读取（先入先出）
  - LIFO队列：后添加的任务先读取（后入先出）
  - 优先队列：输入的数据排序，值最小的数据先读取
- 类：
  - Queue（maxsize=0）：FIFO队列的构造函数
  - LifoQueue（maxsize=0）：LIFO队列的构造函数
  - PriorityQueue（maxsize=0）：优先队列的构造函数
  - maxsize参数用于设定可以放入队列的项目数的上限。若maxsize小于或等于零，则队列大小为无限大。

#### Queue Objects

- `Queue.qsize()`：返回队列的大小

- `Queue.empty()`：检查队列是否为空，是则返回`True`

- `Queue.full()`：检查队列是否为满，是则返回`True`

- ```
  Queue.put(item, block=True, timeout=None)
  ```

  ：将元素放入队列

  - `block=True, timeout=None`：在必要时阻塞，当队列有空位时执行插入
  - timeout为正数：阻塞最多timeout秒，若队列还是没有空位则抛出Full异常
  - block=False：在有空位时直接插入，否则抛出异常

- `Queue.get(block=True，timeout=None)`：将元素从队列中移除并返回
  （未完）