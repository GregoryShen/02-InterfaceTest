

## [9-https请求（SSL）](https://mp.weixin.qq.com/s/XYs0baToMwfhhUQfCDkSYQ)

⚠️: 这篇文章可能有点老了, 最新版应该不是这样的

### 前言

本来最新的 requests 库 v 2.13.0 是支持 https 请求的,但是一般写脚本的时候,会用 fiddler 抓包, 这时候会报 requests.exceptions.SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed(_ssl.c:590)

小编环境:

python: 2.7.12

requests: 2.13.0

fiddler: v 4.6.2.0

### SSL 问题

1. 不启用 filddler,直接发 https 请求,不会有 SSL 问题(也就是说不想看到 SSL 问题关掉 fiddler 就行)
2. 启动 fiddler 抓包,会出现这个错误: requests.exceptions.SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed(_ssl.c:590)

### verify 参数设置

1. Requests 的请求参数默认 `verify=True`
2. 如果你将 verify 设置为 False, Requests 也能忽略对 SSL 证书的验证
3. 但是依然会出现两行 Warning, 可以不用管

### 忽略 Warning

可以忽略这两行警告信息,代码如下:

```python
import requests

# 禁用安全警告
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

url = "https://passport.cnblogs.com/user/signin"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:44.0) Gecko/20100101 Firefox/44.0"
}
r = requests.get(url, headers=headers, verify=False)
print(r.status_code)  # 200
```

## [10-token登录](https://mp.weixin.qq.com/s/UtCp8rmdznHvf6VWjw0jdQ)

### 前言

有些登录不是用 cookie 来验证的，是用 token 参数来判断是否登录。token 传参有两种一种是放在请求头里，本质上是跟 cookie 是一样的，只是换个单词而已；另外一种是在 URL 请求参数里，这种更直观

### 登录返回 token

![](http://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyDVMS1TXjHLicFNrFnp0NE86xOmfuCMWLvmsBhGloH0nMSkFnuOe5RarVoFyAMODKq3QuibG8S0DExw/640)

如上图这个登录，无 cookies，但是登录成功后有返回 token

### 请求头带 token

![](http://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyDVMS1TXjHLicFNrFnp0NE86KSYhb88bCBVHjvIhpW0DckCBWibuPnK5Sp2KTTB2PA9aJdjKJWbGVzA/640)

登录成功后继续操作其他页面，发现 post 请求的请求头都会带 token 参数。这种请求其实比 cookie 更简单，直接把登录后的 token 放到头部就行。

### token 关联

1. 用脚本实现登录，获取 token 参数，获取后传参到请求头就可以了
2. 如果登录有验证码，前面的脚本登录步骤就省略了，自己手动登录后获取 token

```python
import requests

header = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:44.0) Gecko/20100101 Firefox/44.0",
    "Accept": "*/*",
    "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3",
    "Accept-Encoding": "gzip, deflate",
    "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8",
    "X-Requested-With": "XMLHttpRequest",
    "Content-Length": "423",
    "Connection": "keep-alive"
}

body = {"key1": "value1", "key2": "value2"} # 这里账号密码就是抓包的数据
s = requests.session()
login_url = "http://xxx.login"
login_ret = s.post(login_url, headers=header, data=body)
# 这里 token 在返回的 json 里，可以直接提取
token = login_ret.json()["token"]
# 这是登录后发的一个 post 请求
post_url = "http://xxx"
# 添加 token 到请求头
header["token"] = token
# 如果这个 post 请求的头部的其他参数变了，也可以直接更新
header["Content-Length"] = '9'
body1 = {
    "key": "value"
}
post_ret = s.post(post_url, headers=header, data=body1)
print(post_ret.content)
```

### 其他 token

1. 返回 token 的值，有的藏在 cookie 里，有的直接在返回的 json 里，也有的在返回的 xml 页面或者 html 里，形式各种各样，自己去找
2. 传 token 的方式也有几种，有的传在头部，有的在 URL 里

## [15-multipart/form-data表单提交](https://mp.weixin.qq.com/s/8-E6g4vmzuGe5-yrbE5axQ)





## [16-multipart/form-data上传多个附件](https://mp.weixin.qq.com/s/UDO9FIlO0DW-P4d7kXF1sA)







## [17-响应时间与超时(timeout)](https://mp.weixin.qq.com/s/7HzCRTzLiCMESHHiw58RtA)

### 前言

requests 发请求时，接口的响应时间也是我们需要关注的一个点，如果响应时间太长也是不合理的。如果服务端没有及时响应，也不能一直等着，可以设置一个 timeout 超时的时间。

关于 requests 请求的响应时间，很多资料写的是 `r.elapsed.microseconds`获取的，这样写是不对的。

### elapsed 官方文档

1. `elapsed`官方文档地址：https://docs.python-requests.org/en/latest/api/#requests.Response

	class requests.Response

	​		The Response object, which contains a server’s response to an HTTP request.

	elapsed = None

	​		The amount of time elapsed between sending the request and the arrival of the response (as a timedelta). This property specifically measures the time taken between sending the first byte of the request and finishing parsing the headers. It is therefore unaffected by consuming the response content or the value of the `stream` keyword argument.

	简单翻译：计算的是从发送请求到服务端响应回来这段时间（也就是时间差），发送的第一个数据到收到最后一个数据之间，这个时长不受响应内容的影响。

2. 用 help() 查看 elapsed 里面的方法

	```python
	>>> import requests
	>>> r = requests.get("https://www.baidu.com")
	>>> help(r.elapsed)
	Help on timedelta object:
	class timedelta(builtins.object)
	 |  Difference between two datetime values.
	 |  
	 |  timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)
	 |  
	 |  All arguments are optional and default to 0.
	 |  Arguments may be integers or floats, and may be positive or negative.
	 |  
	 |  Methods defined here:
	 |  
	 |  __abs__(self, /)
	 |      abs(self)
	...
	 |  total_seconds(...)
	 |      Total seconds in the duration.	# 总时长，单位秒
	 |  
	 |  ----------------------------------------------------------------------
	 |  Static methods defined here:
	 |  
	 |  __new__(*args, **kwargs) from builtins.type
	 |      Create and return a new object.  See help(type) for accurate signature.
	 |  
	 |  ----------------------------------------------------------------------
	 |  Data descriptors defined here:
	 |  
	 |  days		# 以天为单位
	 |      Number of days.
	 |  
	 |  microseconds	# 获取微秒部分，大于0小于1秒
	 |      Number of microseconds (>= 0 and less than 1 second).
	 |  
	 |  seconds		# 大于0小于1天
	 |      Number of seconds (>= 0 and less than 1 day).
	 |  
	 |  ----------------------------------------------------------------------
	 |  Data and other attributes defined here:
	 |  
	 |  max = datetime.timedelta(days=999999999, seconds=86399, microseconds=9...		# 最大时间
	 |  
	 |  min = datetime.timedelta(days=-999999999)		# 最小时间
	 |  
	 |  resolution = datetime.timedelta(microseconds=1)		# 最小时间单位
	```

### 获取响应时间

1. 获取 elapsed 不同的返回值

	```python
	>>> import requests
	>>> r = requests.get("https://www.baidu.com")
	>>> print(r.elapsed)
	0:00:00.504449
	>>> print(r.elapsed.total_seconds())
	0.504449
	>>> print(r.elapsed.microseconds)
	504449
	>>> print(r.elapsed.seconds)
	0
	>>> print(r.elapsed.days)
	0
	>>> print(r.elapsed.max)
	999999999 days, 23:59:59.999999
	>>> print(r.elapsed.min)
	-999999999 days, 0:00:00
	>>> print(r.elapsed.resolution)
	0:00:00.000001
	```

2. 网上很多资料写的是用 microseconds 获取响应时间，再除以1000*1000 得到时间为秒的单位，当请求小于 1s 时发现不出什么问题，如果时间超过 1s，问题就来了（很显然，大于1s的时候，只截取了后面的小数部分）

3. 所以获取响应时间的正确姿势应该是：`r.elapsed.total_seconds()`，单位是秒。

### timeout 超时

1. 如果一个请求响应时间比较长，不能一直等着，可以设置一个超时时间，让它抛出异常

2. 如下请求，设置超时为0.5s，那么就会抛出异常：requests.exceptions.ConnectTimeout: HTTPConnectionPool

	```python
	>>> import requests
	>>> r = requests.get("http://cn.python-requests.org/zh_CN/latest/", timeout=1)
	>>> print(r.elapsed)
	xxxx
	>>> print(r.elapsed.total_seconds())
	xxx
	>>> print(r.elapsed.microseconds)
	xxx
	```



## [26-参数关联(JSESSIONID)](https://mp.weixin.qq.com/s/AmIfZgLS-uOQd7ZEZ41bcA)

### 前言

参数关联是接口测试和性能测试最为重要的一个步骤，很多接口的请求参数是动态的，并且需要从上一个接口的返回值里取出来，一般只能用一次就失效了。

最常见的案例就是网站的登录案例，很多网站的登录并不仅仅只传`username`和`pwd`两个参数，往往有其他动态参数。

### 登录参数

首先分析下目标网站：学信网：https://account.chsi.com.cn/passport/login 的登录接口请求参数。先随便输入账号和密码，使用 fiddler 抓包查看请求参数，用两个参数是网页自动给的参数（用户没输入）

lt:LT-269530-RIkaD7y6sB6dfBwdX56cfaifqxElxx

execution:e1s1

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAOQxCPX0pnbTqMD2Z42U4zzzof3ib6ku14E7Cc8ouYAzD7R4rLSAHZribib9GHXDXt426ic1qWPnIN1A/640)

关闭浏览器后，重复上面操作，再抓包看请求参数，会发现变了

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAOQxCPX0pnbTqMD2Z42U4z4uSIll4ZGrkgbwRfrJMudn6BJmNXn0gzy2FoflvkJ4BpsGdNXKXqbA/640)

lt:LT-277623-5ldGTLqQhP4foKihHUlgfKPeGGyWVI

execution:e1s1

备注：execution 参数是表示网站刷新次数，可以刷新下再登录，就变成 e2s1 了。

### 获取接口返回数据

我们想登录的话，必须先得到lt和execution 这两个参数，那么问题来了：这两个参数是从哪来的

打开登录页面：https://account.chsi.com.cn/passport/login 直接刷新，看返回的 HTML 内容

```html
<input class="btn_login" name="submit" accesskey="l" value="登录" tabindex="4" type="submit" title="登录" />

<div class="account-oprate clearfix">
       <a class="find-yhm" href="https://account.chsi.com.cn/account/password!rtvlgname">找回用户名</a>
       <a class="find-mm" href="https://account.chsi.com.cn/account/password!retrive">找回密码</a>
       <a href="https://account.chsi.com.cn/account/preregister.action?from=account-login" class="regist-btn">注册</a>
</div>
<input type="hidden" name="lt" value="LT-279621-fnisPBt0FVGNFrfWzJJqhTEyw6VkfH" />
<input type="hidden" name="execution" value="e2s1" />
<input type="hidden" name="_eventId" value="submit" />
```

接下来从 HTML 中解析出 `value="LT-279621-fnisPBt0FVGNFrfWzJJqhTEyw6VkfH"`和`value="e2s1"`这两个值

```python
import requests
from lxml improt etree
import urllib3
urllib3.disable_warnings()

s = requests.Session()
def get_it_execution():
    result = {}
    loginurl = "https://account.chsi.com.cn/passport/login"
    h1 = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"
    }
    s.headers.update(h1)
    r = s.get(loginurl, verify=False)
    dom = etree.HTML(r.content.decode('utf-8'))
   	try:
        result['lt'] = dom.xpath('//input[@name="lt"]')[0].get("value")
        result['execution'] = dom.xpath('//input[@name="execution"]')[0].get("value")
        print(result)
    except:
        print("lt、execution 参数获取失败")
    return result

if __name__ == '__main__':
    result = get_it_execution()
```

运行结果：

{‘lt’: ‘LT-286330-a6Ogf3rt3Fcwt6XZcuKCa4HHzz0QA3’, ‘execution’: ‘e1s1’}

### JSESSIONID

登录里面实际上会有一个非常重要的 cookies 参数，JSESSIONID=4D98C3F3ED18A2489BD17CA722D19AE8，这个JSESSIONID也是动态的，每次重新打开页面都会变。这个参数也是第一次访问登录页面的时候服务器自动返回过来的：

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAOQxCPX0pnbTqMD2Z42U4zMw4aKYYSicB96WYuaCBSkdfU0OUSQuaCa8rPoUH9jSLYJHfdNKYuqWg/640)

cookies 参数关联实现就非常简单了，直接用`requests.Session()`去get请求就能自动保存了，所以上一步`get_it_execution()`实际上也同步了 cookies 参数

### 参考代码

略

## [32-上传文件时自动判断文件类型(filetype)](https://mp.weixin.qq.com/s/P2A6_ZagGLtZeVykujvNFA)





## [37-模拟ajax异步请求(X-Requested-With:XMLHttpRequest)](https://mp.weixin.qq.com/s/k7qTfKXQWcy6ZLyZbJBHDg)

### 前言

有些接口请求头部带上`X-Requested-With: XMLHttpRequest`,返回数据是 json.如果头部不加这个参数,返回数据是普通 html 文本.

这种头部带上`X-Requested-With: XMLHttpRequest`的是 Ajax 异步请求.

### Ajax 请求

Ajax 即“Asynchronous Javascript And XML”(异步 Javascript 和 XML), 是指一种创建交互式、快速动态网页应用的网页开发技术, 无需重新加载整个网页的情况下, 能够更新部分网页的技术.

通过在后台与服务器进行少量数据交换, Ajax 可以使网页实现异步更新. 这意味着可以在不重新加载整个网页的情况下, 对网页的某部分进行更新.

那么服务器如何判断请求来自 Ajax 请求(异步)还是传统的 http 请求(同步)?

1. 传统同步请求参数

   ```http
   accept  */*
   accept-charset  gb2312,utf-8;q=0.7,*;q=0.7
   accept-encoding  gzip,deflate
   accept-language  zh-cn,zh;q=0.5
   cache-control  max-age=0
   connection  keep-alive
   cookie  JSESSIONID=1A3BED3F593EA9747C9FDA16D309AF6B
   keep-alive  300
   user-agent  Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9.0.15) Gecko/2009101601 Firefox/3.0.15 (.NET CLR 3.5.30729)
   ```

2. Ajax 异步请求方式

   ```http
   accept  */*
   accept-charset  gb2312,utf-8;q=0.7,*;q=0.7
   x-requested-with  XMLHttpRequest
   accept-encoding  gzip,deflate
   accept-language  zh-cn,zh;q=0.5
   cache-control  max-age=0
   connection  keep-alive
   cookie  JSESSIONID=1A3BED3F593EA9747C9FDA16D309AF6B
   keep-alive  300
   user-agent  Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9.0.15) Gecko/2009101601 Firefox/3.0.15 (.NET CLR 3.5.30729)
   ```

可以看到 Ajax 请求多了个 `x-requested-with`

### 场景案例

登录禅道网站, 输入账号和密码后, 使用 fiddler 抓包看请求参数, 头部会有个参数: `X-Requested-With: XMLHttpRequest`, 返回的是 json 数据: `'{"result": "success", "locate": "\/zentao\/"}'`

 使用 requests 发请求, 如果头部不带参数:  `X-Requested-With: XMLHttpRequest`:

```python
import requests

url = "http://49.235.x.x:8081/zentao/user-login.html"

body = {
    "account": "admin",
    "password": "yoyo123456",
    "passwordStrength": 1,
    "referer": "/zentao/",
    "verifyRand": "1014015280",
    "keepLogin": 1
}

r = requests.post(url, data=body)
print(r.text)

# 返回 html
# <html><meta charset='utf-8'/><style>body{background:white}</style><script>self.location='/zentao/';

# </script>
```

使用 requests 发请求, 头部带上参数:  `X-Requested-With: XMLHttpRequest`, 模拟 Ajax 异步请求

```python
import requests

url = "http://49.235.x.x:8081/zentao/user-login.html"
h = {
    "X-Requested-With": "XMLHttpRequest"
}

body = {
    "account": "admin",
    "password": "yoyo123456",
    "passwordStrength": 1,
    "referer": "/zentao/",
    "verifyRand": "1014015280",
    "keepLogin": 1
}

r = requests.post(url, data=body)
print(r.text)

# 返回 json
# {"result":"success","locate":"\/zentao\/"}
```

此时就可以返回 json 数据了.

## [40-盘点requests那些不常用(面试经常问)的高级技能](https://mp.weixin.qq.com/s/EKf89MMUBQNEyQDG9vQfsw)

### 前言

面试最喜欢考那些你不常用的功能，因为你用不到这些功能，所以会被你忽略。

### 代理功能

```python
import requests

proxies = {
    "http": "http://10.10.1.10:3128",
    "https": "http://10.10.1.10:1080"
}

requests.get("http://example.org", proxies=proxies)
```

也可以通过环境变量`HTTP_PROXY`和`HTTPS_PROXY`来配置代理。

```shell
$ export HTTP_PROXY="http://10.10.1.10:3128"
$ export HTTPS_PROXY="http://10.10.1.10:1080"

$ python
>>> import requests
>>> requests.get("http://example.org")
```

若你的代理需要使用 HTTP Basic Auth，可以使用 `http://user:password@host/`语法：

```python
proxies = {
    "http": "http://user:pass@10.10.1.10:3128",
}
```

要为某个特定的连接方式或主机设置代理，使用`scheme://hostname`作为 key，它会针对指定的主机和连接方式进行匹配。

```python
proxies = {'http://10.20.1.128': 'http://10.10.1.10:5323'}
```

注意：代理 URL 必须包含连接方式。

### 关于 https 证书

https 请求需要用到 SSL 证书,平时都是让大家简单省事一点, 设置 `verify=False`来忽略 SSL 证书的校验.

Requests 可以为 HTTPS 请求验证 SSL 证书，就像 Web 浏览器一样。SSL 验证默认是开启的，如果证书验证失败，Requests 会抛出 SSLError：

在该域名上我没有设置 SSL，所以失败了。但 Github 设置了 SSL：

可以为`verify`传入 CA_BUNDLE 文件的路径，或者包含任何可信任 CA 证书文件的文件夹路径：

```python
>>> requests.get('https://github.com', verify='/path/to/certfile')
```

或者将其保持在会话中：

```python
s = requests.Session()
s.verify = '/path/to/certfile'
```

注解：如果 verify 设为文件夹路径，文件夹必须通过 OpenSSL 提供的 c_rehash 工具处理

还可以通过 REQUESTS_CA_BUNDLE 环境变量定义可信任 CA 列表。

如果你将`verify`设置为 False, Requests 也能忽略对 SSL 证书的验证。

```python
>>> requests.get('https://kennethreitz.org', verify=False)
<Response [200]>
```

默认情况下，`verify`是设置为 True 的，选项`verify`仅应用于主机证书。

客户端证书：

你也可以指定一个本地证书用作客户端证书，可以是单个文件（包含密钥和证书）或一两个包含两个文件路径的元祖：

```python
>>> requests.get('https://kennethreitz.org', cert=('/path/client.cert', 'path/client'))
<Response [200]>
```



### 获取响应时间



### timeout 超时



### 超时重试



### Ajax 异步请求



### requests 库一些常用插件

