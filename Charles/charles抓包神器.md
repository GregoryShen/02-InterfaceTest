# [【软件测试教程】接口测试必会-charles抓包神器](https://www.bilibili.com/video/BV1564y1u7fD)

## charles介绍及产品对比

### Charles是什么

​		Charles是一个HTTP代理/HTTP监听器/反向代理,使开发人员能够查看其机器和Internet之间的所有HTTP和SSL/HTTPS流量.这包括请求、响应和HTTP标头(包含cookie和缓存信息)

### 同类抓包工具对比

* 常见代理工具:Charles、Fiddler、Burpsuite
* Fiddler:对Windows支持很全面,但对Mac支持不是很友好
* Burpsuite:for hacker 偏黑客级别的代理 工具,适合安全测试
* Charles:友好的支持Mac和Windows版本,测试工程师常用

## charles安装及证书配置

### Charles 安装

* 下载地址:https://www.charlesproxy.com/download

### Charles 证书安装

* 为什么要安装证书
* 由于现在接口都是https协议,如果不安装https证书,抓包会出现unknown
* 安装方式:
  * PC/Mac:Help->SSL Proxying->Install Charles Root Certificate
  * Android/iOS: Help->SSL Proxying->Install Charles Root Certificate on a mobile device or remote browser

## http协议的组成(请求、响应)

### HTTP协议的组成

```html
target
	method: get post put delete head  # 请求方式
	url: host port path
request
	header: host cookie user-agent
  get query		# 请求参数
  body
		form
		json xml kv
response
	status code		# 状态码
	content
```

### HTTP请求篇

* 第一部分:请求行,用来说明请求类型,要访问的资源以及所使用的HTTP版本

  ​	POST说明请求类型为POST, [/v2/api]为要访问的资源,该行的最后一部分说明使用的是HTTP1.1版本

* 第二部分:请求头部,紧接着请求行(即第一行),之后的部分,用来说明服务器要使用的附加信息

  ​	从第二行起为请求头部,HOST将指出请求的目的地.User-Agent,服务器端和客户端脚本都能访问它,它是浏览器类型检测逻辑的重要基础.该信息由你的浏览器来定义,并且在每个请求中自动发送等等

* 第三部分:空行,请求头部后面的空行是必须的

  ​	即使第四部分的请求数据为空,也必须有空行

* 第四部分:请求数据也叫主体,可以添加任意的其他数据

### HTTP 响应篇

* 第一部分:状态行,由HTTP协议版本号,状态码,状态消息 三部分组成

  ​	第一行为状态行,(HTTP /1.1) 表明HTTP版本为1.1版本,状态码为200, 状态消息为(OK)

* 第二部分:消息报头,用来说明客户端要使用的一些附加信息

  ​	第二行和第三行为消息报头:

  ​		Date: 生成响应的日期和时间

  ​		Content-Type: 指定了MIME类型的HTML(text/html),编码类型是UTF-8

* 第三部分: 空行,消息报头后面的空行是必须的

* 第四部分: 响应征文,服务器返回给客户端的文本信息

  ​    空行后面的html部分为响应正文

### 常见状态码

* HTTP/1.1 中定义了5类状态码,状态码由三位数字组成,第一个数字定义了响应的类别
  * 1XX 提示信息 - 表示请求已被成功接收, 继续处理
  * 2XX 成功 - 表示请求已被成功接收,理解,接受
  * 3XX 重定向 - 要完成请求必须进行更近一步的处理
  * 4XX 客户端错误 - 请求有语法错误或者请求无法实现
  * 5XX 服务器错误 - 服务器未能实现合法的请求
* 常见的状态码
  * 200 OK		// 客户端请求成功
  * 400 Bad Request     // 客户端请求有语法错误,不能被服务器所理解
  * 401 Unauthorized     // 请求未经授权,这个状态码必须和WWW-Authenticate 报头域一起使用
  * 403 Forbidden        // 服务器收到请求,但是拒绝提供服务
  * 404 Not Found.       // 请求资源不存在,eg: 输入了错误的URL
  * 500 Internal Server Error    // 服务器发生不可预期的错误
  * 503 Server Unavailable      // 服务器当前不能处理客户端的请求,一段时间后可能恢复正常

## mock实战

三种mock方式:

都在Tools下面:

1. Map Local
2. Map Remote
3. Rewrite
   * 重写功能可以删除、修改、增加、head、body、url、response 等参数
   * Tools->Rewrite 下,需要增加一个接口地址,增加一个重写规则
   * Rewrite 作用:验证线上的一些内容(因为不可以修改线上的数据库)

## 弱网模拟

* Proxy — Throttle Settings — Enable Throttling
* Only for selected 表示仅选择的域名做弱网
* Throttle preset 表示要模拟的网速,可以改上行和下行带宽,还有丢包率

### 弱网参数详解

* Bandwidth(带宽) [吞吐量]
  * 带宽定义数据可以传送超过时间上限, 这是千比特每秒指定.可以指定上载和下载链接的不同带宽限制
* Utilisation(利用)
  * 利用率是总带宽的百分比,可以在任何一个时间使用.它只是作为可用带宽的缩放因子.对于大多数现代互联网连接利用率始终是100%
* Round-trip Latency(请求往返延迟)[延时]
  * 往返延迟测量客户端和远程服务器之间的第一次往返通信的毫秒延迟.它用于客户端向服务器和服务器向客户端的每次请求
* MTU(最大传输单元)
  * 在任何传输的TCP数据包的最大尺寸.指定MTU不改变的可用带宽,但允许Charles在MTU分配带宽大小的地方在每个传输包分割的现实水平
* Reliability(可靠性) [丢包]
  * 可靠性是衡量连接完全失败的可能性.这是非常有用的模拟不可靠的网络条件.可靠性是指定为成功发射10kib消息的可能性,所以,值为50%意味着所有10kib传输一半会成功.较大的邮箱或更小的消息或多或少都有可能失败,所以20kib传输将只有25%的成功率和5kib传输成功率约为70%
* Stability(稳定性) [抖动]
  * 稳定性是衡量一个连接的可能性是不稳定的,因此降低了质量.这是非常有用的模拟网络,如移动网络,定期连接质量差.如果连接不稳定,则连接的质量会在不稳定的质量范围内随机下降.此质量值,然后应用作为另一个缩放因子的可用带宽
* Unstable quality range(不稳定质量范围)
  * 此处设置主要针对于Stability中设置的范围

























