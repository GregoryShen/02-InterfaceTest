

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





## [15-multipart/form-data表单提交](https://mp.weixin.qq.com/s/8-E6g4vmzuGe5-yrbE5axQ)





## [16-multipart/form-data上传多个附件](https://mp.weixin.qq.com/s/UDO9FIlO0DW-P4d7kXF1sA)







## [17-响应时间与超时(timeout)](https://mp.weixin.qq.com/s/7HzCRTzLiCMESHHiw58RtA)





## [26-参数关联(JSESSIONID)](https://mp.weixin.qq.com/s/AmIfZgLS-uOQd7ZEZ41bcA)

### 前言



### 登录参数



### 获取接口返回数据



### JSESSIONID



### 参考代码



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



### 代理功能



### 关于 https 证书

https 请求需要用到 SSL 证书,平时都是让大家简单省事一点, 设置 `verify=False`来忽略 SSL 证书的校验.



### 获取响应时间



### timeout 超时



### 超时重试



### Ajax 异步请求



### requests 库一些常用插件

