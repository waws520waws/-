# Python程序打包EXE遇到的各种坑

废话不多说，反正我现在还没成功，不过我记录一下遇到的坑！

1：安装相关库太慢

解决办法：离线安装

在一大堆教程中，要安装好几个库，但是有些库用pip在线安装一直卡死（看不到进度条,就当卡死吧)，这个问题可以使用离线安装来解决，下面附上解决过程！

# 安装错误提示（其实是太慢了，我强制停止了)

```python
$ pip install pywin32
Collecting pywin32
  Downloading https://files.pythonhosted.org/packages/a3/8a/eada1e7990202cd27e58eca2a278c344fef190759bbdc8f8f0eb6abeca9c/pywin32-224-cp37-cp37m-win_amd64.whl (9.0MB)
ERROR: Operation cancelled by user
```

　　

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 解决方法如下：

首先找到这个库的下载链接，在这里的就是：

```python
https://files.pythonhosted.org/packages/a3/8a/eada1e7990202cd27e58eca2a278c344fef190759bbdc8f8f0eb6abeca9c/pywin32-224-cp37-cp37m-win_amd64.whl
```

　![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后使用其他HTTP下载工具，我用的是IDM，也可以使用浏览器！

![img](https://img-blog.csdnimg.cn/20190904220344843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后打开存放目录，

![img](https://img-blog.csdnimg.cn/20190904220430251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后在这里打开命令行，执行

```python
pip install pywin32-224-cp37-cp37m-win_amd64.whl
```

　　

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)因为我的已经安装了，当时没截图，我也懒得卸载了，反正就是后面接上库文件就行了（输入文件名第一个字母之后，按TAB键可以补全）

![img](https://img-blog.csdnimg.cn/20190904220600755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后到这里第一个坑就解决了，这个方法适合所有库的安装（官方太慢的时候或者需要安装第三方库)！！

# 第二个坑，调试没问题，运行就报错（代码错误）

在这里我确定使用py文件运行是没问题的，但是打包之后却提示我没有定义exit变量/函数（exit是系统变量）

对于这个问题我暂时的解决办法有两个，要么舍弃这个退出功能，否则无法打包，要么定义这个函数！

我的退出原代码为,这样子是打包不了的，至少我打包的时候是这样

```python
bt_exit = tk.Button(win, text="更新软件", font=("宋体", 15), command=exit).place(x=500, y=250) 
```

　　

可以参考这个方法---- [python 退出程序方法](https://www.cnblogs.com/chenc1992/articles/10732862.html)

![img](https://img-blog.csdnimg.cn/20190904225626914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 怎么去定义我就不细说了，能写出程序的人看了这个方法都会懂的

那么第二个坑就解决了（虽然我发现定义了之后点击退出却没有反应，这个可以点X关闭，所以我就先不管了)

# ==第三个坑：运行提示Failed to execute script main==

![img](https://img-blog.csdnimg.cn/20190904225404471.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

首先，执行构建命令:

```python
pyinstaller -F 主程序.py
```

　　

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20190904222807100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后查看文件列表

![img](https://img-blog.csdnimg.cn/20190904222908377.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

将配置文件（背景文件)放在一起！

![img](https://img-blog.csdnimg.cn/20190904223008675.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

最后运行程序!----------------------居然不报错了！

![img](https://img-blog.csdnimg.cn/20190904223203289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)其实呢，在我之前的构建中是报错的，错误提示是

```python
Failed to execute script 
```

　　

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)在这里我也顺便说一下理论能解决这个错误的方法

首先确保所需文件都在你写的相对目录里，例如我这个程序调用的背景写的是

```python
impath = 'timg.jpg'
img = Image.open(impath)
```

　　

所以这个timg.jpg是跟主程序在同一级目录的，这时候你就需要把这些调用的文件放在所在的目录，然后在构建文件编辑一下，方法如下：

找到这个.spec文件，使用文本编辑(在这里我用vim工具)，下面第二张图(VIM)；

windows用户建议使用Notepad++，下面第三张图(Notepad++)

![img](https://img-blog.csdnimg.cn/20190904223719222.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20190904224021809.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20190904224055369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTU0ODg2,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

反正就是在.spec里面修改一下参数，首先找到

```python
a = Analysis
```

　　

然后找到参数datas,在后面加上你需要调用的文件名(不在同级目录的要写路径加文件名),例如我这里要调用的是timg.jpg文件，而这个文件和程序在同一目录，那么参数就是

```python
datas=['timg.jpg'],
```

　　

假如我的代码写的是程序目录下的bg目录下的timg.jpg文件，那么参数就要写成

```python
datas=['bg\timg.jpg'],
```

　　

为避免出现各种错误，如果是小的程序，建议全部调用文件放在程序根目录！那么到这里也就成功一半了（总不能把程序发给别人的时候还要带上配置文件吧)！

接下来继续压缩！

在打包程序的时候是不会打包图片文件的，但是我们可以把图片文件转为py文件，所以解决这个背景问题的关键点就是这个了，下面附上一些别人的教程！

# [Pyinstaller 使用+打包图片方法](https://blog.csdn.net/MemoryD/article/details/83147300)

下面是我的打包记录。

首先安装相关库

```python
pip install img
pip install base64
```

　　

然后新建一个py文件，内容如下

```
#image转base64
import base64
with open("timg.jpg", "rb") as f:#转为二进制格式
    base64_data = base64.b64encode(f.read())#使用base64进行加密
    print(base64_data)
    file = open('1.txt', 'wb')#写成文本格式
    file.write(base64_data)
    file.close()
```

　　

上面的代码需要注意的就是变量，变量解释如下

```python
"timg.jpg" #需要转换的图片文件
"bg.py" #要生成的图片PY文件
```

　　

运行之后得到下面的数据----很长很长的，如果你得到的是很短的，那就考虑一下是不是图片原文件出问题了，还有一个更快的方法，在线转换

http://imgbase64.duoshitong.com/

```
然后得到这个开头的编码
data:image/jpeg;
```

　　

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

转换之后就需要调用，调用方法如下

首先需要导入base64库以及OS库

所以**需要添加**的代码有两部分，第一部分是库导入

```python
import base64
import os
```

　　

第二部分则是创建临时文件的代码

```python
strs = '''...............//Z\n\n\n\n\n \n\n\n'''
imgdata = base64.b64decode(strs)
file = open('timg.jpg', 'wb')
file.write(imgdata)
file.close()
```

　　

上面的strs中，填写的方法为在下面这段代码中插入从网页获取到的图片转base4的加密信息

```python
import os,base64 
strs='''图片编码放这里，其他不要动\n\n\n\n\n \n\n\n'''
imgdata=base64.b64decode(strs)
file=open('1.jpg','wb')
file.write(imgdata)
file.close()
```

　　

*上面的代码中，**1.jpg**表示新生成的图片文字，这个要根据你的实际需求进行更改*

那么，到这里就变相解决了无法打包图片文件的问题了，接下来重新封装程序看看