# Python项目生成requirements.txt文件

我们在写Python脚本的时候往往会用到很多第三方库，但是当我们把脚本换个环境之后就需要手动安装第三方库，有时候有的第三方库还需要一些别的依赖。为了省事，我们可以导出一个requirements.txt，把需要安装的第三方库放在里面。下面我们就讲一下如何导出这个requirements.txt。

## 方法一：

```python
pip freeze > requirements.txt
```

这种方法配合virtualenv才好用，否则会把整个环境中的包都列出来。所以往往不适用