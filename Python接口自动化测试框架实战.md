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

在base里面，重新创建一个`runmethod.py`

```python
import requests
class RunMethod:
    def post_main(self,url,data,header):
        pass
    def get_main(self,url,data,header):
        pass
    
    def run_main(self,method,url,data,header):
        pass
```

首先来完善第一个方法，先完善参数，url肯定不为空，data肯定不为空，header可能为空，所以header设为默认参数None

```python
def post_main(self,url,data,header=None):
    res = None
    if header != None:
    	res = requests.post(url=url,data=data,headers=header).json()
    else:
        res = requests.post(url=url,data=data).json()
    return res

# get_main和post_main 一样
def get_main(self,url,data=None,header=None):
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

### 7-9 主流程封装及错误调试

----

再创建一个`main`的包

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

首先还是需要在操作Excel的类(`util/operation_excel.py`).现在Excel所有的都是去获取值,没有写入值的,通过百度查询到具体方法.

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

拿到Excel以后,把Excel复制一下.(这里需要用到第三方包`xlutils`),从最最一开始倒入复制的方法:

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

然后通过index把第一个sheet的数据拿到:

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

这时候写入的数据还在内存里面.然后再把这个Excel保存一下:

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
                self.data.write_result(i, 'pass')
            else:
              	self.data.write_result(i, 'fail')
```

再次运行程序,结束以后打开`dataconfig`文件夹下的`case.xls`.证明写入成功





## 第9章 获取cookie及请求处理

### 9-3 如何拿到cookie去写入文件















