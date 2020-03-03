## 第6章 mock服务入门到实战

### 6-1 如何在接口开发阶段编写接口测试脚本

### 6-2 mock服务介绍以及实现原理

### 6-3 在case中通过底层函数实现mock

把这个结果模拟返回出来,因为我们现在是没有接口的,那么我们只能通过模拟返回数据,那么如何用`mock`来实现?

首先倒入mock: `import mock`

```python
res = self.run.run_main(url, 'POST', data)
```

既然是调用的`run_main`方法,就要模拟这个方法, 那也就是模拟 `self.run.run_main(url, ‘POST’, data)`的值

那怎样去模拟呢?

```python
mock_data = mock.Mock(return_value=data)  # 把请求值的data作为返回值赋给return_value
```

`mock_data`是一个方法, 打印一下试试看 

```python
print(mock_data)
# 结果是
<Mock id='111111111'>
```

如果需要用`mock_data`那就需要让`mock_data`等于`self.run.run_main`

```python
self.run.run_main = mock_data
res = self.run.run_main(url, 'POST', data)
```

这个时候调用`run_main`就相当于调用`mock_data`了, 由于前面已经规定了`mock_data`的返回值是`data`, 所以`res`的值应该也是`data`

然后后续可以维持不变,继续用`errorcode`字段来判断(因为data里也有`errorcode`)

```python
self.assertEqual(res['errorcode'], 1001, "测试失败")
print("这是第一个case")
```

但是这样比较low, 需要重构封装

### 6-4 重构封装mock服务

```python
import mock
# 模拟mock封装
def mock_test(mock_method,url,method,request_data,response_data):
    mock_method = mock.Mock(return_value=response_data)
    res = mock_method(url,method,request_data)
    return res
```

然后直接调用

```python
from mock_demo import mock_test
# 然后把那几句改写成
res = mock_test(self.run.run_main, url, 'POST', data, data)
```





## 第7章 从接口自动化框架设计到开发

### 7-1 如何设计一个接口自动化框架

----

如果需要设计一个接口的框架，或者是一个模型，我们需要考虑哪些点呢？



思考点

接口地址 请求数据

请求方法 期望结果

还有

header  数据依赖



数据依赖 如果用Unittest方式来解决 就是设定一个全局变量

### 7-2 学习python操作excel获得内容

----

**python操作excel的几种方式**

*原始链接：https://www.cnblogs.com/lingwang3/p/6416023.html*

1. xlrd 主要是用来读取excel文件

```python
import xlrd
workbook = xlrd.open_workbook('文件名')
sheet_names = workbook.sheet_names()
for sheet_name in sheet_names:
    sheet2
```

好像不是很好的样子。。。。这篇有空再看

**python操作excel读写--使用xlrd**

*原始链接：https://www.cnblogs.com/lhj588/archive/2012/01/06/2314181.html*

1. 导入模块

2. 打开Excel读取数据

   ```python
   data = xlrd.open_workbook('文件名')
   ```

3. 使用技巧

   * 获取一个工作表

     ```python
     # 通过索引顺序获取
     table = data.sheets()[0]
     
     # 通过索引顺序获取
     table = data.sheet_by_index(0)
     
     # 通过名称获取
     table = data.sheet_by_name('Sheet1')
     ```

   * 获取整行和整列的值（数组）

     ```python
     table.row_values(i)
     table.col_values(i)
     ```

   * 获取行数和列数

     ```python
     nrows = table.nrows
     ncols = table.ncols
     ```

   * 循环行列表数据

     ```python
     for i in range(nrows):
         print(table.row_values(i))
     ```

   * 单元格

     ```python
     cell_A1 = table.cell(0,0).value
     cell_A4 = table.cell(2,3).value
     ```

   * 使用行列索引

     ```python
     cell_A1 = table.row(0)[0].value
     cell_A2 = table.col(1)[0].value
     ```

   * 简单的写入

     ```python
     row = 0
     col = 0
     
     # 类型0 empty, 1 string, 2 number, 3 date, 4 boolean, 5 error
     ctype = 1 value = '单元格的值'
     
     xf = 0 # 扩展的格式化
     table.put_cell(row, col, ctype, value, xf)
     table.cell(0,0)  # 单元格的值
     table.cell(0,0).value # 单元格的值
     ```

4. demo 代码

   ```python
   import xlrd
   def open_excel(file='file.xls'):
       try:
           data = xlrd.open_workbook(file)
           return data
       except Exception, e:
           print(str(e))
           
   # 根据索引获取Excel表格中的数据
   def excel_table_byindex(file='file.xls', colnameindex=0, by_index=0):
       """
       file: Excel 文件路径
       colnameindex：表头列名所在行的索引
       by_index: 表的索引
       """
       
   ```

   没有写完。。。。

**python script for manipulating excel sheet**

*原始链接：https://stackoverflow.com/questions/11800726/python-script-for-manipulating-excel-sheet*

----

在工具类的包(util)下面创建操作excel的py文件`operation_excel.py`

```python
import xlrd

# 先写low 代码
data = xlrd.open_workbook('../data config/interface.xlsx')  # 提前新建好dataconfig文件夹

# 获得sheet1的内容，这里用的是用索引获取，还可以通过名字获取
tables = data.sheets()[0]

# 打印这个sheet的行数
print(tables.nrows)

# 打印单元格的数据
print(tables.cell_value(2,2))
```

### 7-3 重构操作excel函数

----

执行前一节的代码存在的问题：

1. 不可能把路径写死
2. 我也不知道有多少行

如何重构封装

1. 需要知道sheet 里的内容
2. 执行多少次--> 行数需要拿到
3. 单元格的数据

```python
class OperationExcel:
    def get_data(self, filename, sheet_id):
        data = xlrd.open_workbook(filename)
        tables = data.sheets()[sheet_id]
        return tables
    
if __name__ == '__main__':
    opers = OperationExcel()
    opers.get_data()
```

现在这样做还只是拿到了数据，还是比较麻烦的，希望类实例化的时候就拿到sheet的数据

```python
class OperationExcel:
    def __init__(self,file_name, sheet_id):
        self.data = self.get_data(file_name, sheet_id)
        
    def get_data(self, filename, sheet_id):
        data = xlrd.open_workbook(filename)
        tables = data.sheets()[sheet_id]
        return tables
```

现在存在的问题：实例化的时候就要传进来file_name, sheet_id, 但是用不到， 调用get_data的时候又要传一遍， 那能不能传进来的时候就直接拿来用？

1. 去掉get_data里的参数，同时去掉引用get_data的参数，把参数绑定在self上，在构造函数里初始化
2. 原来引用这两个参数的地方 都相应改成 `self.file_name`、`self.sheet_id`

```python
class OperationExcel:
    def __init__(self,file_name, sheet_id):
        self.file_name = file_name
        self.sheet_id = sheet_id
        self.data = self.get_data()
        
    def get_data(self):
        data = xlrd.open_workbook(self.filename)
        tables = data.sheets()[self.sheet_id]
        return tables
```

觉得还是有问题，现在每次实例化的时候都要传一个file_name和sheet_id，很麻烦，一般我们都是固定的，但是写死也不好，这样别的方法就没法调用了，那就把两个参数的默认值设置成None，如果没传，就用我们写死的，传了，就用传的

```python
class OperationExcel:
    def __init__(self,file_name=None, sheet_id=None):
        if file_name:
            self.file_name = filename
            self.sheet_id = sheet_id
        else:
            self.file_name = '../dataconfig/interface.xlsx'
            self.sheet_id = 0
        self.data = self.get_data()
    
    #获取sheet的内容
    def get_data(self):
        data = xlrd.open_workbook(self.filename)
        tables = data.sheets()[self.sheet_id]
        return tables
    
if __name__ == '__main__':
    opers = OperationExcel()
    print(opers.get_data().nrows)
```

写成这个样子可以验证一下

继续补全其他方法，获取行数和获取单元格数据

```python
class OperationExcel:
    def __init__(self,file_name=None, sheet_id=None):
        pass
    
    #获取sheet的内容
    def get_data(self):
        pass
    
    # 获取单元格的行数
    def get_lines(self):
     	tables = self.data
        return tables.nrows
    
    # 获取某个单元格的内容
    def get_cell_value(self, row, col):
        return self.data.cell_value(row, col)
    
if __name__ == '__main__':
    opers = OperationExcel()
    print(opers.get_lines())
    print(opers.get_cell_value(2,3))
```

验证一下后加的这两个方法的正确性

### 7-4 学习操作JSON文件

----

主要讲的是如何处理请求数据

在dataconfig下再创建一个`login.json`

期望实现： 通过excel中“请求数据”字段的关键字，在login.json文件中查找具体的请求数据

在util 下面新增`operationjson.py`

```python
import json
# 首先需要操作json这个文件
data = json.load('../dataconfig/login.json')
print(data['login'])
```

会报错：

```python
AttributeError: 'str' object has no attribute 'read'
```

说明直接load不可以，需要有一个句柄

```python
import json
# 首先需要操作json这个文件
fp = open("../dataconfig/login.json")
data = json.load(fp)
print(data['login'])
```

现在就可以取出来了

### 7-5 重构JSON工具类

----

```python
class OperationJson:
    
    def read_data(self):
        fp = open("../dataconfig/login.json")
        fp.close()
```

这里就要重提文件操作的部分，因为open打开会占据一定的内存资源，每次使用完毕都要关闭，非常麻烦，所以这里使用`with`

```python
class OperationJson:
    
    def read_data(self):
        with open("../dataconfig/login.json") as fp:
            data = json.load(fp)
            return data
```

现在整个json数据就拿到了，下面一步，我们期望的是：调用这个方法，只需要告诉我一个关键字，就可以返回对应的数据

```python
class OperationJson:
    
    # 读取json文件
    def read_data(self):
        with open("../dataconfig/login.json") as fp:
            data = json.load(fp)
            return data
        
    # 根据关键字获取数据
    def get_data(self):
        # 这个时候需要用到data，就可以像之前那样，把data放进构造函数里
```

```python
class OperationJson:
    
    def __init__(self):
        self.data = self.read_data()
        
    # 读取json文件
    def read_data(self):
        with open("../dataconfig/login.json") as fp:
            data = json.load(fp)
            return data
        
    # 根据关键字获取数据
    def get_data(self, id):
        # 这个时候需要用到data，就可以像之前那样，把data放进构造函数里
        return self.data[id]
        # 这里的id就是关键字
        
# 最后验证一下
if __name__ == '__main__':
    opjson = OperationJson()
    print(opjson.get_data("addcart"))
```

### 7-6 封装获取常量方法

----



### 7-7 封装获取接口数据

----

现在列号已经拿到了，是一个常量，当拿到如”是否执行“列的数据（比如yes）时，不能直接将yes返回出去，否则就要在主程序中判断是否是true，所以对于这种列，在拿到数据的时候，应当对数据做处理，返回合适的数据

应该有一个获取数据的地方，这些数据就是获取Excel里的数据，根据数据的不同，返回不同的东西

在data下新建一个`get_data.py` 的文件

先把需要的东西理一理：

1. 需要操作Excel，所以要导入操作excel那个类
2. 还要导入获取常量的类

```python
from util.operation_excel import OperationExcel
import data_config
```

```python
class GetData:
    # 首先需要知道有多少行，通过index拿到行数
    # 和操作excel 里的行数是一样的
    def __init__(self):
        self.opera_excel = OperationExcel()
        
    # 去获取excel行数，就是我们的case个数
    def get_case_lines(self):
        return self.oprat_excel.get_lines()
    
    # 拿到行数后，第一个看是否执行
    def get_is_run(self,row):
        col = data_config.get_run()
        run_model = self.opera_excel.get_cell_value(row, col)
        flag = None
        if run_model == 'yes':
            flag = True
        else:
            flag = False
        return flag
    
    # 还要判断是否携带header
    def is_header(self, row):
        col = data_config.get_header()
        header = self.opera_excel.get_cell_value(row, col)
        if header == 'yes':
            return 'header'
        else:
            return None
```

现在在`data_config.py`中添加一个方法：获取header的值

```python
def get_header_value():
    header = {
        "header":"1234",
        "cookie":"Mushishi"
    }
```

继续的话，重新修改一下获取header判断为yes的时候，获取header的方式：

```python
def is_header(self, row):
    col = data_config.get_header()
    header = self.opera_execl.get_cell_value(row, col)
    if header == 'yes':
        return data_config.get_header_value()
    else:
        return None
    
# 下一步判断请求方式
def get_request_method(self, row):
    col = data_config.get_method()
    request_method = self.opera_excel.get_cell_value(row, col)
    return request_method

# 下一步获取url
def get_request_url(self, row):
    col = data_config.get_url()
    url = self.opera_excel.get_cell_value(row, col)
    return url

# 获取请求数据,有一个思考点是数据可能为空
def get_request_data(self, row):
    col = data_config.get_data()
    data = self.opera_excel.get_cell_value(row, col)
    if data == '':
        return None
    else:
        return data
```

因为这个时候获取到的请求数据只是excel里的，还需要获取json文件里的真实请求数据，所以需要导入操作json文件的类

```python
from util.operation_json import OperationJson
```

继续

```python
# 通过获取关键字拿到data数据
def get_data_for_json(self,row):
    self.opera_json = OperationJson()
    request_data = self.opera_json.get_data(self.get_request_data(row))
    return request_data

# 获取预期结果
def get_expect_data(self, row):
    col = data_config.get_expect()
    expect = self.opera_excel.get_cell_value(row, col)
    if expect == '':
        return None
    return expect
```

### 7-8 post、get基类的封装

----

前面我们已经讲了,如何去把Excel里面的这些数据去拿到.回过头来思考一下,现在我们已经有了把Excel的操作进行封装,然后如何(根据请求数据中的关键字)去把它的数据拿出来也弄了,按照正常思想,我们现在是不是要干一件事?因为我们现在是要数据能拿到数据,要操作有操作,这个时候可以直接按照思想的来,拿到数据之后直接去做接口测试,有<u>*URL*</u>,有<u>*是否执行*</u>,有<u>*请求方式*</u>,有<u>*header*</u>,然后直接拿到它的<u>*请求数据*</u>,直接拼接起来,post或get一下就可以了.

我们本堂课程就来讲如何去把这些东西做一个最基础的封装.要封装哪些东西呢,首先我们来思考一下:我们现在有了`get_data.py`中`GetData`下的这些东西(获取是否执行等),那么在我们的base里面,我们是不是应该有个东西,这个东西就像我们之前讲的(发送get/post请求,还有一个`run_main`方法来跑两个),把(`send_post`,`send_get`,`run_main`)这些东西封装起来,只是说当时封装的是一个demo,其实这个demo也是完成了80%以上的.我们这节课就是在它的基础上进行一个完善.

首先在base里面，重新创建一个`runmethod.py`

在这个里面我们要用到`requests`,所以`import requests`,然后需要创建一个类`class RunMethod`,当创建完这个class之后,还是按照之前的思想,我们给它创建三个方法,一个是执行post接口的方法,一个是get的,一个是主的.先把框架搭建起来:

```python
import requests
class RunMethod:
    def post_main(self,url,data,header):
        pass

    def get_main(self,url,data,header):
        pass
  
    def run_main(self,method,url,data,header):  # run_main比前两个方法多了一个method参数,这样的话主函数只要传是get还是post,再加上url,data和header,传进来就可以了
        pass
```

首先来完善第一个方法,先完善参数,url,data,header传进去,我们之前说过,url肯定不为空，data肯定不为空，header有可能为空(所以在函数体内可以对它做一个判断)，所以header传进来首先就是一个`None`,如果说你没有的话就给你弄成一个`None`,.这个时候我们可以按照之前的那个判断header是否为空,然后去执行它的接口:

```python
import requests

class RunMethod:
  
  	def post_main(self,url,data,header=None):
      	requests.post(url=url, data=data, headers=header)
```

这个时候我们可以思考一下我们什么情况下来完成这些: 这时候可以加一个判断:如果说header不等于空,我就知道你现在需要发送的是要有headers的这个参数;不然的话,你就应该发送不带headers的那个参数.这是和之前课程的一个区别:

```python
import requests

class RunMethod:
  
  	def post_main(self,url,data,header=None):
      	if header != None:
          	requests.post(url=url, data=data, headers=header)
        else:
          	requests.post(url=url, data=data)
```

这个时候,首先,要把结果返回出去,那么结果最一开始等于None,如果说有结果,它肯定会有结果,如果程序出错或接口出错就没结果了

```python
import requests

class RunMethod:
  
  	def post_main(self,url,data,header=None):
      	res = None
        if header != None:
          	res = requests.post(url=url, data=data, headers=header)
        else:
          	res = requests.post(url=url, data=data)
        return res
```

这样就可以把结果返回出去了.

但是一般情况下,我们需要把这个结果(指res)弄出来,现在我们的接口,结果它还是没有的,我们测试的接口一般情况下是json的,所以把结果处理一下(指添加`.json()`),现在结果就有了,它是一个以json为格式的结果:

```python
import requests

class RunMethod:
  
  	def post_main(self,url,data,header=None):
    		res = None
    		if header != None:
    				res = requests.post(url=url,data=data,headers=header).json()
    		else:
        		res = requests.post(url=url,data=data).json()
    		return res
```

这时候我还是需要把结果好好处理一下,因为我们的结果看起来可能并不是太友善.但是现在先不处理,因为光说很空洞,只有到后面我们把Excel数据拿来结合之后大家才会知道为什么要处理.

下面的`get_main`其实和上面是一样的,直接把`post_main`的内容copy下来,把post改成get,它的结果就一样了.这里我给大家挖了一个坑,至于这个坑能不能填平,我也不知道,但是我们后面会去做.

```python
import requests

class RunMethod:
  
  	def post_main(self,url,data,header=None):
        res = None
        if header != None:
            res = requests.post(url=url, data=data, headers=header).json()
        else:
          	res = requests.get(url=url, data=data).json()
        return res
      
		def get_main(self, url, data, header=None):
      	res = None
        if header != None:
          	res = requests.get(url=url, data=data, headers=header).json()
        else:
          	res = requests.get(url=url, data=data).json()
        return res
```

下面,`run_main`的参数,同样的,method肯定不能为空,url不能为空,get的时候data是可以为空的,因为它有可能就只有一个URL,所以改写`run_main`中的参数`data=None`,header也可以为空.这时候我就需要去判断了,就判断你这个`method`是get还是post.如果说method等于post,我就去执行`post_main`,我只需要把它的url,data,header传进去就可以了.else之后就是`self.get_main()`,同样的把`url`,`data`,`header`传进去.这个时候我们调用`self.post_main`/`get_main`的时候它会返回一个值,这个时候同样的可以把这个值记录在这里(初始设为None的模式),然后把这个值返回出去(`return res`)

```python
# get_main和post_main 一样
def get_main(self,url,data=None,header=None):   # 但这里要注意,data是后面改的,是从run_main中反应出data可以为空后后补的默认值为空
    res = None
    if header != None:
    	res = requests.get(url=url,data=data,headers=header).json()
    else:
        res = requests.get(url=url,data=data).json()
    return res

def run_main(self,method,url,data=None,header=None):
    res = None
    if method == 'POST':
        res = self.post_main(url,data,header)
    else
        res = self.get_main(url,data,header)
    return res    
```

到目前为止这个方法就完事了,现在剩下的事就是把找一个地方去调用`run_main`,然后把数据传进去,然后就去执行,就是在整个的主流程控制里面去执行.接下来我们就应该是把主流程结合起来了.

### 7-9 主流程封装及错误调试

----

前面讲到了我们把发送接口的方法封装了起来,而且把它放在了一个类里面,然后通过`run_main`主函数去调用.这时候我们只需要在流程里面根据一些条件去调用它就行了. 

我们怎么去执行主流程呢.首先还是要来思考一下主流程需要干哪些事情?现在我们已经有了Excel(Excel里面已经把接口地址、是否执行、请求方式、是否携带header和请求数据等都封装好了),而且数据也已经准备好了(在`/dataconfig/login.json`)

我们这节课来讲主流程入口的封装,首先我们要知道的是:我们已经有了`run_main`这个方法,然后有了json数据,还有Excel,

再创建一个`main`的包,就是一个主要的程序的入口,在包里面创建一个`run_test.py`.首先,

我们要调用base下面的`runmethod.py`,所以要先导入,然后把RunMethod进行实例化

```python

```

我们也需要把拿进去

```python
from base.runmethod
```

剩下的

首先要知道这个case执行多少行,我们需要干的一件事

```python
from base.runmethod import RunMethod
from data.get_data import GetData
class RunTest:
    def __init__(self):
        self.run_method = RunMethod()
        self.data = GetData()
        
    # 程序执行的主入口
    def go_on_run(self):
        res = None
        #10->0,1,2,3,...,9
        rows_count = self.data.get_case_lines()
        for i in range(1,rows_count):
            url = self.data.get_request_url(i)
            method = self.data.get_request_method(i)
            is_run = self.data.get_is_run(i)
            data = self.data.get_data_for_json(i)
            header = self.data.is_header(i)
            if is_run:
                res= self.run_method.run_main(method,url,data,header)
            return res
        
if __name__ == '__main__':
    run = RunTest()
    print(run.go_on_run()
```

运行，如果报引入包错误，提示：`No module named base.runmethod`，说明系统找包的路径是从sys.path 找的，需要把工程目录的路径添加进去

```python
import sys
sys.path.append("E:/www/ImoocInterface")
```



### 7-11 获取接口状态

----

> 如何判断接口状态？

如果只是为了监控线上接口是否正常，看返回的状态码就行了。只需在底层函数验证就行了

在`runmethod`方法下

那么就可以在预期结果下写200，然后如果实际结果也是两百，就可以实现接口状态简单监控



### 7-12 通过预期结果判断case是否执行成功

----

我们需要把预期结果和实际结果进行一个对比,在预期结果栏是一个字符串,实际结果也是一个字符串,那也就是说要判断这个case有没有执行通过我就需要拿预期结果栏里的值和返回的实际结果值进行一个对比.

我肯定不可能知道这个接口的所有数据,或者我也不可能把它都写下来,我是不是可以拿接口返回的值里面的某一个数据作为预期结果,然后以这个数据在实际返回的结果里进行查找,如果查到找就认为这条case是通过的.

我现在需要有一个地方来比较预期结果里的字符串是否在返回的结果里面.可以创建一个方法,这个方法就是比较字符串的->在util包下创建`common_util.py`(通用的方法)

```python
class CommonUtil:
  
  	def is_contain(self,str_one, str_two):
      	'''
      	判断一个字符串是否在另外一个字符串中
      	str_one: 查找的字符串
      	str_two: 被查找的字符串
      	'''
        if str_one in str_two:
            return True
        else:
            return False
```

这样写太low了,改写成:

```python
class CommonUtil:
  
  	def is_contain(self,str_one, str_two):
      	'''
      	判断一个字符串是否在另外一个字符串中
      	str_one: 查找的字符串
      	str_two: 被查找的字符串
      	'''
        flag = None
        if str_one in str_two:
            flag = True
        else:
            flag = False
        return flag
```

然后在主程序里面(`main/run_test.py`)需要把预期结果栏的值拿到,就要用到:

之前操作Excel的方法-> `util/operation_excel.py`里的`get_cell_value()`;

调取某个单元格的内容.就要用到:

`data/get_data.py`里的`get_expect_data()`获取预期结果

> `get_expect_data()`方法的具体内容
>
> ```python
> def get_expect_data(self, row):
>     col = int(data_config.get_expect())
>     expect = self.opera_excel.get_cell_value(row, col)
>     if expect == '':
>         return None
>     return expect
> ```

只需要通过`dataconfig`拿到预期结果是哪一列的,然后就可以拿到expect的值,然后调用.

然后把预期结果值加进`main/run_test.py`里`class Runtest`的`go_on_run()`方法里:

```python
expect = self.data.get_expect_data(i)
```

这样就拿到预期结果值了.

然后在`main/run_test.py`里引入比较两个字符串的方法:

```python
from util.common_util import CommonUtil
```

然后在`RunTest`的构造函数里实例化一下:

```python
class RunTest:
  
  	def __init__(self):
        self.run_method = RunMethod()
        self.data = GetData()
        self.com_util = CommonUtil()
```

然后`go_on_run`就变成了:

```python
def go_on_run(self):
    res = None
    rows_count = self.data.get_case_lines()
    for i in range(1, rows_count):
        url = self.data.get_request_url(i)
        method = self.data.get_request_method(i)
        is_run = self.data.get_is_run(i)
        data = self.data.get_data_for_json(i)
        expect = self.data.get_expect_data(i)
        header = self.data.is_header(i)
        if is_run:
            res = self.run_method.run_main(method, url, data, header)
            if self.com_utl.is_contain(expect, res):
                print('测试通过')
            else:
              	print('测试失败')  
```



### 7-13 将测试结果写入到Excel中

前面我们讲到了我们把实际结果和预期结果进行了对比,来判断它是否通过.通过了之后要告诉Excel应该把实际结果写进去,那如何去写这个实际结果呢?

首先还是需要在操作Excel的类(`util/operation_excel.py`).因为这个Excel现在所有的都是去获取值,没有写入值的,我们应该怎么去写它的 值呢?通过百度查询到具体方法.

在`OperationExcel`类下增加方法:

```python
class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
```

这时候存在一个问题,如果使用`xlwt`这个包去写,以写的方式打开的时候,之前的内容就会被清空,所以还是用`xlrd`,首先还是打开Excel(现在还是read模式):

```python
class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
        read = xlrd.open_workbook(self.filename)
```

拿到Excel以后,把Excel复制一下(这里需要用到第三方包`xlutils`),从最一开始导入复制的方法:

```python
from xlutils.copy import copy

class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
        read_data = xlrd.open_workbook(self.filename)
        write_data = copy(read_data)
```

把原来读出来的Excel数据利用`copy`方法复制一份为`write_data`.现在就有两份同样的数据了.这时候在wirte上操作是不会影响到原有数据的.

我现在拿到的是整个的Excel(指`write_data`),然后通过`index`把第一个sheet的数据拿到:

```python
from xlutils.copy import copy

class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
        read_data = xlrd.open_workbook(self.filename)
        write_data = copy(read_data)
        sheet_data = write_data.get_sheet(0)
```

然后就可以在这个sheet上进行写入操作了:

```python
from xlutils.copy import copy

class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
        read_data = xlrd.open_workbook(self.filename)
        write_data = copy(read_data)
        sheet_data = write_data.get_sheet(0)
        sheet_data.write(row,col,value)
```

这时候写入的数据还在内存里面,写完然后再把这个Excel保存一下:

```python
from xlutils.copy import copy

class OperationExcel:
  	...
    
    # 写入数据
    def write_value(self, row, col, value):
      	'''
      	写入Excel数据
      	row, col, value
      	'''
        read_data = xlrd.open_workbook(self.file_name)
        write_data = copy(read_data)
        sheet_data = write_data.get_sheet(0)
        sheet_data.write(row,col,value)
        write_data.save(self.file_name)
```

然后在`main/run_test.py`的`go_on_run`方法中直接调用就可以了—>替换原有`print(“测试通过”)`部分

刚刚创建的方法需要传row, col, value, 列是在`data_config.py`里的,所以要在`data/get_data.py`下再封装一个写入数据的方法:

```python
def write_result(self,row,value):
    col = int(data_config.get_result())
    self.opera_excel.write_value(row,col,value)
```

然后回到`main/run_test.py`的`go_on_run`方法:

```python
def go_on_run(self):
    res = None
    rows_count = self.data.get_case_lines()
    for i in range(1, rows_count):
        ...
        if is_run:
            res = self.run_method.run_main(method, url, data, header)
            if self.com_utl.is_contain(expect, res):
                self.data.write_result(i, 'pass')   # 这里只需要传行号和结果
            else:
              	self.data.write_result(i, 'fail')
```

再次运行程序,结束以后打开`dataconfig`文件夹下的`case.xls`.证明写入成功

### 7-14 数据依赖问题从设计思路开始

前面的课程已经把接口自动化的大概模型弄出来了,而且我们也能够根据这个case它是否运行把它的case执行掉,然后把它的实际结果写到实际结果栏.按照这样的思路,我们整个的case、录入的接口是能够都运行完成的.按照正常道理来说这样思考是不是没有问题了.

给大家举一个简单的例子,打开慕课网,下了一个订单,然后点「立即支付」(然后跳转到订单详情页面,包括支付方式、订单金额和优惠券抵扣,页面最下有「立即支付」字样),来看一下订单支付的这个接口,点击「立即支付」调用的是`/api/pay/pay`这个接口,这个接口需要传递的参数有:

```python
{
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"trade_number": "1711012046395438",
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"pay_way": "1"
}
```

 它有一个订单号(`trade_number`),剩下的就是一些加密的东西(`secrect`, `token`, `pay_way`等).我们可以思考下,在购物流程里面,我这个订单肯定是要依赖于我下这个单,我要买这个课程(点「立即购买」),然后在跳转的页面上「提交订单」,提交完订单(调用的是`/api/pay/submitorder`接口,会传一个`goods_ids`)之后,它会给我生成一个订单号(`out_trade_no`),这个订单号要用于支付(`/api/pay/pay`)的接口.

也就是说我要测试`/api/pay/pay`这个接口,必须先执行`/api/pay/submitorder`拿到它的订单号`out_trade_no`,拿到订单号之后把`/api/pay/pay`的`trade_number`字段填成`out_trade_no`的数据,然后再执行测试,它才能够返回数据.如果随便填一个`trade_number`,就会报错(比如“没有这个订单”之类的),这样就没办法验证数据了.如果只是想像之前那样验证状态就没问题.像这种情况我们应该怎样去解决呢?

打开Excel,新增三列:case依赖,依赖的返回数据,数据依赖字段

**case依赖:**假如说要执行Imooc-12(以`/api/pay/pay`为例), 要先执行Imooc-11,也就是说Imooc-12依赖于Imooc-11,所以就在Imooc-12点case依赖列里填入第11个case的case_id:Imooc-11

**依赖的返回数据**:执行了Imooc-11之后,要拿回`out_trade_no`这个字段的值,所以在**依赖的返回数据**栏填入该字段的名称

**数据依赖字段**:拿到之后用于mooc-12(`/api/pay/pay`)这个接口的`trade_number`字段,<u>*依赖的返回数据*</u>的字段名和*<u>数据依赖字段</u>*的字段名称有可能是不一样的,这里慕课网就是不一样的

先拿依赖的`case_id`,依赖返回数据字段是哪一个(就填入<u>*依赖的返回数据*</u>栏),请求数据字段是哪一个(就填入<u>*数据依赖字段*</u>栏),依次填写.

再操作一个流程:

​	  按照之前的设计,我们只需要找到它上一个接口返回的字段就可以了,当有多个重名的接口字段时,<u>*依赖的返回数据*</u>(上一个例子是`out_trade_no`)就需要改写为`data:[1]:name`.就是定义一个规则,如果你要这样去取数据,你是哪一层的,第几个值,如果没有这个的话就默认获取`data:name`.这样我们就不用考虑依赖字段是怎样解决的

现在如果说有哪个case它依赖于哪一条数据,我们只需要把它依赖的case拿到,然后把<u>*依赖的返回数据*</u>拿回来, 然后更新到待测接口的<u>*数据依赖字段*</u>就可以了.

再次把请求数据以及整个的URL拼接数据,然后就可以了:

以上面的`/api/pay/submitorder`和`/api/pay/pay`接口为例:

正常的两个接口的返回/请求数据如下:

```python
# submitorder 接口的返回数据
response_data = {
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"goods_ids": "357",
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"coupon_id": "0"
}

# pay 接口的请求数据
request_data = {
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"trade_number": "1711012046395438",
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"pay_way": "1"
}
```

这时候request_data就需要变为:

```python
request_data = {
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"trade_number": num,
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"pay_way": "1"
}
```

`trade_number`字段就变为`num`,为一个变量,指的是`response_data`接口的某一个值,也就是在`data_config`里

```python
# submitorder 接口的返回数据
response_data = {
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"goods_ids": "357",
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"coupon_id": "0"
}

num = response_data["goods_ids"]

# pay 接口的请求数据
request_data = {
  	"timestamp" : "1509540501557",
  	"uid": "5249191",
  	"trade_number": "1711012046395438",
  	"secret": "32c11c011bd5db6635b79153d96cb55d",
  	"from": "2",
  	"token": "921e73771e5b2295b42872f14f0b1fc3",
  	"pay_way": "1"
}
```

这样的话请求数据(指`request_data`)就和之前请求数据列里的数据格式一样了.

### 7-15 数据依赖问题方法封装之通过case_id获取case数据





## 第9章 获取cookie及请求处理

### 9-3 如何拿到cookie去写入文件



3. 50:04
4. 43:27
5. 35:18

















