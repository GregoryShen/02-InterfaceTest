## [1-安装与使用](https://mp.weixin.qq.com/s/OyJzQ4W2RuWNeDG7h73sCw)





## [2-发 post 请求(json和urlencoded)](https://mp.weixin.qq.com/s/oqyKqi3PDp3bN--jh_iESQ)

### 前言

使用 postman 发送 http 协议 post 请求, 两种请求类型 `application/json` 和 `application/x-www-form-urlencoded`

### application/json

请求参数是 json 格式, 这种是最常见的, 以登录接口为例

接口名称: 用户账户登录

接口地址: /api/v1/login

请求方式: POST

请求参数:

|  字段名  |  变量名  | 必填 |  类型  | 示例值 |   描述   |
| :------: | :------: | :--: | :----: | :----: | :------: |
|  用户名  | username |  是  | String |  test  | 用户账户 |
| 用户密码 | password |  是  | String | 123456 |   密码   |



### application/x-www-form-urlencoded



## [3-全局变量和环境变量](https://mp.weixin.qq.com/s/EdRuJx8jdIspwtUqmuw6eg)

### 前言

当接口请求中有多个地方用到同一个值时，可以设置变量，在脚本中引用变量。

postman 可以设置全局变量和环境变量，这样只需要改一个地方，其它脚本引用变量都会生效。

### 使用变量

在多个地方使用相同的值时，使用变量会非常有用。例如，如果多个请求中具有相同的 base_url, 但 base_url 可能会更改，则可以将其存储在变量中。如果 base_url 更改，则只需要更改变量值，无论使用变量名称的位置如何，它都会在整个集合中反映出来。相同的原则适用于您请求中重复数据的任何部分。

postman 支持的变量作用范围：

* Global

	全局变量使您可以访问集合，请求，测试脚本和环境之间的数据。全局变量在整个工作空间中都可用

* Collection

	集合变量可在集合中的整个请求中使用，并且独立于环境，因此请不要根据所选环境进行更改。

* Environment

	环境变量可以针对不同的环境定制处理，例如本地开发与测试或生产

* Data

	数据变量来自外部 CSV 和 JSON 文件，以定义在通过 Newman 或 Collection Runner 运行集合时可以使用的数据集

* Local

	局部变量是临时的，只能在请求脚本中访问。局部变量值的范围仅限于单个请求或收集运行，并且在运行完成后不再可用。

<img src="https://assets.postman.com/postman-docs/var-scope.jpg" style="zoom:80%;" />

变量的作用范围如图所示：

<img src="https://assets.postman.com/postman-docs/Variables-Chart.png" style="zoom:80%;" />

### 设置为变量（variable）

如果我们要把部分值设置为变量，可以先选中这部分内容，这时会弹出`Set as variable`选项。

点`Set as variable` > `Set as a new variable`。变量的范围可以选全局变量/环境变量/集合变量

### 全局变量（Globals)

有一个注册接口`http://localhost:8201/api/v1/register`， 还有一个登录的接口`http://localhost:8201/api/v1/login`，前面一部分是一样的，这部分可以用一个变量 base_url 来定义，这个环境是可能会变的。

定义一个全局变量，设置变量名称为 base_url。

点`set variable` 按钮,此时选中的部分会自动变成`{{base_url}}`引用变量的值

### 查看和编辑变量

点开眼睛按钮，查看变量

在 Global是 区域点 Edit 按钮，可以自己编辑添加一些变量

添加 username 和 password 两个变量

请求 body 引用变量

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNr5jAQnGMCWic97eUV7QTz9ZhWbGRmzewibLTFx1c89d1KvKU55gricsCA/640)

### 环境变量（Environment）

当我们有多套测试环境的时候，比如有开发环境，测试环境，联调环境，预发布环境等，每套环境的测试数据不一样，至少 base_url 地址是不一样的。

在运行的时候可以选择不同的环境运行，默认是： No Environment

新增一个测试环境，左上角 New - Environment

设置环境名称和变量（此时 base_url 地址应该从全局变量里移除）

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNlCA7eny3svmso3vo0soTI3c0ffPl9sFDcq6yD2Mher4Fu1jhOzqUSw/640)

点击 add 添加成功。

运行的时候选‘test 环境’运行。

点眼镜按钮，查看当前环境的环境变量和全局变量（**==全局变量是对任意环境都会生效==**）

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNyJufOlgiaELZd7u8PdFUg2rABjTlrROFf3BZNxibd3LFnWVRjYbuibd8Q/640)

## [4-集合变量(collection variables)的使用](https://mp.weixin.qq.com/s/bPDgnw-tlKvMad2ysdrnKg)

### 前言

postman 定义环境变量和全局变量用的比较多，当使用多个集合（collection）的时候，每个集合也可以分别定义不同的集合变量。

一个集合可以看成一个小的项目，不同集合定义不同变量很有必要。

### 注册接口为例

接口名称：用户注册

接口地址：/api/v1/register

请求方式： POST

请求参数：

|  字段名  |  变量名  | 必填 |  类型  |   示例值   | 描述             |
| :------: | :------: | :--: | :----: | :--------: | ---------------- |
|  用户名  | username |  是  | String |    test    | 用户账户长度xx位 |
| 用户密码 | password |  是  | String |   123456   | 密码长度6-16位   |
|   邮箱   |  email   |  否  | String | 123@qq.com | xxx看不清        |

请求示例：

```http
POST http://localhost:8201/api/v1/register HTTP/1.1
User-Agent: PostmanRuntime/7.26.8
Accept: */*
Postman-Token: 807ce386-f92b-4647-a80f-77f050c5a314
Host: 49.235.92.12:7005
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Length: 42
Content-Type: application/json

{"username": "test_yoyo","password":"123456"}
```

### 定义集合变量

选中左侧集合， Edit 编辑

Variables 变量编辑

定义两个变量 register_name 和 register_password

注册接口引用 collection 变量

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczN0LZia16gyYwvEhUk3NJ59yFyhLDC0tIlnksVnCoh9qpPbpZdxvN9nfg/640)



## [5-Test脚本中自定义变量(参数关联)](https://mp.weixin.qq.com/s/VYS3FyqlOJrabqKAIu-6cg)

### 前言

上个接口返回 token，下个接口需在请求头部传 token，这就是我们经常说的参数关联。Postman 可以在 Tests 脚本中自定义变量。

### 查询个人信息接口

需用户先登录，返回 token

```python
{
    "code": 0,
    "msg": "login success!",
    "username": "test",
    "token": "ce5087209dd8abca2e93e8457252056243c0aded"
}
```

查询个人信息即可欧请求示例

```http
GET http://localhost:8201/api/v1/userinfo HTTP/1.1
Content-Length: 0
Authorization: Token 2439b83901810851e273b494c29df357cbe2ed92
```

### Tests 脚本中自定义变量

打开登录接口，在 Tests 区域写 JavaScript 脚本解析返回的 Response 对象，从 json 里面提取 token 并设置为环境变量

```javascript
// response 解析 json
jsonData = pm.response.json();
// 设置为环境变量
pm.environment.set("token", jsonData.token);
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNrJ78goYBweYJ56ssEW3icsCl3zNPfUMgBNN8libGe1XpGK1h7nbJmZpQ/640)

运行后点开眼睛按钮，会发现环境变量里新增一个 token 的变量

token 变量没初始值（initial_value），但会有当前值（current_value）

### 引用变量

引用变量：{{token}}

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNVCWYx4wibmjH7EJ7XYgLVUKibOZ8RxBN9KhnhYj7VHqGcWROYkT7La2g/640)

先执行登录接口后获取到 token，再执行查询接口就可以查询成功了。

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCTpX5PxfXfoTNf5EyEEiczNr1icRAXFMXFHl0PRCYM4iaav2N195lCQney8A56CTGDvoRq0icib0DaPrA/640)

### 其他变量设置

```javascript
// 使用 pm.globals 来定义一个全局变量
pm.globals.set("variable_key", "variable_value");

// 使用 pm.collectionVariables 定义集合变量
pm.collectionVariables.set("variable_key", "variable_value");

// 使用 pm.environment 定义的环境变量（在当前选择的环境）
pm.environment.set("variable_key", "variable_value");

// 可以 unset 用来删除变量
pm.environment.unset("variable_key");
```



### 在脚本中使用变量



### 访问变量







## [6-引用随机变量(`$guid`,​`$timestamp`,`$randomInt`)](https://mp.weixin.qq.com/s/PEyhKoVWb75jQ40t5YR8Ew)





## [7-参数化引用外部文件(txt/csv/json)测试数据](https://mp.weixin.qq.com/s/H71wYnGv5Ilr1Rz6Q-jN5Q)





## [8-设置断言(Tests脚本编写)](https://mp.weixin.qq.com/s/1EPaVP7tY8n1Mh0C7EnPpg)

### 前言

### Tests 编写

### 校验返回的 body 是 json 格式

### 校验 body 具体内容

### 校验状态码和返回头部

### 校验返回头部参数

### 断言返回值与变量相等





## [9-点 code 按钮生成代码段](https://mp.weixin.qq.com/s/FsNis9W4ZgT_1aszT9yc_Q)





## [10-请求前参数预处理(pre-request)](https://mp.weixin.qq.com/s/xMNrwODVpQcYz0AEIXbhXg)





## [11- sign 签名预处理(pre-request)](https://mp.weixin.qq.com/s/Kkmzcd8QQAeN3Cg2eTzntg)





## [12-预处理(pre-request) 发送请求](https://mp.weixin.qq.com/s/QctsaH4xyQMO6V6x93g4xQ)





## [13-cookies 管理器](https://mp.weixin.qq.com/s/bCATezHFdZZYRrM7b1zEdQ)





## [14-Windows上如何使用postman进行抓包（模拟fiddler抓包)](https://mp.weixin.qq.com/s/r8oGSVayl2nZMMJCRTbnWw)





## [15-构建请求工作流（setNextRequest）](https://mp.weixin.qq.com/s/Y246HD_Wg7pARqqnCdCQIg)





