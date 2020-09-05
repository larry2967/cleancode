<font color= black size=3 face="微軟正黑體" cellpadding=1000000  style="line-height: 1.5">

# **Pythonic: 符合Python風格的程式**
- 程式表現較佳
- 精簡且易於理解
- 若團隊使用相同程式結構與模式，可專注問題、避免犯錯

# **Indexes and slices**
- 使用slice取得部分元素、指定開始/結束位置/間隔
```python
>>> my_numbers = (1, 1, 2, 3, 5, 8, 13, 21)
>>> my_numbers[2:5]        # 呼叫了tuple裡面的__getitem__
(2, 3, 5)
```

- 當myobject[key]的東西被呼叫時，__getitem__被呼叫，並傳入參數key
- sequence (e.g., list, tuple, string) 都有__getitem__和__len__的物件，所以才能做迭代

自己要怎麼去實現magic method呢?
```python
class Items:
    def __init__(self, *values):
        self._values = list(values)

    def __len__(self):
        return len(self._values)        

    def __getitem__(self, item):
        return self._values.__getitem__(item)

>>> ll = Items(1, 2, 3)
>>> print(len(ll)) # __len__
3

>>> print([s for s in ll]) # __getitem__
[1, 2, 3]
```
如果你就是想自創 (i.e., 不想用__getitem__, __len__記得遵守下列規範：

- 索引結果的型態要跟instance一樣型態，如slice a list，結果也是list
- 不含最後一個元素

# **Context managers(環境管理器)**
- 常見於資源管理，如開啟檔案或服務連線時，確保檔案與連線會自動關閉
- 由__enter__, __exit__兩神奇函數組成

例如：
```python
# 確保檔案最後有被關閉
with open(filename) as f: # __enter__
    """first code"""
    """last code"""      # __exit__
```



範例：DB停止 → 備份 → DB運作

```python
def stop_database():
    print("systemctl stop postgresql.service")


def start_database():
    print("systemctl start postgresql.service")


class DBHandler:
    def __enter__(self):
        stop_database()
        return self

    def __exit__(self, exc_type, ex_value, ex_traceback):
        start_database()


def db_backup():
    print("pg_dump database")


>>> with DBHandler():
...     db_backup()
systemctl stop postgresql.service # __enter__
pg_dump database
systemctl start postgresql.service # __exit__

```
利用__enter__回傳某些東西
```python
def stop_database():
    print("systemctl stop postgresql.service")

def start_database():
    print("systemctl start postgresql.service")

class DBHandler:
    def __enter__(self):
        stop_database()
        return 1 # 媽我在這!!!

    def __exit__(self, exc_type, ex_value, ex_traceback):
        start_database()

def db_backup():
    print("pg_dump database")

>>> with DBHandler() as db_someting: # as 接回__enter__ return 回來的東西
...     print(df_someting)
...     db_backup()
systemctl stop postgresql.service # __enter__
1                                 # __enter__ return 1
pg_dump database
systemctl start postgresql.service # __exit__
```

__exit__回傳True
```python
def stop_database():
    print("systemctl stop postgresql.service")

def start_database():
    print("systemctl start postgresql.service")

class DBHandler:
    def __enter__(self):
        stop_database()

    def __exit__(self, exc_type, ex_value, ex_traceback):
        start_database()
        return True

def db_backup():
    print("pg_dump database")

>>> with DBHandler():
...     print(ahuhiuhdiuwwdcniownio) # 這個變數沒有被assign，應該會有error
...     db_backup()
systemctl stop postgresql.service  # __enter__
systemctl start postgresql.service # __exit__ 強制return True所以沒有error被回傳
                                   # 安ㄋㄟ甘對

# 沒有強制return True的話輸出會如下
# systemctl stop postgresql.service
# systemctl start postgresql.service
# NameError: name 'ahuhiuhdiuwwdcniownio' is not defined
```

><font color=red>**不要讓__exit__回傳True，除非你有一個特別的理由**</font>

## Implementing context managers
共有四種方式：
1. 定義好class的__enter__ 與__exit__ 兩個方法函數。
2. contextlib.contextmanager
將此裝飾器套用在函式上時，會將此函式轉為環境管理器。
此函式必須是**產生器**(generator)，也就是函數裡會有一個<font color=#FF8282>yield</font>，
利用<font color=#FF8282>yield</font>拆成兩個部分，分別是__enter__及__exit__。

```python
import contextlib
def stop_database():
    print("systemctl stop postgresql.service")

def start_database():
    print("systemctl start postgresql.service")

def db_backup():
    print("pg_dump database")

@contextlib.contextmanager
def db_handler():
    stop_database()  # __enter__
    yield
    start_database() # __exit__

>>> with db_handler():
...     db_backup()
systemctl stop postgresql.service
pg_dump database
systemctl start postgresql.service
```
3. contextlib.ContextDecorator
將裝飾器套用至函式，讓它可以在環境管理器中執行。
```python
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()

    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()

@dbhandler_decorator()
def offline_backup():
    print("pg_dump database")

>>> offline_backup()
systemctl stop postgresql.service
pg_dump database
systemctl start postgresql.service
```
><font color=red>**但是這個方法無法在環境管理器中取得你想要使用的東西**</font>

4. contextlib.supress (similar to try/except)
當收到特定例外時，不會失敗。
```python
import contextlib
class NonFatalError(Exception):
    pass

def non_idempotent_operation():
    raise NonFatalError('The operation failed because of existing state')

def suppress_example(): 
    with contextlib.suppress(NonFatalError):
        print('trying non-idempotent operation')
        non_idempotent_operation()
        print('succeeded!')
    print('done')

>>> suppress_example()
trying non-idempotent operation
done
```
所以換句話說
如果是非特定例外9會失敗啦~
```python
import contextlib
class NonFatalError(Exception):
    pass

def non_idempotent_operation():
    raise NonFatalError(
        'The operation failed because of existing state'
    )

def suppress_example(): 
    with contextlib.suppress(TypeError):
        print('trying non-idempotent operation')
        non_idempotent_operation()
        print('succeeded!')
    print('done')

>>> suppress_example()
trying non-idempotent operation
NonFatalError: The operation failed because of existing state
```

參考資料：[https://blog.gtwang.org/programming/python-with-context-manager-tutorial/](Python 的 with 語法使用教學：Context Manager 資源管理器)

# **Properties, Attributes, and different types of methods for objects**

## Python裡的__底線
- 提昇程式碼的品質
- 強化程式碼邏輯
- 避免寫出危險的程式碼


> ![underscore_3.jpg](.attachments/underscore_3-f9beec54-a5fb-4e69-9027-affcf4e6f8c1.jpg)
![underscore_3_1.jpg](.attachments/underscore_3_1-2fc9686e-95ad-4611-baf1-0dee70d2ee3c.jpg)
![underscore_4.jpg](.attachments/underscore_4-8bffedca-e28c-4263-8538-d4d6d71631a8.jpg)
![underscore_5.jpg](.attachments/underscore_5-9823df53-593f-4702-94d2-abe153ed3db4.jpg)
![underscore_6.jpg](.attachments/underscore_6-aa0d2bc6-d07e-4120-89d4-7caf95de82f5.jpg)

><font color=red>**避免使用雙底線**</font>

參考資料：[https://aji.tw/python%E4%BD%A0%E5%88%B0%E5%BA%95%E6%98%AF%E5%9C%A8__%E5%BA%95%E7%B7%9A__%E4%BB%80%E9%BA%BC%E5%95%A6/](Python, 你到底在底線__什麼啦！)




### Properties
1.   將 class (類別) 的 method (方法) 轉換為只能讀取的 attribute (屬性)
2.   可以類別中把一個方法變成屬性使用，並做到檢查屬性的值是否符合我們的預期

```PYTHON
class MyBank():
  def __init__(self, name):
    self.name = name
    self._password = None
    
  def password(self):
    return self._password
```

```
>>> acct_1 = MyBank('kevin')
>>> acct_1.password = '123'
>>> print(acct_1.password)
123

>>> acct_1.password = '456'
>>> print(acct_1.password)
456
```
#### 加入 @property
```PYTHON
class MyBank2():
  def __init__(self, name):
    self.name = name
    self._password = None
    
  @property
  def password(self):
    return self._password
```

```
>>> acct_2 = MyBank2('kevin')
>>> acct_2.password = '123'
>>> print(acct_2.password)
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-194-ec84ff4e295a> in <module>()
----> 1 acct_2.password = '123'
      2 print(acct_2.password)

AttributeError: can't set attribute
```

><font color="#dd0000">加入 @property 之後，password 從方法變成只可讀取的屬性，因此當我們要將 password 賦值時，就出現錯誤

#### 加入 @setter
```PYTHON
import re

class MyBank3():
  def __init__(self, name):
    self.name = name
    self._password = None

  @property
  def password(self):
    return self._password

  @password.setter
  def password(self, new_password):
    if bool(re.search('@', new_password)) == False:
      raise ValueError('密碼要包含特殊字元')
    else:
      self._password = new_password 
      print('密碼設定成功為: ', self._password)
```
><font color="#008000">不需要修改函式名字 ( 此處為 password ) ，在函式上方加入 decorator 即可

```
>>> acct_3.password = 'abcde'
>>> print(acct_3.password)
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-198-648f85b7260d> in <module>()
----> 1 acct_3.password = 'abcde'
      2 print(acct_3.password)

<ipython-input-196-4a8a51d2f969> in password(self, new_password)
     11   def password(self, new_password):
     12     if bool(re.search('@', new_password)) == False:
---> 13       raise ValueError('密碼要包含特殊字元')
     14     else:
     15       self._password = new_password

ValueError: 密碼要包含特殊字元
```

```
>>> acct_3.password = 'abcde@esun'
>>> print(acct_3.password)
密碼設定成功為:  abcde@esun
abcde@esun
```
><font color="#008000">符合自定義的寫入規範後，就能成功賦值


---
## Iterable Objects
### 基本觀念

![相關名詞的關係](.attachments/relationships-cd1f8489-5f7c-4b6f-bc92-a83f186b7d4a.png 
 =80%x)

1. **Iteration**：走訪/迭代/遍歷一個 object 裡面被要求的所有元素之「過程」或「機制」。是一個概念性的詞。
2. **Iterable**：可執行 Iteration 的 objects 都稱為 Iterable ( 可當專有名詞 ) 。參照官方文件，是指可以被 for loop 遍歷的 objects。具備以下程式碼的 objects 就是 Iterable。
    - __ iter __    或
    - __ getitem __
3. **Iterator**：一個物件，用來對 iterable 執行 interation，必須可以產生下一個值。遵照 Python Iterator Protocol 的 objects ; 以 Python3 而言，參照官方文件，具備以下程式碼的 objects 皆為 Iterator
    - __ iter __ 且
    - __ next __
    

![forloop.png](.attachments/forloop-5a637ea2-37dc-4f19-a8fc-160b21e1ef19.png =80%x)

4. **關於 yield**
    - yield 只能出現在 function 裡，並使得這個 function 變成 generator object，自帶 __ iter __ 和 __ next __ 這2個 attributes。在 Python 中，一邊迴圈一邊計算的機制，稱為生成器
    - generator 記住上一次返回時在函式體中的位置。對生成器函式的第二次（或第 n 次）呼叫跳轉至該函式中間，而上次呼叫的所有區域性變數都保持不變。
    - 原先和 return 搭配的 function，就是吃進 input 然後輸出 output，life cycle 就會消失，yield 的寫法可以視為擴充 function 的特性，使其具有「記憶性」、「延續性」的特質，可以不斷傳入 input 和輸出 output，且不殺死 function。未來要撰寫具有該特質的 function 時就可以考慮使用 yield 來取代「在外部存一堆 buffer 變數」的做法。


### Creating iterable objects
```PYTHON
class MyIterator():
  def __init__(self, max_num):
    self.max_num = max_num
    self.index = 0
  
  def __iter__(self): # __iter__ 僅僅是回傳 self ，原因是該 object 已經有 __next__ 故本身就是一個 Iterator 
    return self

  def __next__(self): # 定義 next 方法
    self.index += 1
    if self.index < self.max_num:      
      return self.index # 以 self.index 來作為要 iterate 出去的值
    else: 
      raise StopIteration
```

```
>>> myiter = MyIterator(4)

# 遍歷: 逐步移動指針
>>> print(next(myiter))
1
>>> print(next(myiter))
2
>>> print(next(myiter))
3
>>> print(next(myiter))
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-50-60bcaebd4b5a> in <module>()
      2 print(next(myiter))
      3 print(next(myiter))
----> 4 print(next(myiter))
      5 print(next(myiter))

<ipython-input-48-b6da58d376f0> in __next__(self)
     12       return self.index # 以 self.index 來作為要 iterate 出去的值
     13     else:
---> 14       raise StopIteration

StopIteration: 
```

```
>>> myiter = MyIterator(4)

# 第一次使用此 iterable object
>>> for i in myiter:
>>>   print(i)
1
2
3

# 第二次使用此 iterable object
>>> "-".join(map(str, myiter))
''

# 第三次使用此 iterable object
>>> max(myiter)
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-54-fb47f4d6fd36> in <module>()
----> 1 max(myiter)

ValueError: max() arg is an empty sequence
```

><font color="#dd0000">這個名為 myiter 的 iterable object 只被創建出1個/次，每次使用 next() 或 __ next __ 進行遍歷時，皆會移動指針，當指針移到最後一個元素時，此 iterable object 便無法再被使用


```PYTHON
# 使用 yield、刪除 __next__()

class MyIterator2():
  def __init__(self, max_num):
    self.max_num = max_num
  
  def __iter__(self): 
    num = 1
    while num < self.max_num:
      yield num   # 把 num 放進 generator
      num += 1

```

><font color="#008000"> - 當一個 function ( 此處為 __ iter __ ) 中帶有 yield 時，該 function 就會自動變成 gerenator，而自帶 __ next __ ; 這是一個 Iterator。
><font color="#008000"> - 因為 yield 產生 __ next __ 的效果，故只寫 __ iter __ 即可

```
>>> myiter_2 = MyIterator2(4)
# 第一次使用此 iterable object

>>> for i in myiter_2:
>>>   print(i)
1
2
3

# 第二次使用此 iterable object
>>> "-".join(map(str, myiter_2))
'1-2-3'

# 第三次使用此 iterable object
>>> max(myiter_2)
3
```

><font color="#008000">需要做遍歷時 ( 會呼叫 __ iter __ ) ，每一次都會產生一個 generator，把按照你邏輯一個一個計算出的 element 存放於此，然後再做遍歷的動作 ( 如: 取 max, min, 做 join...etc)


### Creating Sequences
- sequence 一次保存了所有東西，不像 iter 系列一個一個訪問
- __ getitem __

```PYTHON
# 自建 list

class MyList():
  def __init__(self, stt_pt, end_pt):
    self.stt_pt = stt_pt
    self.end_pt = end_pt
    self._pool = self._create_range() # 實體屬性是由後面的某個實體方法來賦值、定義

  def _create_range(self): # 給頭尾之後，自動產生有順序的值
    now_pt = self.stt_pt
    pool = []
    while now_pt < self.end_pt:
      pool.append(now_pt)
      now_pt += 1
    return pool

  def __getitem__(self, pool_no):
    return self._pool[pool_no]

  def __len__(self):
    return len(self._pool)
```

```
>>> my_sequence = MyList(1, 5)

>>> for i in my_sequence:
>>>   print(i)
1
2
3
4

>>> my_sequence[2]
3

>>> len(my_sequence)
4
```
:
><font color="#008000">為何 Iteration 要設計 __ next __ 和__ getitem __ 這兩種 pattern 呢？ 實作面來說，
>><font color="#008000">- 用 __ next __ 這種方式寫會更適合用在生成前後值相依或是與 streaming 概念相似的資料，例如「生成 Fibonacci Sequence。此方法較耗費 CPU usage。
>><font color="#008000">- 而 __ getitem __ 一般使用情境是已經將所有資料儲存在某處、某變數中，然後透過 key 來取值。此方法較耗費 memory。
>><font color="#008000">- trade-off: CPU usage vs. memory



---
## Container Objects
- Containers are objects that implement a __ contains __ method (that usually returns a Boolean value).
- 當物件使用 in, not in 時呼叫 ( This method is called in the presence of the in keyword of Python.)

```PYTHON
class GuessAge():
  def __init__(self):
    self.age_range = range(20, 85)    

  def __contains__(self, age):
    if age in self.age_range:
      return True
    else:
       False
```

```
>>> guess = GuessAge()

>>> print( 26 in guess)
True

>>> print( 14 in guess)
False
```

---
## Dynamic Attributes for Objects
- __ getattr __
- When implementing __ getattr __, raise AttributeError. Use it carefully.

```PYTHON
class Esuner():
  average_age = 32 # 定義一個變數

  def show_salary(): # 定義一個函式
    print('averge pay is $3000')
```

```
>>> Esuner.average_age
32

>>> Esuner.show_salary()
averge pay is $3000

>>> Esuner.height
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-20-e9222c1fc334> in <module>()
----> 1 Esuner.height

AttributeError: type object 'Esuner' has no attribute 'height'
```

><font color="#dd0000">原本沒有定義 height 這個屬性，所以呼叫不到會報錯


```PYTHON
class Esuner2():
  average_age = 32 # 定義一個變數

  def show_salary(self): # 定義一個函式
    print('averge pay is $3000')

  def __getattr__(self, new_attr):
    self.new_attr = new_attr
    raise AttributeError('此屬性原本不存在，但已創造成功，請賦值')
  
  def __setattr__(self, new_attr, value):
    self.__dict__[new_attr] = value
```

```
>>> target = Esuner2()

>>> target.average_age
32

>>> target.show_salary()
averge pay is $3000


>>> target.height
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-43-98b0097098a6> in <module>()
----> 1 target.height

<ipython-input-39-3649ac890765> in __getattr__(self, new_attr)
      7   def __getattr__(self, new_attr):
      8     self.new_attr = new_attr
----> 9     raise AttributeError('此屬性原本不存在，但已創造成功，請賦值')
     10 
     11   def __setattr__(self, new_attr, value):

AttributeError: 此屬性原本不存在，但已創造成功，請賦值
```

><font color="#dd0000">雖然成功創造新屬性，但作者強烈建議還是要 raise AttributeError

```
>>> target.height = 180
>>> target.height
180
```
---
## Callable Objects
- It is possible (and often convenient) to define objects that can act as functions.
- 可以讓類別的實體物件表現得像函式一樣，可以被呼叫。__ call __ 在那些類的實體物件經常改變狀態的時候會非常有效。The magic method __ call __ will be called when we try to execute our object as if it were a regular function.
- When we have an object, a statement like this object(args, **kwargs) is translated in Python to object.__ call __(args, **kwargs).

```PYTHON
from collections import defaultdict

class Persona():
  def __init__(self, heigt, weight, salary):
    self.heigt = heigt
    self.weight = weight
    self.salary = salary
  
  def __call__(self, weight, salary): # 成年之後，身高不太會變動，因此只保留體重與薪水的修改彈性
    self.weight = weight
    self.salary = salary
```

```
>>> no_00001 = Persona(180, 75, 50000)
>>> no_00001.weight
75

>>> no_00001(85, 60000)
>>> no_00001.weight
85
```

---
## Summary of Magic Methods
- in this chapter, we explored the core of Python: its protocols and magic methods.
- 如果要讓你自定義的類別或類別實體物件可以和某些語句 ( statement ) 搭配使用的話，那你就要在定義類別時，把能實現相對應功能的 magic method 寫進去。


| Statement | Magic method | Python concept |
| -------- | -------- | -------- |
|  obj[key]<br>obj[i:j]<br>obj[i:j:k]      | __ getitem __ (key)     | Subscriptable object    |
| with obj: ...     | __ enter __ / __ exit __      | Context manager     |
| for i in obj: ...     |  __ iter __ / __ nex __ <br> __ len __ / __ getitem __     | Iterable object <br>Sequence     |
| obj.<attribute>      |  __ getattr __     | Dynamic attribute retrieval     |
| obj(*args, **kwargs)     | __ call __(*args, **kwargs)     | Callable object     |

---
## Caveats in Python
- 如果有看到，應該被修正!
### Mutable Default Arguments
- don't use mutable objects as the default arguments of functions.
```PYTHON
def wrong_display(user_data: dict = {'name': 'Kevin', 'age': 18}):
  name = user_data.pop('name')
  age = user_data.pop('age')
  return f" {name}({age}) "
```

><font color="#dd0000">問題1: the default mutable argument, 
問題2: the body of the function is mutating a mutable object



```
>>> wrong_display()
'Kevin(18)'

>>> wrong_display({'name': 'Emily', 'age': 16})
'Emily(16)'
```
```
>>> wrong_display()
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-75-a00ea0cfb7ed> in <module>()
----> 1 wrong_display()

<ipython-input-71-2516f2172109> in wrong_display(user_data)
      1 def wrong_display(user_data: dict = {'name': 'Kevin', 'age': 18}):
----> 2   name = user_data.pop('name')
      3   age = user_data.pop('age')
      4   return f"{name}({age})"
      5 

KeyError: 'name'
```
><font color="#dd0000"> - 只有在第 1 次執行時 ( 使用預設參數而沒有輸入值 ) 能運作
><font color="#dd0000"> - 之後使用此函式時，若 user_data 因沒有吃到新的資料 ( 原本 dict 裡的東西已經被 pop 刪除 )，會出現 KeyError


><font color="#008000">解法1: 把 dict 的預設值設為 None
解法2: 將原本想做為預設值的內容指派到函式的主體


```PYTHON
def  good_display(user_data: dict = None):
  user_data = user_data or {'name': 'Kevin', 'age': 18}
  name = user_data.pop('name')
  age = user_data.pop('age')
  return f" {name}({age}) "
```

```
>>> good_display()
' Kevin(18) '

>>> good_display({'name': 'Emily', 'age': 16})
' Emily(16) '

>>> good_display()
' Kevin(18) '
```

><font color="#008000">就算沒輸入參數，也能返回預設值



### Extending Built-In Types
* The correct way of extending built-in types such as lists, strings, and dictionaries is by means of the collections module.
  - collections.UserDict
  - collections.UserList
  - collections.UserString
* 如果不用 **collections** 模組，而是直接擴充內建類型，如 dict，那你可以不會得到的預期的結果
```PYTHON
class BadList(list):
  def __getitem__(self, index):
    value = super().__getitem__(index)
    if index % 2 == 0:
      prefix = 'even'
    else:
      prefix = 'odd'
    return f" [{prefix} {value}] " # 將 value 轉成 str 類型
```

```
>>> b1[0]
' [even 1] '

>>> b1[1]
' [odd 2] '
```


```
>>> ",".join(b1)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-108-717b86fcb8a0> in <module>()
----> 1 ",".join(b1)

TypeError: sequence item 0: expected str instance, int found
```

><font color="#dd0000">- 當執行歷遍時，負責轉變型態為 str 的 __ getitem __ method 並沒有被呼叫
><font color="#dd0000">- This issue is actually an implementation detail of CPython (a C optimization), and in other platforms such as PyPy it doesn't happen.



><font color="#008000">解法: 使用 collections 模組


```PYTHON
from collections import UserList 

class GoodList(UserList):
  def __getitem__(self, index):
    value = super().__getitem__(index)
    if index % 2 == 0:
      prefix = 'even'
    else:
      prefix = 'odd'
    return f" [{prefix} {value}] " # 將 value 轉成 str 類型
```

```
>>> g1 = GoodList((1,2,3,4, 5))

>>> print(g1[0])
 [even 1] 
 
>>> print(g1[1])
 [odd 2] 
 
>>> ",".join(g1)
' [even 1] , [odd 2] , [even 3] , [odd 4] , [even 5] '
```

