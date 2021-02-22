# Python标准库中的marshal模块

marshal-**内部的Python对象序列化**

​    该模块包含可以以二进制格式读取和写入Python值的函数。该格式是针对Python的，但独立于机器架构问题(例如，您可以将Python值写入PC上的文件，将文件传输到Sun，并在那里读取它)。使用marshal序列化的二进制数据格式还没有文档化，它可能会在Python版本之间发生变化(尽管很少发生)。

​    这不是一个通用的“持久性”模块。对于**通过RPC调用的Python对象的一般持久性和传输**，请参阅模块pickle和shelve。marshal模块主要用于支持读取和编写.pyc文件的Python模块的“伪编译”代码。因此，当需要时，Python维护人员保留在向后不兼容的方式中修改编组格式的权利。如果您正在**序列化和反序列化Python对象，则使用pickle模块——性能是可比较的，版本独立性是有保证的，而pickle支持的对象范围比marshal大得多**。

​    并不是所有的Python对象类型都被支持；通常，只有值与特定的Python调用的对象无关才能被这个模块编写和读取。支持以下类型：布尔值、整数、浮点数、复数、字符串、字节、字节数组、元组、列表、集合、frozenset(不可变集合)、字典和代码对象。单独的None、Ellipsis和StopIteration也可以编组和解组。对于低于版本3的格式，递归列表、集合和字典不能编写(见下面)。

​    模块中有可以读取/写入文件，以及在字节类对象上操作的函数。

> **marshal.dump(value, file[, version])**
> ​    将值写入到一个打开的输出流里。参数value表示待序列化的值。file表示打开的输出流。如:以”wb”模式打开的文件，sys.stdout或者os.popen。对于一些不支持序列类的类型，dump方法将抛出ValueError异常。但也会将垃圾数据写入文件。该对象将不能通过load()来正确读取。
> ​    版本参数表示转储应该使用的数据格式(见下文)。

> **marshal.load(file)**
> ​    从打开的文件中读取值并返回它。如果没有读取有效的值(例如，因为数据具有不同的Python版本的不兼容的编组格式)，会引发EOFError、ValueError或TypeError错误。文件必须是可读的二进制文件。
> ​    注意，如果一个包含不支持类型的对象被编组了dump()， load()将会用None代替unmarshallable类型。

> **marshal.dumps(value[, version])**
> ​    返回将被转储(值、文件)写入文件的字节对象。该值必须是受支持的类型。如果值有(或包含有)不支持类型的对象，则抛出ValueError异常。
> ​    版本参数表示转储应该使用的数据格式(见下文)。

> **marshal.loads(bytes)**
> ​    将bytes样对象转换为值。如果没有找到有效值，会引发EOFError、ValueError或TypeError错误。输入中的额外字节被忽略。

> **marshal.version**
> ​    指示模块使用的格式。版本0是历史格式，版本1共享字符串，版本2使用二进制格式的浮点数。版本3增加了对对象实例和递归的支持。当前版本是4。

​    注：这个模块的名称源于模块化设计人员使用的一些术语(其中包括其他一些术语)，他们使用“编组”这个术语来将数据以自包含的形式发送出去。严格地说，“to marshal”的意思是将一些数据从内部转换成外部形式(例如，在RPC缓冲区中)，以及反向过程的“解组”。

​    例子：

```shell
>>>  import  marshal
>>>  import  os
>>>  x=[30,  5.0,  [1,  2,  3],  (4,  5,  6),  {'a':  1,  'b':  2,  'c':  3},  {8,  9,  7}]
>>>  f=open(os.path.join(os.getcwd(  ),'ftest.txt'),'wb')
>>>  marshal.dump(x,f)
102
>>>  f.close()
>>>  f2=open(os.path.join(os.getcwd(  ),'ftest.txt'),'rb')
>>>  y=marshal.load(f2)
>>>  print(list(y))
[30,  5.0,  [1,  2,  3],  (4,  5,  6),  {'a':  1,  'b':  2,  'c':  3},  {8,  9,  7}]
>>>  f2.close()
>>>  t1=marshal.dumps(x)
>>>  t2=marshal.loads(t1)
>>>  print(list(t2))
[30,  5.0,  [1,  2,  3],  (4,  5,  6),  {'a':  1,  'b':  2,  'c':  3},  {8,  9,  7}]
>>>  
```

