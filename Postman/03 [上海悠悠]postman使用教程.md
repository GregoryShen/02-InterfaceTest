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

// 局部变量是使用以下语法再请求脚本中设置的临时值
pm.variables.set("variable_key", "variable_value");
```

局部变量不会在会话之间持久存在，但是允许您在执行请求或收集/监视运行期间临时覆盖所有其他作用域。例如，如果您需要为本地运行的单个请求或集合处理临时测试值，并且不希望该值与您的团队同步或在请求/集合完成运行后仍然可用，则可以使用局部变量。

### 在脚本中使用变量

可以使用表示范围级别和`.get`方法的对象在脚本中检索变量的当前值：

```javascript
// access a variable at any scope including local
pm.variables.get("variable_key");

// access a global variable
pm.globals.get("variable_key");

// access a collection variable
pm.collectionVariables.get("variable_key");

// access an environment variable
pm.environment.get("variable_key");
```

使用`pm.variables.get()`在脚本中访问变量提供更改变量的作用域，而不会影响你的脚本功能的选项。此方法将返回当前优先级最高（或范围最窄）的任何变量。

### 访问变量

您可以在 Postman 用户界面中使用双花括号来引用变量。例如，要在请求身份验证设置中引用名为“用户名”的变量，可以使用以下语法，在名称周围使用双花括号：`{{username}}`.运行请求时，邮递员将解析该变量并将其替换为当前值。例如您可能有一个请求 URL 引用一个变量，如下所示：

```javascript
http://pricey-trilby.glitch.me/customer?id={{cust_id}}
```

cust_id 请求运行时，邮递员将发送您当前为该变量存储的任何值。如果 cust_id 当前为3，则请求将被发送到以下包含查询参数的 URL：

```javascript
http://pricey-trilby.glitch.me/customer?id=3
```

或者可以在请求 body 通过变量的引用来访问变量：

```javascript
{"customer_id": "{{cust_id}}"}
```

关于变量更多语法可参考：https://learning.postman.com/docs/sending-requests/variables/#using-data-variables

## [6-引用随机变量(`$guid`,​`$timestamp`,`$randomInt`)](https://mp.weixin.qq.com/s/PEyhKoVWb75jQ40t5YR8Ew)





## [7-参数化引用外部文件(txt/csv/json)测试数据](https://mp.weixin.qq.com/s/H71wYnGv5Ilr1Rz6Q-jN5Q)





## [8-设置断言(Tests脚本编写)](https://mp.weixin.qq.com/s/1EPaVP7tY8n1Mh0C7EnPpg)

### 前言

当一个接口发送请求有返回结果后，可以在 postman 里面的 Tests 写脚本断言结果是否符合预期。

Tests 是接口返回 response 之后的脚本操作，可以使用 JavaScript 为 Postman API 请求编写 Tests 脚本。

### Tests 编写

Tests 可以添加到单个请求，文件夹和集合中，这里以单个请求为例。

登录接口返回：

```json
{
    "code": 0,
    "msg": "login success!",
    "username": "test",
    "token": "580406b0df935496f96313fc98ae7e18d39d62af"
}
```

### 校验返回的 body 是 json 格式

首先可以校验返回的 body 是 json 格式：

```javascript
pm.test("response must be valid and have a body", function() {
    pm.response.to.be.ok;
    pm.response.to.be.withBody;
    pm.response.to.be.json;
});
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhjCQHmvQ1OVjZiboLlvMlpAiaG5A0q0DPOngjV4QFO2xmIqWKayiat96Og/640)

运行后可以看到接口返回 TestResults 位置显示 PASS，说明此校验通过

### 校验 body 具体内容

上面是直接`pm.response.to.be`方式对 response 对象校验的，也可以用`pm.expect(actual_result).to`方式对提取的返回结果校验

```javascript
// 校验 code 为 0
pm.test("response code must to be 0", function () {
    pm.expect(pm.response.json().code).to.equal(0);
});

// 校验 msg 为 login success!
pm.test("response msg must to be login success!", function () {
    pm.expect(pm.response.json().msg).to.equal("login success!");
});

// 校验 token 长度为 40 位
pm.test("response token length must to be 40", function() {
    pm.expect(pm.response.json().token).to.lengthOf(40);
});
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhiaf8kq8vOA6OzKxPyROu76U6FuVj49V2iaIVt0lZlNA8nibTrnhibG9HFA/640)

### 校验状态码和返回头部

校验返回状态码是200

```javascript
pm.test("Status test", function () {
    pm.response.to.have.status(200);
});

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhdFIvz19aP5MADAcpgTXPVVwcZ2CFbT4Hraryic3qBpMFEKTl75YaEUg/640)

### 校验返回头部参数

校验 `Content-Type`在返回头部

​```javascript
pm.test("Content-Type header is present", () =>{
    pm.response.to.have.header("Content-Type");
});
```

校验返回的头部`Content-Type`值为`application\json`

```javascript
pm.test("Content-Type header is application/json", () => {
    pm.expect(pm.response.headers.get("Content-Type")).to.eql('application/json');
});
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhg0aUfawPdTmdBiaqTYmYozjZqs2bf3BMo7jHXyuy1EGoOefqCXdNq0w/640)

### 断言返回值与变量相等

如果我前面登录的 body 参数引用了环境变量 username

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhUOx89cGVTy7giatHAJwnXibK6Vm1QAZjsakvR3E8WUYpHdUyHokA9Dibw/640)

接口返回的 json 数据又有这个账号名称，想断言结果返回的值和变量 username 相等，于是可以先获取环境变量值：

```javascript
pm.environment.get("name");
```

于是脚本这样写：

```javascript
pm.test("Response property matches environment variable", function() {
    pm.expect(pm.response.json().username).to.eql(pm.environment.get("username"));
});
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhwibVlnPKJZ0QpFFh3QcicibVWtYchL6xQa4A1tloiaRECBjYGDG6f9gkkA/640)

## [9-点 code 按钮生成代码段](https://mp.weixin.qq.com/s/FsNis9W4ZgT_1aszT9yc_Q)

### 前言

postman 可以生成各种语言的代码发送接口请求，对于会使用 postman 但 Python 脚本还不熟练的小伙伴会很有帮助。

### code

postman 上接口调试没问题后，可以点右侧 code 按钮

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhq0ibPXYTTvgqUkqdba7ycCYwc8tdJAxj2aic0VqQ2zD1Krib8niaMH5bibQ/640)

可以生成 http 协议请求报文，这对排查问题非常方便

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhAbHC7ZktOS9WLKQLibH2cjxqibZyA8UaebH4XicicuO9WT4nRXJe3NStfA/640)

### 生成 Python 代码段

可以选择不同的开发语言，选择 Python Requests 请求

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhqGs6sM80KiaYuSDkr1JIQohyoZlic5jkyiaOytmG2Aded6t974yRu4icIg/640)

点 Copy to Clipboard 按钮会全部复制出来。

### curl 请求

也可以生成 curl 请求

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhP1NPr41sAbU9JQuge7wdtMLvzN2gYYp0DLnKfCmibKabrIwfRBx1PcQ/640)

### postman 支持的语言和框架

| Language   | Framework              |
| ---------- | ---------------------- |
| cURL       | cURL                   |
| HTTP       | (Raw HTTP request)     |
| JavaScript | Fetch                  |
| JavaScript | jQuery                 |
| JavaScript | XHR                    |
| Python     | http.client (Python 3) |
| Python     | Requests               |
| Shell      | Httpie                 |
| Shell      | wget                   |

## [10-请求前参数预处理(pre-request)](https://mp.weixin.qq.com/s/xMNrwODVpQcYz0AEIXbhXg)

### 前言

接口测试的时候，有些参数并不是固定的，需动态处理下，比如前面讲的注册时候在字符串后面加时间戳，可以通过动态变量来生成。

有些复杂的参数处理，如果系统没提供对应的动态变量，我们可以自己写一个请求前参数处理，通过 postman 的 pre-request 功能来实现

### 注册接口

前面讲到在请求参数重，引用时间戳变量:`{{$timestamp}}`可以动态生成请求的参数

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhtave8jsTDhe2aTibd9EUNW3icgrsFs17fOJgicLNYIyicaRVqQIckBDQgw/640)

接下来再讲通过 postman 的 pre-request 功能对请求前参数预处理来实现

### pre-request 预处理请求参数

注册账号用“test”名称会发现已经被注册了, username 引用环境变量中的 username 变量。

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhv2Z2vibgxTO0dJcjiaRRahJ5zsjGppwcV94bsJ5ibNpnBdiaJPcqZKC7Qg/640)

于是在 pre-request script 对请求参数预处理，先把 username 变量不要写死，引用另外一个变量{{env_username}}

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhNqeibHLYw1IF4OrsnoMgo8icwRa32aeORkyszWrgZyGiaRIiahPE5ZSWpw/640)

env_username 变量在 pre-request script 脚本里定义

```javascript
// 获取时间戳
var timestrip = Math.round(new Date()/1000).toString()
console.log(timestrip);

// 设置环境变量 env_username 的值
pm.environment.set("env_username", 'test_'+Math.round(new Date()/1000));
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhPeOHBz2l7yD4YLoic2rzicRrIo427mMFF2sjz3d5T1KtEKdEM2ZqpL9A/640)

console.log() 可以在 console 查看日志

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhX8OkKQoa7DERIc4wgOAx6mICe1Oib58l8IZj2cYnmwkydpyicuzRXkSw/640)

点 code 按钮可以快速查看 http 代码，看请求参数是否正确

### pre-request script 中常用代码

```javascript
pm.globals.unset("variable_key");	//清除全局变量
pm.environment.unset("variable_key");	// 清除环境变量
pm.globals.get("variable_key");		// 获取全局变量
pm.variables.get("variable_key");	// 获取一个变量
pm.environment.get("variable_key");	// 获取环境变量
pm.sendRequest("https://postman-echo.com/get", function (err, response){
    console.log(response.json());
});	// 发送一个请求
pm.globals.set("variable_key", "variable_value");	// 设置环境变量
```

如果不会写也没关系，点右侧可以快速生成代码

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCwCU5yWxFia1G1K1LH57fRhAJ6Rpc9sbSLWia4tbY4wCBwAd764HVZpUxgMKCKXqGk7LCCIj14x5Hw/640)

更多 Pre-request Scrip 脚本参考官网https://go.pstmn.io/docs-prerequest-scripts

## [11- sign 签名预处理(pre-request)](https://mp.weixin.qq.com/s/Kkmzcd8QQAeN3Cg2eTzntg)





## [12-预处理(pre-request) 发送请求](https://mp.weixin.qq.com/s/QctsaH4xyQMO6V6x93g4xQ)

### 前言

可以使用`pm.sendRequest`方法从“pre-request”或“Tests”脚本异步发送请求。

如果您要执行计算或同时发送多个请求，而不必等待每个请求完成，则可以在后台执行逻辑。

### pre-request 发送请求

点`Send a request`快速生成一个请求示例

```javascript
pm.sendRequest("https://postman-echo.com/get", function (err, response) {
    console.log(response.json());
});
```

* `pm.sendRequest`是发送一个请求
* function 中的 err 表示请求返回的错误信息
* response 表示响应内容
* `console.log()`是控制台输出日志

发送一个 post 请求示例

```javascript
// Example with a full-fledged request
const postRequest = {
    url: 'https://postman-echo.com/post',
    method: 'POST',
    header: {
        'Content-Type': 'application/json',
        'X-Foo': 'bar'
    },
    body: {
        mode: 'raw',
        raw: JSON.stringify({key: 'this is json'})
    }
};


pm.sendRequest(postRequest, (error, response) => {
    console.log(error ? error : response.json());
});
```

参数说明：

* `const`是 js 中用来定义变量的关键字，由 const 定义的变量不可修改，而且必须初始化
* `url`表示要发送的请求的 url 地址
* `method`指定请求方法 GET/POST
* `header`定制请求头信息，传 json 格式的数据的话，需定义请求头为`Content-Type:application/json`
* `body`表示 post 请求 body 参数
* `JSON.stringify()`方法是将一个 JavaScript 值（对象或数组）转换为一个 JSON 字符串

### 更多示例

以下是官方文档给的示例：https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/

```javascript
// Example with a plain string URL
pm.sendRequest('https://postman-echo.com/get', (error, response) => {
    if (error) {
        console.log(error);
    } else {
        console.log(response);
    }
});

// Example containing a test
pm.sendRequest('https://postman-echo.com/get', (error, response) => {
    if (error) {
        console.log(error);
    }
    
    pm.test('response should be okey to process', () => {
        pm.expect(error).to.equal(null);
        pm.expect(response).to.have.property('code', 200);
        pm.expect(response).to.have.property('status', 'OK');
    });
});
```

### 请求定义和响应结构参考文档

[Request 请求参数参考文档](http://www.postmanlabs.com/postman-collection/Request.html#~definition)

[Response 返回参考文档](http://www.postmanlabs.com/postman-collection/Response.html)

## [13-cookies 管理器](https://mp.weixin.qq.com/s/bCATezHFdZZYRrM7b1zEdQ)

### 前言

提供了一个 MANAGE COOKIES 模式，使得可以编辑每个域关联的 cookie。模拟 Web 网站登录的时候，一般都会有 cookies。

### Cookies 关联

以登录禅道网站为例，没访问之前点开 Cookies 管理器，这时候 MANAGE COOKIES 是空的。

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u86E9g7vM41t2TiaNKcxdd6456kmtUkhibl3lXumlOTbVn28AQtvoD5My6A/640)

发一个 get 请求，访问登录首页

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u864Z8MfWqAzoB9oksjnmqe9ZLFjNJX0lQicgXnibgt6HZqLib0bvaWF3xZg/640)

此时服务端返回了4个 cookies

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u86KD0iawO4o4Bq6rsRLz4m0t8kD8CpeMyRLA8bSuhBIg9EXibrTQvbe7pg/640)

服务端返回的 cookies 会自动缓存，点开 Cookies 管理器

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u86tHVlmwdaKPX7frR0DyMGeG6AHAoZgyUccnOwFIWibIIZ1dfKULOGoCQ/640)

此页面可以添加/编辑/删除 cookies

### 登录网页

发送 post 请求登录网页，请求头部参数

Content-Type:application/x-www-form-urlencoded
X-Requested-With:XMLHttpRequest

登录成功，接口返回：`{"result":"success","locate":" \/zentao\/"}`

查看 console 会发现 post 请求会自动带上前面缓存的 cookies

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u86fpma7zw0U1dcM6GMglA01LZYeQf5KDZMrHqx508tsd7qSGUED9FjWg/640)

点 code 按钮查看 HTTP 协议也可以看到：

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyCg7SoAOguevnImbWdw8u86VbLf4JSE2ddFQdCOJJ9Of6xp0Xm55ic164IoAKHOSjEIS5Nv2ex5Y8w/640)



## [14-Windows上如何使用postman进行抓包（模拟fiddler抓包)](https://mp.weixin.qq.com/s/r8oGSVayl2nZMMJCRTbnWw)





## [15-构建请求工作流（setNextRequest）](https://mp.weixin.qq.com/s/Y246HD_Wg7pARqqnCdCQIg)

### 前言

postman 在收集运行时，所有请求都豆浆按照在 Postman 中看到的顺序运行。因此，首先按照文件夹的顺序执行所有请求，然后再执行集合根目录中的所有请求。

可以用内置函数来覆盖此行为`postman.setNextRequest()`，顾名思义，此功能允许您指定下一个运行的请求。

### 工作流

假设有4个接口：注册，test_demo，登录，个人信息，写到一个 Collections

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476tonkn1MLKCsMBFQX0MunlWicCurxAeIiboYbJLSTG57B1jztvSj3kNy6Q/640)

接下来要改变执行顺序：注册-登录-个人信息-test_demo

### setNextRequest

设置下一个要执行的请求：

```javascript
postman.setNextRequest("request_name");
```

在 Tests 中使用示例

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476tuic5RzEBxXqQM6RObmVhhhCIkbjMD4HC4mAfjg3Mows39NHKj1tUruA/640)

### 注册后执行登录

“注册”接口执行后，在 Tests 中写执行第三个请求“登录”

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476tCttJfAmMu9PxXUcD0IzG3B6NGiaVjxtbRLOVgoCguZLQzwBMDqAGVyA/640)

工作流有4个接口，指定了注册后执行登录

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476tJRwcMvfwwN06I6TA81SMT0e3JN0ibf8bibJ8xD0Eaibl9c5x2ytNsJLzg/640)

运行结果是：注册-登录-个人信息，此时会跳过`test_demo`接口

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476t7GnHbZh8OQG0vjF5YEcP5oMia4qoLEbR03pE6Uyb9yCz6GLIfSUuf7Q/640)

也就是说当没指定顺序的时候，会按收集器写的顺序执行，如果指定了下一个请求，会执行下个请求（然后遵循前面规则按顺序往下执行）

### 循环当前请求

`setNextRequest()`执行当前接口名称，导致 Postman 连续运行当前请求。如果我要重复执行100次注册，可以通过 for 循环实现

```javascript
// 重复执行 100 次注册
for (i = 0;i <= 100;i++) {
    console.log(i);
    postman.setNextRequest("注册");
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476tiaYC1eXjg8wd05wibyxtSasuAETSnQdFIZ17JGOIZDViclRWhKeiamjicYA/640)

在 Collection Runner 运行的时候会重复执行100次登录接口

### 停止执行工作流程

```javascript
postman.setNextRequest(null);
```

关于 `postman.setNextRequest()`以下有几个要点：

* 指定后续请求的名称或 ID，收集运行期将负责其余的工作。
* 它可以在预请求或测试脚本中使用。如果有多个分配，则最后一个设置值优先
* 如果`postman.setNextRequest()`请求中不存在，则收集运行器默认为线程执行并移至下一个请求

使用此工作流程时，请记住以下两个事实：

* postman.setNextRequest() 总是在当前请求的末尾执行。这意味着，如果您将此功能放在预请求或测试脚本中任何其他代码块之前，则这些代码块仍将执行。
* postman.setNextRequest() 具有作用域，这是收集运行的来源。如果运行集合，则可以跳转到集合中的任何请求（甚至使用相同语法的文件夹中的请求）。

但是，如果运行文件夹，则范围postman.setNextRequest()仅限于该文件夹。

因此，您可以跳到该文件夹中的任何请求，但不能跳到该文件夹之外的任何请求。
它包括其他文件夹中的请求，以及集合中的根级别请求

### 手动拖动顺序

使用 postman.setNextRequest() 设置工作流实际上不太直观，我们习惯上按看到的顺序执行，在 Collection 区域也可以手动拖动改变顺序：

![](https://mmbiz.qpic.cn/mmbiz_png/qia7WF9xhFyAj9g40pex3aXAffGTP476t08VBNDtO4jicPZXssez3ftTBmUNUY2KcvYlY83eTVt92UVia2uOD67Ng/640)

官方文档参考：https://learning.postman.com/docs/running-collections/building-workflows/



