# Python 队列（Queue）作用和用法

#### 作用：

- **解耦**：
  使程序直接实现松耦合，修改一个函数，不会有串联关系。
  提高处理效率：ＦＩＦＯ　＝　现进先出，ＬＩＦＯ　＝　后入先出。
- **队列**：
  队列可以并发的派多个线程，对排列的线程处理，并切每个需要处理线程只需要将请求的数据放入队列容器的内存中，线程不需要等待，当排列完毕处理完数据后，线程在准时来取数据即可。请求数据的线程只与这个队列容器存在关系，处理数据的线程`down`掉不会影响到请求数据的线程，队列会派给其他线程处理这分数据，它实现了解耦，提高效率。队列内会有一个有顺序的容器，列表与这个容器是有区别的，列表中数据虽然是排列的，但数据被取走后还会保留，而队列中这个容器的数据被取后将不会保留。**当必须在多个线程之间安全地交换信息时，队列在线程编程中特别有用**。

**多线程尽量使用`Queue`而不是使用列表（`list`），因为列表的`pop()`和`push()`效果看起来和和Queue的队列、栈一样，但`list`不是线程安全的**

#### Python四种类型的队例：

- `Queue`：FIFO 即first in first out 先进先出
- `LifoQueue`：LIFO 即last in first out 后进先出
- `PriorityQueue`：优先队列，级别越低，越优先
- `deque`:双边队列

#### 导入三种队列



```cpp
from queue import Queue,LifoQueue,PriorityQueue
```

#### Queue 先进先出队列：

**常用方法**：

- `Queue.qsize()` 返回队列的大小
- `Queue.empty()`如果队列为空，返回`True`,反之`False`
- `Queue.full()` 如果队列满了，返回`True`,反之`False`，`Queue.full` 与 `maxsize` 大小对应
- `Queue.get([block[, timeout]])`获取队列，`timeout`等待时间
- `Queue.get_nowait()` 相当于`Queue.get(False)`，非阻塞方法
  `Queue.put(item)` 写入队列，`timeout`等待时间
- `Queue.task_done()` 在完成一项工作之后，`Queue.task_done()`函数向任务已经完成的队列发送一个信号。每个`get()`调用得到一个任务，接下来`task_done()`调用告诉队列该任务已经处理完毕。
- `Queue.join()`实际上意味着等到队列为空，再执行别的操作



```bash
#基本FIFO队列  先进先出 FIFO即First in First Out,先进先出
#maxsize设置队列中，数据上限，小于或等于0则不限制，容器中大于这个数则阻塞，直到队列中的数据被消掉
q = Queue(maxsize=0)

#写入队列数据
q.put(0)
q.put(1)
q.put(2)

#输出当前队列所有数据
print(q.queue)
#删除队列数据，并返回该数据
q.get()
#输也所有队列数据
print(q.queue)

# 输出:
# deque([0, 1, 2])
# deque([1, 2])
```

#### LifoOueue 后进先出队列：



```bash
#LIFO即Last in First Out,后进先出。与栈的类似，使用也很简单,maxsize用法同上
lq = LifoQueue(maxsize=0)

#队列写入数据
lq.put(0)
lq.put(1)
lq.put(2)

#输出队列所有数据
print(lq.queue)
#删除队尾数据，并返回该数据
lq.get()
#输出队列所有数据
print(lq.queue)

#输出:
# [0, 1, 2]
# [0, 1]
```

#### 优先队列：



```bash
# 存储数据时可设置优先级的队列
# 优先级设置数越小等级越高
pq = PriorityQueue(maxsize=0)

#写入队列，设置优先级
pq.put((9,'a'))
pq.put((7,'c'))
pq.put((1,'d'))

#输出队例全部数据
print(pq.queue)

#取队例数据，可以看到，是按优先级取的。
pq.get()
pq.get()
print(pq.queue)#输出：[(9, 'a')]
```

#### 双边队列：



```bash
#双边队列
dq = deque(['a','b'])

#增加数据到队尾
dq.append('c')
#增加数据到队左
dq.appendleft('d')

#输出队列所有数据
print(dq)
#移除队尾，并返回
print(dq.pop())
#移除队左，并返回
print(dq.popleft())#输出:deque(['d', 'a', 'b', 'c'])cd
```