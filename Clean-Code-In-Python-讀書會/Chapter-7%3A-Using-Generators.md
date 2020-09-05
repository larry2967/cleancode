# Chapter 7 : 產生器

## **為摸有產生器**
Python的產生器(generator)物件，主要是用來解決用可迭代物 (Iterable) 存取資料時，會受到記憶體容量限制的問題，包括集合數據類型（list、tuple、dict、set、str等）。
~~就是想要節省記憶體啦~~

## **名詞介紹**

- 可迭代物件 (iterable)
我們已經知道可以對list、tuple、dict、set、str等型別的資料使用for...in...的迴圈語法從其中依次拿到資料進行使用，我們把這樣的過程稱為遍歷，也叫迭代。因此可以通過for...in...這類語句迭代讀取資料供我們使用的物件稱之為可迭代物件（Iterable）。
``` python 
x = [1, 2, 3]
y = iter(x)
next(y)
>>> 1
next(y)
>>> 2
type(x)
>>> <class ‘list’>
type(y)
>>> <class ‘list_iterator’>
``` 
> 在這裡，x 是「可迭代」的物件，而 y 和 z 則是兩個互不相關的迭代器實例，它們從「可迭代物件」x 來產出資料。y 和 z 會有指向 x 某個元素的「狀態」，並依照某些規則輸出 x 裡的元素。在這個例子中，x 是 list 型別的資料結構，但這並非必要條件。

>實務面上，可迭代物件通常會實作 \__iter__() 和 \__next__() 這兩個方法，使得該物件同時是可迭代及迭代器。

可以使用 isinstance() 判斷一個物件是否是 Iterable 物件：
``` 
from collections import Iterable
isinstance([1,2,3], Iterable)
>>> True
``` 
- 迭代器物件 (iterator)
迭代器是具有狀態物件 (helper)，對它使用 next() 會產出下一個值。當物件實作了 \__next__() ，它就是迭代器。

- 產生器物件 (generator)
平時閒置著，直到你對它要求回傳值，才會回神並產出值，之後便回歸閒置狀態。
    - 產生器是一種特別的迭代器。
    - 產生器都是迭代器，但反過來不一定成立


## **產生器的範例**

### 先說說 return：回傳函數的執行結果，並代表函數的執行結束
``` python 
def func():
    print("我叫周潤發")
    return "林志玲"   # return在函式中表示返回的意思
print("返回值是", func())
```
### 再看看 yield：一直產生數字的函數，每次產生一個值，函數被凍結再喚醒，直到產出最後一個數字為止
yield就像是return會傳回值，但又不中斷函式的執行
``` python
# 函式中包含了yield, 此函式就是生成器函式
# 大坑: 生成器函式執行之後. 產生一個產生器. 而不是執行函式
def func():
    print("我叫周潤發")
    yield "林志玲"   # yield表示返回. 不會終止函式的執行
    print("寶寶幹嘛去了??")
    yield "寶寶回來了"
    print("寶寶你在幹嘛?")
    # yield "沒了"

ret = func() # 執行函式, 此時沒有執行函式.
# # 此時我們拿到的是產生器
print("返回值是", ret) 

# 執行到下一個yield
print(ret.__next__()) # 第一次執行__next__此時函式才開始執行
print('-------------')
print(ret.__next__()) # 執行到下一個yield
print('-------------')
print(ret.__next__()) # StopIteration
```
### 執行畫面
``` terminal
返回值是 <generator object func at 0x10f554308>
我叫周潤發
林志玲
-------------
寶寶幹嘛去了??
寶寶回來了
-------------
寶寶你在幹嘛?
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-2-57508ef49576> in <module>
     16 print(ret.__next__()) # 執行到下一個yield
     17 print('-------------')
---> 18 print(ret.__next__()) # StopIteration

StopIteration:
```
## **產生器的使用範例**
### 基本用法
``` python 
print([x**2 for x in range(10)]) # 典型的列表推導
>>> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

print({x**2 for x in range(10)}) 
>>> {0, 1, 64, 4, 36, 9, 16, 49, 81, 25}

print((x**2 for x in range(10))) # 典型的產生器表示式
>>> <generator object <genexpr> at 0x10f554830>
```
### 記憶體容量比較
最後一個是產生器，我們可以按照疊代器的使用方法來使用，即可以通過next()函數、for循環、list()等方法使用。

#### <font color=red>永遠傳遞產生器運算式給可迭代物的函式，例如min()、max()、sum()</font>
``` python
# Generator
sum(x**2 for x in range(10000000))
>>> 285

#list
sum([x**2 for x in range(10000000)])
>>> 285

```
#### 用Generator可以省多少？
|方法|佔用記憶體|
|--|--|
|sum(Generator)|50.3984 MB|
|sum(list)|436.4609 MB|

>https://stackoverflow.com/questions/9850995/tracking-maximum-memory-usage-by-a-python-function

# 典型迭代寫法
* enumerate
  * 用於在一個可迭代或可遍歷的物件（如列表、字串、陣列、字典）中，將物件組成一個一個序列和索引，可以同時獲得索引和索引值。
```python
>>> list(enumerate("abcd"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd')]
```

* 運用更底層的方式，建立無窮序列的物件
```python
class NumberSequence:
    def __init__(self, start=0):
        self.current = start
    
    def next(self):
        current = self.current
        self.current += 1
        return current
```

```python
>>> seq = NumberSequence()
>>> seq.next()
0
>>> seq.next()
1

>>> seq2 = NumberSequence(10)
>>> seq2.next()
10
>>> seq2.next()
11
```
* 無法與 enumerate 產生相同的結果
* 無法當作參數傳給可迭代物件的函式
* NumberSequence 不支援迭代
```
>>> list(zip(NumberSequence(), "abcd"))
Traceback (most recent call last):
  File "...", line 1, in <module>
 TypeError: zip argument #1 must support iteration
 ```


* 透過 __iter__使物件可被迭代
* 使用 __next__取代 next()

```python
class SequenceOfNumbers:
    def __init__(self, start=0):
        self.current = start
        
    def __next__(self):
        current = self.current
        self.current += 1
        return current
        
    def __iter__(self):
        return self
```
* 可迭代
* 可使用next()內建函式
```python
>>> list(zip(SequenceOfNumbers(), "abcd"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd')]
>>> seq = SequenceOfNumbers(100)
>>> next(seq)
100
>>> next(seq)
101
```

* next()
    * 可將迭代物一到下一個元素並回傳
```python
>>> word = iter("hello")
>>> next(word)
'h'
>>> next(word)
'e' # ...
```
* 若無元素可產生，引發 StopIteration exception
    
```python
>>> ...
>>> text(word)
'o'
>>> next(word)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```


* 提供預設值參數，不丟出 StopIteration
```python
>>> next(word, "default value")
'default value'
```

# 使用產生器

```python
def sequence(start=0):
    while True:
        yield start
        start += 1
```

```python
>>> seq = sequence(10)
>>> next(seq)
10
>>> next(seq)
11

>>> list(zip(sequence(), "abcd"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd')]
```

* 重複的迭代

itertools 提供許多能產生迭代器的工具函式：
``` python 
from itertools import count
counter = count(start=13)
next(counter)
>>> 13
next(counter)
>>> 14
``` 

從有限可迭代物件生成無窮迭代器：
``` python 
from itertools import cycle
colors = cycle([‘red’, ‘white’, ‘blue’])
next(colors)
>>> ‘red’
next(colors)
>>> ‘white’
next(colors)
>>> ‘blue’
next(colors)
>>> ‘red’
``` 
* itertools.tee
  * 當對一個物件執行多次迴圈，可考慮使用 itertools.tee
  * 同時處理各類型迭代，不需重複執行多次迴圈

```python
def process_purchases(purchases):
    min_, max_, avg = itertools.tee(purchases, 3)
    return min(min_), max(max_), median(avg)
```
* 嵌套的迴圈

* 多維度找值，找到值要停止迭代，但break無法完全生效，因必須跳出多個for迴圈
```python
def search_nested_bad(array, desired_value):
    coords = None
    for i, row in enumerate(array):
        for j, cell in enumerate(raw):
            if cell == desired_value:
                foords = (i, j)
                break
                
        if coords is not None:
            break
    
    if coords is None:
        raise ValueError(f"{desired_value} not found")
        
    logger.info("value %r found at [%i, %i]", desired_value, *coords)
```
* 簡化版，不使用break跳離迴圈
```python
def _iterate_arrayd2(array2d):
    for i, row in enumerate(array2d):
        for j, cell in enumerate(row):
            yield (i, j), cell
            
def search_nested(array, desired_value):
    try:
        coord = next(
            coord
            for (coord, cell) in _iterate_array2d(array)
                if cell == desired_value
            )
        except StopIteration:
            raise ValueError("{desired_value} not found")
            
    logger.info("value %r found at [%i, %i]", desired_value, *coord)
    return coord
```

# 協程 (Coroutine)

以下資料參考：
1. [深入理解Python的yield from语法](https://juejin.im/post/5b3af9fb51882507d4487144)
2. [淺談coroutine與gevent](http://blog.ez2learn.com/2010/07/17/talk-about-coroutine-and-gevent/)

### 為何要使用協程？

假如我們做一個爬蟲。我們要爬取多個網頁，這裡簡單舉例兩個網頁(三個spider函數)，獲取HTML（耗IO耗時間），然後再對HTML對行解析取得我們感興趣的數據。

~~~python
def spider_01(url):
    html = get_html(url) # IO
    ...
    data = parse_html(html)

def spider_02(url):
    html = get_html(url) # IO
    ...
    data = parse_html(html)

def spider_03(url):
    html = get_html(url) # IO
    ...
    data = parse_html(html)
~~~

我們都知道，<font color=red>get_html()</font> 等待返回網頁是非常耗IO的，一個網頁還好，如果我們爬取的網頁數據極其龐大，這個等待時間就非常驚人，是極大的浪費。

聰明的程序員，當然會想如果能在 <font color=red>get_html()</font> 這裡暫停一下，不用傻乎乎地去等待網頁返回，而是去做別的事。等過段時間再回過頭來到剛剛暫停的地方，接收返回的html內容，然後還可以接下去解析 <font color=red>parse_html(html)</font>。

這時候 coroutine 就出現了，將程序在 IO 等待時切換到不同的 process 執行，而且還可以將使用者傳入的資訊透過 send 傳入(ex. 使用者帳密)，並繼續執行網頁。

### 從 yield from 認識 coroutine
* 簡單應用：拼接可迭代對象

使用 <font color=red>yield</font>
~~~python
# 字符串
astr='ABC'
# 列表
alist=[1,2,3]
# 字典
adict={"name":"wangbm","age":18}
# 生成器
agen=(i for i in range(4,8))

def gen(*args, **kw):
    for item in args:
        for i in item:
            yield i

new_list=gen(astr, alist, adict， agen)
print(list(new_list))
# ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
~~~
使用 <font color=red>yield from</font>
~~~python
# 字符串
astr='ABC'
# 列表
alist=[1,2,3]
# 字典
adict={"name":"wangbm","age":18}
# 生成器
agen=(i for i in range(4,8))

def gen(*args, **kw):
    for item in args:
        yield from item

new_list=gen(astr, alist, adict, agen)
print(list(new_list))
# ['A', 'B', 'C', 1, 2, 3, 'name', 'age', 4, 5, 6, 7]
~~~
由上面兩種方式對比，可以看出，yield from後面加上可迭代對象，他可以把可迭代對象裡的每個元素一個一個的yield出來，對比yield來說代碼更加簡潔，結構更加清晰。
* 複雜應用：生成器的嵌套 (透過 yield from)

名詞定義
> 1、調用方：調用委派生成器的客戶端（調用方）代碼
2、委託生成器：包含yield from表達式的生成器函數
3、子生成器：yield from後面加的生成器函數

舉例：實現即時計算平均值

比如，第一次傳入10，那返回平均數自然是10.
第二次傳入20，那返回平均數是(10+20)/2=15
第三次傳入30，那返回平均數(10+20+30)/3=20

~~~python
# 子生成器
def average_gen():
    total = 0
    count = 0
    average = 0
    while True:
        new_num = yield average
        count += 1
        total += new_num
        average = total/count

# 委托生成器
def proxy_gen():
    while True:
        yield from average_gen()

# 调用方
def main():
    calc_average = proxy_gen()
    next(calc_average)            # 预激下生成器
    print(calc_average.send(10))  # 打印：10.0
    print(calc_average.send(20))  # 打印：15.0
    print(calc_average.send(30))  # 打印：20.0

if __name__ == '__main__':
    main()
~~~

委託生成器，只起一個橋樑作用，它建立的是一個雙向通道，它並沒有權利也沒有辦法，對子生成器yield回來的內容做攔截。

~~~python
# 子生成器
def average_gen():
    total = 0
    count = 0
    average = 0
    while True:
        new_num = yield average
        if new_num is None:
            break
        count += 1
        total += new_num
        average = total/count

    # 每一次return，都意味着当前协程结束。
    return total,count,average

# 委托生成器
def proxy_gen():
    while True:
        # 只有子生成器要结束（return）了，yield from左边的变量才会被赋值，后面的代码才会执行。
        total, count, average = yield from average_gen()
        print("计算完毕！！\n总共传入 {} 个数值， 总和：{}，平均数：{}".format(count, total, average))

# 调用方
def main():
    calc_average = proxy_gen()
    next(calc_average)            # 预激协程
    print(calc_average.send(10))  # 打印：10.0
    print(calc_average.send(20))  # 打印：15.0
    print(calc_average.send(30))  # 打印：20.0
    calc_average.send(None)      # 结束协程
    # 如果此处再调用calc_average.send(10)，由于上一协程已经结束，将重开一协程

if __name__ == '__main__':
    main()
~~~