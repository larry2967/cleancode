# 裝飾器基本概念
<IMG src="https://static.coderbridge.com/img/techbridge/images/kdchang/python-tips/decorator_tutorial_code.png" alt=" 如何使用 Python 學習機器學習（Machine Learning）"/>

# Function Decorator
```python
def my_decorator(func):
    print('裝飾器加點料')
    return func

def my_func():
    print('被裝飾函式執行')

my_func = my_decorator(my_func)

my_func()
#裝飾器加點料
#被裝飾函式執行
```
加上裝飾器之後...
```python
def my_decorator(func):
    print('裝飾器加點料')
    return func

@my_decorator
def my_func():
    print('被裝飾函式執行')

my_func()
#裝飾器加點料
#被裝飾函式執行

```


前情提要：很多初學者會混淆這 func 與 func() 兩種賦值：

```python
def func():
    return "hello,world"
    
ref1 = func        # 將函數對象賦值給ref1
ref2 = func()      # 調用函數，將函數的返回值("hello,world"字符串)賦值給ref2

type(ref1)
>>> function       # ref1引用了函數對象本身
type(ref2)
>>> str            # ref2則引用了函數的返回值

# 進一步驗證ref1是可調用的，而ref2是不可調用的
callable(ref1)
>>> True
callable(ref2)
>>> False
```
---

Decorator 程式語言的設計模式，也是一種特殊的 function（**`把被裝飾函式當做參數傳入裝飾器函式，再把被裝飾函式傳回`**）

1. 可重複利用!!!!!
2. 擴充簡單!!!!!

:question:問題：retries_limit 的次數好像無法調整 :broken_heart:



## Passing arguments to decorators - 傳遞引數給裝飾器



> *The first one is to create decorators as nested functions with a new level of indirection, making everything in the decorator fall one level deeper. The second approach is to use a class for the decorator.*

* **As a nested functions**： 將裝飾器作成含有新的間接層的嵌套型函式(將裝飾器裡面的東西往下移動一層)
    * 基本：不含參數的 function decorator(decorator 裡面再定義一個函式 wrapper + Closure)
    * 進階：含參數的 function decorator
    
* **Use a class for decoratro**： 透過類別來定義裝飾器(__call__)

> *the second approach favors readability more*

#### 範例說明
* Clean Code 5 - Function-based Decorator

```python
from functools import wraps

def decorate(f):
    @wraps(f)
    def wrapper(*args,**kwargs):
        print(f'do something before calling function {f.__name__}')
        f(*args,**kwargs)
        print(f'do something after calling function {f.__name__}')
    return wrapper

@decorate
def myFunc():
    print('主程式')

if __name__ == '__main__':
    myFunc()

# 輸出結果    
# do something before calling function myFunc
# 主程式
# do something after calling function myFunc
```

範例：顯庭的裝飾器

![圖片1.jpg](.attachments/圖片1-4c3b8240-405e-4a84-a73f-4f82e425fa6b.jpg)

```python
def decorator_tim(func):
    def wrapper(argument):
        start = datetime.datetime.now()
        print("Start execute {0}, start: {1}"
              .format(func.__name__, start.strftime("%Y/%m/%d %H:%M:%S")))
        func(argument)
        end = datetime.datetime.now()
        print("End execute {0}, end: {1}, cost: {2} s"
              .format(func.__name__, end.strftime("%Y/%m/%d %H:%M:%S"), (end - start).total_seconds()))
    return wrapper

@decorator_tim
def sleep_time(wait_time):
    print('Wait {}s'.format(wait_time))
    time.sleep(wait_time)

>>> sleep_time(10)
Start execute sleep_time, start: 2020/03/29 13:44:54
Wait 10s
End execute sleep_time, end: 2020/03/29 13:45:04, cost: 10.006737 s

# 若沒有加參數則會…
>>> sleep_time(10)
TypeError: wrapper() takes 0 positional arguments but 1 was given

```
> *由此可以發現，當python直譯器執行到 @decorator_tim 時，就開始進行裝飾了，這就是相當執行了 `sleep_time = decorator_tim(sleep_time)`*

&radic; 閉包大魔王(Closure)延伸閱讀：LEGB、作用域、capture variable。

### 說明
* 1.首先，開看我們的裝飾器函式decorator_tim，該函式接收一個引數func，其實就是接收一個方法名
* 2.decorator_tim內部又定義一個函式wrapper，在wrapper函式中增加時間記錄點，並在記錄完後呼叫傳進來的引數func，同時decorator_tim的返回值為內部函式wrapper，其實就是一個閉包函式。
* 3.然後，再來看一下，在上增加 @decorator_tim，那這是什麼意思呢？
當python直譯器執行到這句話的時候，會去呼叫decorator_tim函式，同時將被裝飾的函式名作為引數傳入(此時為sleep_time)，根據閉包一文分析，在執行decorator_tim函式的時候，此時直接把wrapper函式返回了，同時把它賦值給sleep_time，此時的sleep_time已經不是未加裝飾時的sleep_time了，而是指向了decorator_tim.wrapper函式地址。

* 4.接下來，在呼叫sleep_time()的時候，其實呼叫的是decorator_tim.wrapper函式，那麼此時就會先執行時間記錄，然後再呼叫原來的sleep_time()，該處的sleep_time就是通過裝飾傳進來的引數sleep_time。這樣下來，就完成了對sleep_time的裝飾，實現了時間記錄
---

* Clean Code 5 - 含參數的Function-based Decorator

  再多包一層function來傳遞參數

```python
from functools import wraps

def decorate(para1,para2):
    def outer_wrapper(f):
        @wraps(f)
        def wrapper(*args,**kwargs):
            print(f'do something before calling function {f.__name__} with {para1}')
            f(*args,**kwargs)
            print(f'do something after calling function {f.__name__} with {para2}')
        return wrapper
    return outer_wrapper

@decorate('傳入參數1','傳入參數2')
def myFunc():
    print('主程式')

if __name__ == '__main__':
    myFunc()
    
# 輸出結果
# do something before call function myFunc with 傳入參數1
# 主程式
# do something after call function myFunc with 傳入參數2
```

## Decorators with nested functions
書中案例無法在裝飾器指定重試的次數，只能在裝飾器定義好固定次數，於是嵌入另一層函式。

* 需要一個間接層 level of indirection
  * 1.用第一層函數接收參數。
  * 2.接著定義一個新函數，他就是裝飾器。
  * 3.接著再定義一個新函數，就是這個裝飾程序回傳的結果

![nested.jpg](.attachments/nested-39f06d64-5028-4ee5-878f-e8b81ea65d46.jpg)


```python
# decorator_parametrized_1.py
@with_retry()
def run_operation(task):
    return task.run()

@with_retry(retries_limit=5)
def run_with_custom_retries_limit(task):
    return task.run()

@with_retry(allowed_exceptions=(AttributeError,))
def run_with_custom_exceptions(task):
    return task.run()

@with_retry(retries_limit=4, allowed_exceptions=(ZeroDivisionError, AttributeError))

def run_with_custom_parameters(task):
    return task.run()
```
  
## Decorator objects
> *裝飾器函式其實是一個介面約束，它必須接受一個 callable 物件作為引數，然後返回一個 callable 物件。
在python中，一般 callable 物件都是函式，但是也有例外。比如只要某個物件重寫了 call 方法，那麼這個物件就是 callable 的。*

```python
class Test(object):
    def __init__(self, func):
        print('test init')
        print('func name is %s ' % func.__name__)
        self.__func = func
        
    def __call__(self, *args, **kwargs):
        print('裝飾器中的功能')
        self.__func()

@Test
def test():
    print('this is test func')
    
test()

>>> test init
>>> func name is test 
>>> 裝飾器中的功能
>>> this is test func
```
> *和之前的原理一樣，當 python 直譯器執行到到 @Test 時，會把當前 test 函式作為引數傳入 Test 物件，呼叫 init 方法，同時將 test 函式指向建立的 Test 物件，那麼在接下來執行 test() 的時候，其實就是直接對建立的物件進行呼叫，執行其 call 方法。*

## Good uses for decorators
* 可以用於日誌記錄、性能測試、事務處理、緩存、權限驗證等場景，目前是抽離出大量與函數功能本身無關的雷同代碼到裝飾器繼續重用。
也是為已存在的函數添加額外的功能。

    * Transforming parameters
    * transformed underneath
    * Tracing code: Logging the execution of a function with its parameters
    * Validate parameters
    * Implement retry operations
    * Simplify classes by moving some (repetitive) logic into decorators


--- 
## Effective decorators – avoiding common mistakes
### Preserving data about the original wrapped object

> *One of the most common problems when applying a decorator to a function is that some of the properties or attributes of the original function are not maintained, leading to undesired, and hard-to-track, side-effects.*

* 使用裝飾器可能沒有保存好原始(hello)函式的特性或屬性而造成副作用，常見的就是修改掉函式的名稱與 docstring。

```python
def decorator(func):
    def wrapper():
      print("%s was called" % func.__name__)
      func()
      print("bye~")
    return wrapper
    
@decorator
def hello(name="Tim"):
    print("Hello %s!" % name)
hello()

>>> hello was called
>>> Hello Tim!
>>> bye~

# 檢查方式
print(hello.__name__)
>>> wrapper

# 檢查方式
print(help(hello))
>>> Help on function wrapped in module __main__  # 若置於其他 decorator.py 則會 show
>>> wrapper

# 檢查方式
hello.__qualname__
>>> 'decorator.'<locals>.wrapper
```

* 搭配使用 __name__, help, __qualname__ 來檢查
* 過往裝飾器會將原始(被裝飾)函數變成新的(稱 wrpper )，我們看到的是 wrapper 的特性、docstring，不是原始函式的。
  * 問題：whick is a major concern，會讓你在記錄或追蹤函式時，難以除錯。
  
### 如何修正
這時候 functions.wraps 就可以登場了，加了一段 @wraps(func)，上碼！

> *functions.wraps 目的在消除裝飾器對原函數造成的影響，即原函數的相關屬性進行拷貝，達到裝飾器不修改原函數為目的。*

```python
from functools import wraps
def decorator(func):
    @wraps(func)
    def wrapper():
      print("%s was called" % func.__name__)
      func()
      print("bye~")
    return wrapper
    
@decorator
def hello(name="Tim"):
    print("Hello %s!" % name)
    
hello()
>>> hello was called
>>> Hello Tim!
>>> bye~

print(hello.__name__)
hello
```

## Dealing with side-effects in decorators
> *Decorating should be placed in the innermost function definition, or there will be problems when it comes to importing.
除了裝飾器裝飾的函式外，裝飾器要執行的任何工作應該都放在最裡面的函式定義。*

你可以這樣修改:

```python
def decorator_tim(func):
    start = datetime.datetime.now()
    def wrapper(argument):
        print("Start execute {0}, start: {1}"
              .format(func.__name__, start.strftime("%Y/%m/%d %H:%M:%S")))
        func(argument)
        end = datetime.datetime.now()
        print("End execute {0}, end: {1}, cost: {2} s"
              .format(func.__name__, end.strftime("%Y/%m/%d %H:%M:%S"), (end - start).total_seconds()))
    return wrapper

@decorator_tim
def sleep_time(wait_time):
    print('Wait {}s'.format(wait_time))
    time.sleep(wait_time)

>>> sleep_time(1)
Start execute sleep_time, start: 2020/05/20 13:00:00
Wait 1s
End execute sleep_time, end: 2020/05/20 13:00:01, cost: 1.006737 s

>>> sleep_time(1)
Start execute sleep_time, start: 2020/05/20 13:00:00
Wait 1s
End execute sleep_time, end: 2020/05/20 13:01:01, cost: 61.002737 s
```


# Class Decorator

好處:
a. 符合DRY原則：將密切關聯的程式放在一起
b. 可建立較簡單且較小的class，方便維護

```python
class CustomLogger(object):
    def __init__(self, in_prefix, out_prefix):
        self.in_prefix = in_prefix
        self.out_prefix = out_prefix
    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print(self.in_prefix)
            func(*args, **kwargs)
            print(self.out_prefix)
        return wrapper

@CustomLogger('->', '<-')
    def g(a, b):
        print('calling g with args ', (a, b,))
g(2, 4)

#->
#calling g with args  (2, 4)
#<-
```

# 讓裝飾子有副作用
```python
REG_EVENT = {}

def test(event_cls):
    REG_EVENT[event_cls.__name__] = event_cls
    return event_cls

class Event:
    """test"""
    
class UserEvent:
    TYPE = 'user'

@test
class UserLoginEvent(UserEvent):
    """some thing"""
    
@test
class UserLogoutEvent(UserEvent):
    """some thing"""
```
在 import 時，全部的裝飾子已經執行，產生註冊字典
```python
from xxxx import REG_EVENT

REG_EVENT
#{'UserLoginEvent': __main__.UserLoginEvent,
# 'UserLogoutEvent': __main__.UserLogoutEvent}
```

#一定有效的裝飾子
```python
from functools import wraps

class DBdriver:
    def __init__(self, db_string):
        self.db_string = db_string
    def execute(self, query):
        return f"query '{query}' at '{self.db_string}'"

def inject_db_driver(func):
    @wraps(func)
    def wrapped(dbstring):
        return func(DBdriver(dbstring))
    return wrapped

@inject_db_driver
def run_query(driver):
    return driver.execute('select 1'')

run_query('esb12267@10.240.205.111:5432')
#"query 'select 1' at 'esb12267@10.240.205.111:5432'"
```
* 但是在 class 內第一個引數為 self，進入裝飾子會發生引數個數不合 
```python
class Test:
    @inject_db_driver
    def run_query(self, driver):
        return driver.execute('select 1'')

t = Test()
t.run_query('esb12267@10.240.205.111:5432')

#TypeError: wrapped() takes 1 positional argument but 2 were given
```

* 使用協定描述器 ( protocol descriptor )

```python
from functools import wraps
from types import MethodType

class inject_db_driver:
    """
    """
    def __init__(self, func):
        self.func = func
        wraps(self.func)(self)
    def __call__(self, db_string):
        print('call')
        return self.func(DBdriver(db_string))
    def __get__(self, instance, owner):
        print('get')
        print(instance)
        print(owner)
        if instance is None:
            return self
        return self.__class__(MethodType(self.func, instance))
```

使用在函數上

```python
@inject_db_driver
def run_query(driver):
    return driver.execute('select 1')

run_query('esb12267@10.240.205.111:5432')
# call
# "query 'select 1' at 'esb12267@10.240.205.111:5432'"
```

使用在類別內
```python
class Test:
    @inject_db_driver
    def run_query(self, driver):
        return driver.execute('select 1')
    
t = Test()
t.run_query('esb12267@10.240.205.111:5432')
# get
# <__main__.Test object at 0x7f30236d8208>
# <class '__main__.Test'>
# call
# "query 'select 1' at 'esb12267@10.240.205.111:5432'"
```

# 補充
```
class Test: pass

def test_func(self):
    print('test')

t = Test()
t.test_func = MethodType(test_func, t)

t.test_func()
# test
```

#何時使用裝飾子
* 不要一開始就建立裝飾子
* 重複開始出現，模式開始變清晰
* 確定至少被使用三次
* 內部程式越少越好

# 好的裝飾子
* 封裝、單一功能：不需要知道內部邏輯
* 正交性：獨立不耦合
* 重用性：可套用至多種型態

#