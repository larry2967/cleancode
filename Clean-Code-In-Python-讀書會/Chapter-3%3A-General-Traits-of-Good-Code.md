## Design by Contract 依合約設計
### 概念
將程式的輸入、輸出、副作用都文件化，取得共識，將大家都認同違法合約的狀況引發exception，明確的指出為何無法繼續執行。
- 先決條件 (precondition): （必要）在程式執行前做的檢查，確保所有參數與資料傳入都是正確的，只要驗證不過，就不能執行主要程式。
- 後置條件 (postcondition): （必要）在呼叫函式或return前執行，確保程式產出是呼叫方希望收到的東西
- 不變量 (invarient): 在函式執行時保持不變的東西，最好用docstring將之文件化
      - 類別不變量:驗證物件某個時間點下的狀態
      - 內部不變量:使用斷言取代註解
      - 流程不變量:斷言程式流程中絕不會執行到的程式碼部分。

**內部不變量搭配assert**
~~~python
if balance >= 10000:
    ...
elif 10000 > balance >= 100:
    ...
else: # balance 一定是少於 100 的情況
    ...
~~~
~~~python
if balance >= 10000:
    ...
else if 10000 > balance >= 100:
    ...
else:
    assert balance < 100, balance
    ...
~~~
**流程不變量搭配assert**
~~~python
def foo(list):
    for ele in list:
        if ...:
            return
    # 這邊應該永遠不會被執行到
~~~
~~~python
def foo(list):
    for ele in list:
        if ...:
            return
    assert False
~~~
- 副作用: 可能的副作用寫在docstring中，比如說可能會更動或覆寫掉其他值

####  :star: 無累贅原則(nonredundancy principle)
函式的precondition只能讓合約雙方之一執行，而不能讓兩邊都執行

### 實作
- 先決條件 (precondition): 主要希望把函式的主程式和檢查程式分開，可用decorator實作
  - 可以用靜態分析工作mypy來檢查
  - 記住nonredundancy principle，檢查的函式放在使用方或是函式本身，擇一即可(與等等的DRY有關)
- 後置條件 (postcondition): 使用try...except，若是遇到沒有default error-type可以解決的問題，可以**自訂例外**

### 小結

- 有效找出問題所在
- 幫助我們釐清程式的結構
- 先決條件與使用方有關（如果它想要執行某部分的程式，就有義務配 合那些程式），而後置條件則與負責提供“使用方可以驗證的保證”的元件有關。

## Defensive Programming 防禦型程式設計
>保護自己的設計
### 概念
Defensive Programming的重點在於如何處理接下來可能發生的錯誤(Error handling procedures)，或是如何處理不該出現的錯誤(assertion)。
### 1. Error handling procedures    
#### a. 換值
- 優缺點：將結果換成另一個比較安全的值傳進函式中，雖可避免程式crash，但有可能會產出不正確的結果。<font color='Red'>**我們不鼓勵你這樣做R**</font>
- 適用情況：
    - 環境變數的預設值未被設定
    - config(設定檔)裡缺少項目
    - 函式缺少函數
- 實作：
    - 可以替**參數定義預設值**
    - 利用 dict 的 get()
~~~python
# self-defined
def connect_db(host='localhost', port=5432)
    ...
~~~
#### b. 紀錄錯誤
通常有traceback和logging兩種方法
- traceback是紀錄錯誤訊息
- logging可以做到紀錄訊息，但不一定是錯誤訊息，功能比較多元一點
   - logging 模組也提供可以紀錄完整的堆疊追蹤 (stack traces)，若在 logging.error() 加上 exc_info 參數，並將該參數設為 True，就可以紀錄 Exception
   ~~~python
   import logging
   try:
       x = 5 / 0
   except:
       logging.error("Catch an exception.", exc_info=True)
   ~~~
   - 若要在 logging 內紀錄 exception 訊息，可使用 logging.exception()，它會將 exception 添加至訊息中，此方法的等級為 ERROR，也就是說 logging.exception() 就等同於 logging.error(exc_info=True)
   ~~~python
   import logging
   try:
       x = 5 / 0
   except:
       logging.exception('Catch an exception.')
   ~~~
   - 若不想使用 ERROR 級別紀錄 exception 訊息，可使用 DEBUG、INFO、WARNING、CRITICAL 級別並加上參數 exc_info=True 的設定：
   ~~~python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   try:
        x = 5 / 0
   except:
        logging.debug('Catch an exception.', exc_info=True)
        logging.info('Catch an exception.', exc_info=True)
        logging.warning('Catch an exception.', exc_info=True)
        logging.critical('Catch an exception.', exc_info=True)
   ~~~

#### c. 處理例外
- 在有錯誤發生時，明確的指出有例外情形，而不是根據商業邏輯來改變程式流程。
- 在真的有需要讓使用方知道的時候，才發出exception
- 如果你的程式發出太多的exception，是時候考慮把函式拆小點了

實作：
記錄例外、留下Log、回傳預設值、或發出不同的exception.
可以利用`raise <e> from <original_exception>`

#### <font color='Red'>一個函式只做一件事！！！程式的例外處理也不例外！（關注點的概念）</font>
概念：將error handling抽象化成多個function，以抽象層拆開例外的物件


#### <font color='Red'>Error should not pass silently！</font>
千萬不要有空的except區塊，如果遇到問題，要知道如何修正

#### <font color='Red'>不要公開traceback</font>
可以讓程式開發者了解怎麼發生error，但是絕對不要公開
特別是你的服務有對外的時候，駭客就可以嘗試性地去攻擊你的網站
因為你會回覆錯誤，等於讓駭客有跡可循。通常用個找不到網頁或是出了一些錯誤，籠統地回覆client端會比較好


### 2. assertions
assert出現的時候，代表這東西根本不該存在，要直接中斷程式執行。
- 有時會結合 unit test 使用。(Yi 喜歡)
- 在大型framework間內部程式溝通的檢查。
~~~python
>>> def Divide(a,b):
...     assert b!=0, "b cannot be 0"
...     return a/b

>>> Divide(5,0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in Divide
AssertionError: b cannot be 0
~~~
### 3. <font color=Red>**內聚性與耦合性**</font>

- 內聚性: 一次只做好一件事，像是Unix命令
- 耦合性: 代表兩個以上的物件彼此依賴的程度，過度依賴是不好的
  - 無法重複使用程式碼
  - 連鎖反應
  - 低階的抽象

:star:好的程式應該要有高內聚和低耦合 -> Interface要定義好很重要

~~~python
# 高內聚低耦合
def calculate_price(base_price: float, tax: float, discount: float) -> float:
       return (base_price * (1 + tax)) * (1 - discount)

def show_price(price: float) -> str:
    return "$ {0:,.2f}".format(price)
	
def show_price1(price: float) -> str:
    return "$ {0:,.4f}".format(price)

def show_price2(price: float) -> str:
    return "$ {0:,.0f}".format(price)

def str_final_price(
    base_price: float, tax: float, discount: float, fmt_function=str
) -> str:
    return fmt_function(calculate_price(base_price, tax, discount))

# 低內聚高耦合
def str_final_price(
    base_price: float, tax: float, discount: float, fmt='str'
) -> str:
    
    price = (base_price * (1 + tax)) * (1 - discount)
    
    if fmt == 'str':
        return str(price)
    elif fmt == 'show_price':
        return "$ {0:,.2f}".format(price)
    elif fmt == 'show_price1':
        return "$ {0:,.4f}".format(price)
    elif fmt == 'show_price2':
        return "$ {0:,.0f}".format(price)
    else:
        raise NameError(f"'{fmt}' format not defined")
~~~
<font color=red>* 高內聚低耦合</font>
![image.png](.attachments/image-4e543b9a-fe35-460c-bbc0-c85188a0c336.png)
<font color=red>* 低內聚高耦合</font>
![image.png](.attachments/image-67b5b18c-15af-4ebc-8adf-bfd66fef5f53.png)
上面兩個範例用起來幾乎一模一樣，但後者的程式碼混在一起，後續使用上會非常困難，大大降低 reusability。


## Design Concept
### 1. DRY/OAOO
#### Don't Repeat Yourself. / Once and Only once.
#### :star:單一來源原則(single source of truth)
所有的東西在一個地方定義，在一個地方修改。

    可能實作方式：
    * 建立函式
    * 建構全新物件
    * 環境管理器
    * 迭代器或產生器
    * 裝飾器
如果你不想這麼做，你可能要付出
- 容易出錯
- 維護成本很高
- 不靠譜，大家要記得哪邊要改，也代表你的程式碼品質較低
~~~python
#week2 db_tool.py 作業
def get_rawdata_db_conn(self):
    self.rawdata_db_info = self.config_pass['RawDataDbInfo' + self.role]
    pwd = crv_crypto.decrypt_str(self.rawdata_db_info['password'])
    conn = _psycopg2.connect(database=self.rawdata_db_info['database'],
                             host=self.rawdata_db_info['host'], port=self.rawdata_db_info['port'],
                             user=self.rawdata_db_info['user'], password=pwd)
    print('log in rawdata db as {0}'.format(self.rawdata_db_info['user']))
    return conn
def get_feature_db_conn(self):
    self.feature_db_info = self.config_pass['FeatureDbInfo' + self.role]
    pwd = crv_crypto.decrypt_str(self.feature_db_info['password'])
    default_schema = self.proj_name + '.'
    default_schema = _replace_schema(default_schema, os.environ['SCHEMA'])[:-1]
    conn = _psycopg2.connect(database=self.feature_db_info['database'],
                             host=self.feature_db_info['host'], port=self.feature_db_info['port'],
                             user=self.feature_db_info['user'], password=pwd,
                             cursor_factory=_MLaaSCursor, options=f"-c search_path={default_schema}")
    print('log in feature db as {0}'.format(self.feature_db_info['user']))
    print('use default schema {0}'.format(default_schema))
    return conn
~~~
大家用你的眼睛看看484很多一樣，我們避免寫太多重複的code
不如就合在一起，讓世界更美好一點吧XD
[yi指定佑瑋學長的作業參考](https://tfsa0w0p02:8080/tfs/IF/_git/if_code_quality?path=%2Fweek2_ch2%2Fdb_tool_12267.py&version=GBmaster)

### 2. YAGNI
#### You Ain't Gonna Need it. 
我們經常會在開發當中盡可能地迎合未來的可能需求，而為了迎合某些產生概率極低的需求而設計的成本是非常高的!
你可能深思熟慮花了很多時間設計，卻在未來兩三年內英雄無用武之地R。
- 不要預測你的未來需求，專心處理當前需求就好
- 設計是否有必要，應基於當前的需求為主，而不是為了因應未來的各種變化，而有畫蛇添足的設計
- 過度設計往往會延緩開發跌代的速度

總而言之，<font color=Red>:star:我們覺得是先求有再求好:star:</font>
<font color=#00AAAA>**請貫徹玉山的金句**</font>

### 3. KIS
Keep it Simple.
實作最精簡的功能來正確地解決問題，不要讓解決方案超乎需求的複雜。設計越簡單，可維護性越高。

For example.
~~~python
class sklearn.svm.SVC(C=1.0, kernel='rbf', degree=3, gamma='scale', coef0=0.0, shrinking=True, probability=False, tol=0.001, cache_size=200, class_weight=None, verbose=False, max_iter=-1, decision_function_shape='ovr', break_ties=False, random_state=None)[source]¶
~~~
其實這個是簡化版本，我們比較長看到的是KISS原則(Kepp It Simple & Stupid)
保持簡單易懂，我們應該致力於代碼的可理解性，降低複雜度也意味著維護變得簡單
引用Martin Fowler 的金句
![image.png](.attachments/image-7b2d001e-3184-485c-a940-aac9bfe512be.png =400x300)
<font color=#00AAAA>:star: Any fool can write code that a computer can understand. Good programmers write code that humans can understand.:star:</font>
**任何一個傻瓜都能寫出電腦可以理解的程式，但只有寫出人類容易理解的程式才是優秀的程式開發者。**


### 4. EAFP/LBYL
我們想先問大家，請大家憑直覺作答(主持人請大家舉手選A或B，不用猶豫全憑你的第六感)
請問大家如果我要從字典取出key相對的value，我應該會怎麼寫?
:star:有人要分享哪個寫法誰比較U質或有什麼想法ㄇ:star:

~~~python
#例子A
if "key" in test_dict:
    x= test_dict["key]
else:
    print("key不存在")
~~~~
~~~python
#例子B
try:
    x=test_dict["key"]
except KeyError:
    print("key不存在")
~~~
#### Easier to Ask Forgiveness than Permission. 道歉比獲得許可來得容易
~~~python
try: 
    with open(filename) as f:
    ... 
except FileNotFoundError as e:
    logger.error(e)
~~~
#### Look Before You Leap. 三思而後行
~~~python
if os.path.exists(filename): 
    with open(filename) as f: 
    ...
else:
    logger.error()
~~~
在Python中比較推崇EAFP，比起先檢查判斷，不如先做再來抓exception.

- 效能分析：
EAFP 的異常處理往往也會影響一點性能，因為在發生異常的時候，程式會進行保留現場、回溯traceback等操作，但在異常發生頻率比較低的情況下，性能相差的並不是很大。
而 LBYL 則會消耗更高的固定成本，因為無論成敗與否，總是執行額外的檢查。
相比之下，如果不引發異常，EAFP 更優一些。

- 易讀性
當性能速度不用作考慮條件時，來看看代碼的易讀性。. EAFP將業務邏輯代碼，跟防禦性代碼隔離開，讓開發者可以更專注于開發業務邏輯，不管資料變數是否合理，按照正常的邏輯思維執行下去，如果發生錯誤就在異常裡面糾正。
LBYL則容易打亂開發者的思維，在做一件事之前，總是要先要判斷能不能這樣做，傳入的資料是否合理，類型是否正確，需要增加很多判斷內容，代碼連貫性差，常見的一堆if判斷，多數就是這種風格導致。

- 武功流派
**你寫EAFP是比較符合<font color=Red>**pythonic**</font>的啦!!**
  - 編寫 Java/C/C++的時候通常都是採用 LBYL風格，很少在一些不可控的情況才會使用 EAFP 風格。
  - 語言之間的設計哲學差異，Java 對類型要求非常嚴格，要求明確，類/方法等，它假定你應該知道，任何時候你正在使用的物件類型，以及它能做什麼。
  - Python 的語法意味著你不必明確的知道物件的顯示類型是什麼，你只需要關心你在使用時候它能有相應的回饋。在這種寬鬆的限制下，唯一明確的態度就是認為代碼會工作，準備面對結果。

________________________________________________________________________________
BY Team 4
## Composition and inheritance
- 繼承主要目的是重用程式碼。
- 常見風險點：
1. 繼承父類別所有功能。
2. 耦合度會過高。
- 建議作法：製作高度內聚的物件，以便輕鬆組合他們，並應用在各種情境。

- Specialization ( 專門化 ) ：使用類別的公用方法與屬性定義元件。( 類別公用方法要與父類別定義一致 )
```
範例：

基礎類別 -> BaseHTTPServer.BaseHTTPRequestHandler.
擴展並加入額外功能為子類別 -> SimpleHTTPRequestHandler
```

- python 多重繼承
範例：
Q：ConcreteModuleA12 會使用 BaseModule1 or BaseModule2?
![image.png](.attachments/image-6b6e55cf-ac3f-4f67-a104-734c425b743e.png)

A：BaseModule1，而且並不會衝突，原因是 python 透過 C3 linearization 或 Method Resolution Order ( MRO ) 演算法來定義應該要呼叫什麼方法。

- python 多重繼承應用：Mixin
目的：Mixin 封裝了常見的基礎類別，主要是重用程式碼，本身並沒有用途，需搭配使用。

範例：
   1. 傳遞字串並透過符號切割
```python
class BaseTokenizer:
    
    def__init___(self,str_token):
       self.str_token = str_token

    def__iter__(self):
        yield from self.str_token.split("-")
```
```
>>> tk = BaseTokenizer("28a2320b-fd3f-4627-9792-a2b38e3c46b0")
>>> list(tk)
 ['28a2320b','fd3f','4627','9792','a2b38e3c46b0']
```
2. 新增轉換英文字大寫功能，使用 Mixin 功能達到。
```python
class UpperIterableMixin:
    def__iter___(self):
       return map(str.upper, super().__iter__())

class Tokenizer(UpperIterableMixin, BaseTokenizer):
     pass     
```

## Arguments in functions and methods
-  Arguments 如何傳入Functions
在python 裡，什麼都是物件，每個物件包含一個identity、type、value
 ![pythonobjectmutable.png](.attachments/pythonobjectmutable-5940cc6f-7302-4647-8308-f2f4375888ce.png =500x)

|  |意義| 看細節 | 可不可變 | 
| ----|---- | -------- | -------- |
| Identity |  Object 記憶體位址  | id()     | X     |
| Type   | 物件型態 | type()     | X     |
| Value   | 值 |  直接access   | **immutable** (Numeric types: int, float,  string, tuple, frozen set)  <br> **mutable**(list, dict, set, byte array) |

```Python
def sayhi(name):
    name += ', hi!'
    print(name)
    
# case1: string
immutable= 'Tim'

sayhi(immutable)
# output: Tim, hi!
print(immutable)
# output: Tim

# case2: list
mutable = list('Tim')

sayhi(mutable)
# output: Tim, hi!
print(mutable)
# output: ['T', 'i', 'm', ',', ' ', 'h', 'i', '!']
```

Questions: DataFrame的 value 是mutable 還是immutable?
```Python
def add_col(df):
    df['new_col'] = 1
    print(df)
    
d = {'col1': [1, 2], 'col2': [3, 4]}
df = pd.DataFrame(data=d)
add_col(df)
#output
#     col1  col2  new_col
# 0     1     3        1
# 1     2     4        1
print(df)
#output
#     col1  col2  new_col
# 0     1     3        1
# 1     2     4        1


add_col(df.copy())
#output
#     col1  col2  new_col
# 0     1     3        1
# 1     2     4        1
print(df)
# output
#     col1  col2
# 0     1     3
# 1     2     4
```

<font color='red'>Tips: 不要改變引數，避免造成不必要的副作用</font>

- 可變數量的引數
    - *args: positional argument
        - 優點：可依照自己想要的拆分方式
        ```python
        first, *middle, last = range(6)
        # first:0
        # *middle:[1,2,3,4]
        # last: 5
        ```
    - **kwargs
        - 把引數用dict的方式包裝
        ```python
        def func(**kwargs):
            print(kwargs)
        
        func(key='value')
        # output: {key:'value'}
        
        ```
    - 順序：Standard arguments, *args arguments , **kwargs arguments
    - 常見的use case: 當你要繼承某class，但又不希望前一個class引數更動的時候影響到你，缺點是會失去識別性跟易讀性
    ```python
    class MySubclass(Superclass):
        def __init__(self, *args, **kwargs):
            self.myvalue = kwargs.pop('myvalue', None)
            super(MySubclass, self).__init__(*args, **kwargs)
    ```

- 引數數量
  - 可解辦法：
      1. 物件化
      2. 用可變引數
  - 程式有越多引數，越可能與呼叫的function高度耦合
    ```python
    def print_welcome_msg(name):
        print("hi! welcome, "+ name)

    def get_item(name, item):
        print("{0} wants to buy item: {1}".format(name, item))

    def print_payment(method):
        print("payment {0}".format(method))

    # 為了要一次呼叫前面寫的function, 引數會越帶越多
    def print_cust_journey(name, item, method):
        print_welcome_msg(name)
        get_item(name, item)
        print_payment(method)

    print_cust_journey("Peter", "lunch", "cc")
    #output:
    #hi! welcome, peter
    #peter wants to buy item: lunch)
    #payment cc

    ```
  
  -  接收引數的緊湊函式簽章
        ```python
        def print_cust_journey(**kwargs):
            print_welcome_msg(kwargs['name'])
            get_item(kwargs['name'], kwargs['item'])
            print_payment(kwargs['method'])

        print_cust_journey(name="peter", item="lunch", method="cc")
        ```
        <font color='red'>Tips: 引數太多，可視為code smell</font>
- 函式的引數數量
  - 不良設計的徵兆(code smell，代碼異味) : 如果函式需要過多參數才能正常工作，可以視為代碼異味。 (是否有例子?)
  - 建議做法1: 為傳入的所有引數建立新物件 (具體化，reify)
  - 建議作法 2:使用可變的位置引數與關鍵字引數建立具備動態簽章的函式 (可拆多個小函式)
 #### <font color='Red'>強調!函式應該只做一件事，且只有一件事!!</font>
- 接收過多引數的緊湊函式簽章
```
範例：

track_request (request.headers, request.ip_addr, request.request_id)
 -> track_request(request) 
```
 參考文章：

分類 https://medium.com/@tonyhsu/%E5%88%9D%E6%AC%A1%E8%AA%8D%E8%AD%98-code-smells-%E8%88%87%E5%88%86%E9%A1%9E-4410ba4623c3

## Final remarks on good practices for software design
- 軟體的正交性(Orthogonality)==>"makes changes (or side-effects) local" 互相獨立
```
範例：
(如何計算價格)
def calculate_price(base_price: float, tax: float, discount: float) ->
return (base_price * (1 + tax)) * (1 - discount)  

(顯示價格)
def show_price(price: float) -> str:
return "$ {0:,.2f}".format(price)

def str_final_price(
base_price: float, tax: float, discount: float, fmt_function=str
) -> str:
return fmt_function(calculate_price(base_price, tax, discount))
```
- 解構程式碼 (根據相似性來放置元件)
  - 拆成套件 package 
  - 好處: 
     1. 模組被匯入時，需要解析與載入的記憶體比較少
     2. 模組本身可能匯入較少的模組，fewer dependencies


## Summary
- 程式碼式設計的一部分
  - 依照合約設計(Design by Contract) + 防禦性設計 (更為強健)
- exceptions , 避免將例外當成控制流程結構使用 (避免寫入商業邏輯)
- 物件導向設計 (object-oriented design) : 繼承 vs. 組合 
- number of arguments in functions :  clean design!! 永遠考慮Python 的特質
