# Python2 urllib2 与 Python3 urllib.request API对照

| python2                                 | python3                                        |
| --------------------------------------- | ---------------------------------------------- |
| urllib2.urlopen()                       | urllib.request.urlopen()                       |
| urllib2.install_opener()                | urllib.request.install_opener()                |
| urllib2.build_opener()                  | urllib.request.build_opener()                  |
| urllib2.URLError                        | urllib.error.URLError                          |
| urllib2.HTTPError                       | urllib.error.HTTPError                         |
| urllib2.Request                         | urllib.request.Request                         |
| urllib2.OpenerDirector                  | urllib.request.OpenerDirector                  |
| urllib2.BaseHandler                     | urllib.request.BaseHandler                     |
| urllib2.HTTPDefaultErrorHandler         | urllib.request.HTTPDefaultErrorHandler         |
| urllib2.HTTPRedirectHandler             | urllib.request.HTTPRedirectHandler             |
| urllib2.HTTPCookieProcessor             | urllib.request.HTTPCookieProcessor             |
| urllib2.ProxyHandler                    | urllib.request.ProxyHandler                    |
| urllib2.HTTPPasswordMgr                 | urllib.request.HTTPPasswordMgr                 |
| urllib2.HTTPPasswordMgrWithDefaultRealm | urllib.request.HTTPPasswordMgrWithDefaultRealm |
| urllib2.AbstractBasicAuthHandler        | urllib.request.AbstractBasicAuthHandler        |
| urllib2.HTTPBasicAuthHandler            | urllib.request.HTTPBasicAuthHandler            |
| urllib2.ProxyBasicAuthHandler           | urllib.request.ProxyBasicAuthHandler           |
| urllib2.AbstractDigestAuthHandler       | urllib.request.AbstractDigestAuthHandler       |
| urllib2.HTTPDigestAuthHandler           | urllib.request.HTTPDigestAuthHandler           |
| urllib2.ProxyDigestAuthHandler          | urllib.request.ProxyDigestAuthHandler          |
| urllib2.HTTPHandler                     | urllib.request.HTTPHandler                     |
| urllib2.HTTPSHandler                    | urllib.request.HTTPSHandler                    |
| urllib2.FileHandler                     | urllib.request.FileHandler                     |
| urllib2.FTPHandler                      | urllib.request.FTPHandler                      |
| urllib2.CacheFTPHandler                 | urllib.request.CacheFTPHandler                 |
| urllib2.UnknownHandler                  | urllib.request.UnknownHandler                  |



## 解决AttributeError: module 'urllib' has no attribute 'request'问题

语言版本：python3.7

环境：win10

最近写爬虫的时候导入urllib并使用urllib.request时总是报错

```
AttributeError: module 'urllib' has no attribute 'request'
```

去urllib包里寻找发现__init__.py文件是空的，以为自己误删了，后来去github的cpython看源码，发现他的__init__.py文件也是空的。这意味着我们import urllib时仅仅导入该包，其他什么都做不了。因此需要我们导入的时候这样导入：

```
import urllib.request
```

然后正常使用就没有问题了