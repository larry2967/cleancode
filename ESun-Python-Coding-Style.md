
# <font color=#CC0000>**ESun Python Coding Style** </font>
---

### <font color=#00AAAA>文件圖示說明 </font>

- :wrench: : 強制，表示必須遵守，僅有極少數例外。
- :pushpin: : 建議或說明，表示在一定情境下可考慮之作法。


### <font color=#00AAAA>一、 基本命名原則 </font>
  
 :wrench: 1.  應該避免的名稱
> - 單字符名稱 (只允許計數器和疊代器使用)。
> - package/module 名字中不能使用連字符 (-)。
> - 雙下劃線開頭並結尾的名稱 (Python 的保留字，例如 `__init__` )

 :wrench: 2. 類別、屬性、方法、函數或其他物件等識別項只要是非專屬於迴圈 Loop 中使用的，一律採明確名稱方式命名，其名稱需要明白的表明用途，並且易於發音以利溝通，非必要不可使用無意義的名稱
> - Example - Good
> ```python
> customer_name
> ```
> - Example - Bad
> ```python
> x1
> abc
> ```

:wrench: 3. 命名字元僅允許底線、英文、數字，嚴禁使用中文。

:wrench: 4. 命名原則上採用 Camel Case 和 Snake Case，非必要不得使用匈牙利命名法 Hungarian Notation。各種功能類別命名撰寫規範如下表：

> | Type                    |Public       | Internal    | Remark |
> | ------------------ | -----------| -----------|---|
> | Packages             | `lower_with_under`   |                                | Snake Case |
> | Modules                    | `lower_with_under ` | `_lower_with_under`              |  Snake Case |
> | Classes                    | `CapWords`          | `_CapWords`                      | Upper Camel Case |
> | Exceptions                 | `CapWords`          |                                | Upper Camel Case |
> | Functions                  | `lower_with_under()`| `_lower_with_under()`            | Snake Case |
> | Global/Class Constants     | `CAPS_WITH_UNDER`   |` _CAPS_WITH_UNDER`               | Snake Case |
> | Global/Class Variables     | `lower_with_under`  | `_lower_with_under`              | Snake Case |
> | Instance Variables         | `lower_with_under ` | `_lower_with_under` (protected用法) /  `__lower_with_under` (private用法)  | Snake Case |
> | Method Names               | `lower_with_under()`| `_lower_with_under()`(protected用法) / `__lower_with_under()` (private用法) | Snake Case |
> | Function/Method Parameters | `lower_with_under`  |                                | Snake Case |
> | Local Variables            | `lower_with_under`  |                                | Snake Case |
>
> 備註：
> -  駝峰命名法 ( Camel Case ) 分為大駝峰 ( Upper Camel Case 或是 Pascal Case ) 和小駝峰 ( Lower Camel Case )
> -  可以建立變數的白名單

 :pushpin: 5. 使用慣用的英文字。例如 Customer 就比 Client 來的恰當。

 :pushpin: 6. 命名要可以讓開發或維護的人員很快的就了解其意義，並以單字或單詞為原則，但也不宜過長，建議不超過30個英文字母。


###  <font color=#00AAAA>二、 類別(Class)開發原則 </font>

:pushpin: 1.  如果 class 沒有繼承自其他 class，就從 object 繼承。nested class 也套用此規則。
繼承自 object 是為了使屬性(properties)正常工作，並且這樣可以保護代碼，使其不受Python 3000的一個特殊的潛在不兼容性影響。這樣做也定義了一些特殊的方法，這些方法實現了包括 (`__new__`, `__init__`, `__delattr__`, `__getattribute__`, `__setattr__`, `__hash__`, `__repr__`, and `__str__` )。
> - Example - Good
> ```python
> class SampleClass(object):
>     pass
>
> class OuterClass(object):
>     class InnerClass(object):
>         pass
>
> class ChildClass(ParentClass):
>     """Explicitly inherits from another class already."""
 - Example - Bad
> ```python
> class SampleClass:
>     pass
> 
> class OuterClass:
>     class InnerClass:
>         pass
> ```

:pushpin: 2. 一般類別 Class 應依實際代表的角色來命名。
> - 例： Product 代表產品， Customer 代表顧客

 :pushpin: 3. 類別中的 Method 名稱應該依照他要執行的動作的英文動詞來命名。
> - 例： Copy(), Create(), FindArticle()

 :pushpin: 4. 當動作對象或回傳物件為複數時，應用複數名詞表示。
> - 例： GetProducts(), MoveFiles(), GetCaseList()

 :pushpin: 5.方法應以執行單一動作 (single action) 為原則，非必要情況，應避免在方法內跨越不同類別，以降低耦合性

 :pushpin: 6. 建議使用適當的封裝。訪問和設置數據成員時，通常會使用簡單，輕量級的訪問和設置函數。建議用屬性（properties）來代替它們。

> - Example
> ```python
> import math
>
> class Square(object):
>     """A square with two properties: a writable area and a read-only perimeter.
>
>     To use:
>     >>> sq = Square(3)
>     >>> sq.area
>     9
>     >>> sq.perimeter
>     12
>     >>> sq.area = 16
>     >>> sq.side
>     4
>     >>> sq.perimeter
>     16
>     """
>
>     def __init__(self, side):
>         self.side = side
>
>     def __get_area(self):
>         """Calculates the 'area' property."""
>          return self.side ** 2
>
>     def ___get_area(self):
>         """Indirect accessor for 'area' property."""
>         return self.__get_area()
>
>     def __set_area(self, area):
>         """Sets the 'area' property."""
>         self.side = math.sqrt(area)
>
>     def ___set_area(self, area):
>         """Indirect setter for 'area' property."""
>         self._SetArea(area)
>
>     area = property(___get_area, ___set_area,
>                     doc="""Gets or sets the area of the square.""")
>
>     @property
>     def perimeter(self):
>         return self.side * 4
> ```


### <font color=#00AAAA> 三、 Module / Function / Method 開發原則 </font>

即使是一個打算被用作腳本的文件，也應該是可以被導入 (import) 的。且導入時不應該導致這個腳本的主功能被執行。

:wrench: 1.  主功能應該放在一個 main() 函數中。

:wrench: 2. Module 間禁止互相導入。

:wrench: 3. 在 Python 中，pydoc 以及單元測試要求 module 必須是可導入的。程式碼應該在執行主程序前都要檢查 ``` if __name__ == '__main__' ```，避免 module 被導入時 `main()` 就不會被執行。
> - Example
> ```python
> def main():
>      ...
>
>if __name__ == '__main__':
>    main()
>```

:wrench: 4. Arguments 不得被重新定義。


### <font color=#00AAAA> 四、 Function開發原則 </font>

:pushpin: 1. Lambda 函數僅適用於單行函數。如果代碼超過 60-80 個字符，必須要定義成 regular 函數 (regular function)。

:pushpin: 2. 定義函數或方法時，不要使用變數作為默認值。
> - Example - Good
> ```python
> def foo(a, b=None):
>     if b is None:
>         b = []
> ```
> 
> - Example - Bad
> ```python
> def foo(a, b=[]):
>     ...
> def foo(a, b=time.time()):  # time.time()是指哪個時間?
>     ...
> def foo(a, b=FLAGS.my_thing):  # sys.argv 還沒被傳送...
>     ...
> ```

:pushpin: 3. 條件表達式 (Conditional Expressions) 適用於單行函數。在其他情況下，推薦使用完整的 if 語句。 
> - Example
> ```python
> x = true_value if condition else false_value
> 
> if condition:
>     x = true_value
> else:
>     x = false_value
>     ...
> ```

:pushpin: 4. 注意在生成器函數 (Generators) 的文檔字串中使用 `yields:` 而不是 `returns:`。

:pushpin: 5. 當有 Code 重複時，應考慮抽離重複部份製作 method，或應用其他有效解決 code 重複的重構技巧


### <font color=#00AAAA>五、 變數開發原則 </font>

:wrench: 1. 在同一文件中，統一使用單引號。
> - Example - Good
> ```python
> address = 'home'
> Python('Why are you hiding your eyes?')
> Gollum('I\'m scared of lint errors.')
> Narrator('"Good!" thought a happy Python reviewer.')
> ```
> - Example - Bad
> ```python
> address = "home"
> Python("Why are you hiding your eyes?")
> Gollum("Always the great lint. Watching. Watching.")
> ```

:pushpin: 2. 避免全域變數。如果需要，全域變數應該僅在 module 內部可用，並調整成公共函數來使用。

:pushpin: 3. 連結字串時請判定 `+` 和 `%` 之間的使用情境。連結字串變數使用 `+`，特定字串格式或是連結混合型態使用 `%`。
> - Example - Good
> ```python
> x = a + b
> x = '%s, %s!' % (imperative, expletive)
> x = '{}, {}!'.format(imperative, expletive)
> x = 'name: %s; score: %d' % (name, n)
> x = 'name: {}; score: {}'.format(name, n)
> ```
> - Example - Bad
> ```python
> x = '%s%s' % (a, b)  # use + in this case
> x = '{}{}'.format(a, b)  # use + in this case
> x = imperative + ', ' + expletive + '!'
> x = 'name: ' + name + '; score: ' + str(n)
> ```

:pushpin: 4. 避免在循環中用 `+` 和 `+=` 運算子來累加字串。

:pushpin: 5. 使用 iterator 連接字串時，可以將每個子串加入列表( `append`)，然後在循環結束後用 `.join()` 連接列表。(也可以將每個子串寫入一個`cStringIO.StringIO` )

> - Example - Good
> ```python
> items = ['<table>']
> for last_name, first_name in employee_list:
>     items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
> items.append('</table>')
> employee_table = ''.join(items)
> ```
> - Example - Bad
> ```python
> employee_table = '<table>'
> for last_name, first_name in employee_list:
>     employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
> employee_table += '</table>'
> ```


### <font color=#00AAAA> 六、 註解原則 </font>

**Docstrings 規範**
說明：**methods**、**functions** 與 **class** 應該透過 **docstrings** 提供足夠的信息，使他人調用該函數時不需要看代碼即可使用。

:wrench: 1. 使用三重雙引號 """ 包圍 docstrings，撰寫規範：
> (1) 先一行以句號、問號或驚歎號為結尾的簡短說明。
> (2) 接著之後的每段之間要有一個空行，並且與第一行的第一個引號對齊。
> (3) 每段以一個標題行開始，標題行以冒號結尾。標題行外的其他說明內容應被縮排 4 個空格。
>
> - Example
> ```python
> def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
>     """Fetches rows from a Bigtable.
> 
>     Retrieves rows pertaining to the given keys from the Table instance
>     represented by big_table. Silly things may happen if
>     other_silly_variable is not None.
> 
>     Args:
>         big_table: An open Bigtable Table instance.
>         keys: A sequence of strings representing the key of each table row
>             to fetch.
>         other_silly_variable: Another optional variable, that has a much
>             longer name than the other args, and which does nothing.
> 
>     Returns:
>         A dict mapping keys to the corresponding table row data
>         fetched. Each row is represented as a tuple of strings. For
>         example:
> 
>         {'Serak': ('Rigel VII', 'Preparer'),
>          'Zim': ('Irk', 'Invader'),
>          'Lrrr': ('Omicron Persei 8', 'Emperor')}
> 
>         If a key from the keys argument is missing from the dictionary,
>         then that row was not found in the table.
> 
>     Raises:
>         IOError: An error occurred accessing the bigtable.Table object.
>     """
>     ...
> ```
> - 補充說明：可透過 `__doc__` 取得 docstring 的說明
> ``` python
> fetch_bigtable_rows.__doc__
> ```


**Docstrings 規範：methods 與 functions**

:wrench: 2. 包涵 methods, functions 跟 generators 都必須要有 docstrings。除非滿足以下任一條件:
> - 外部不可見
> - 簡單明瞭

:wrench: 3. docstrings 中 **Args** 的撰寫方式：
> (1) 列出每個參數的名字，並在名字後使用一個冒號和一個空格，分隔對該參數的描述，描述需包涵類型的說明。
> (2) 如果描述太長超過了單行80字符，使用 4 個空格縮進。描述應該包括所需的類型和含義。

:wrench: 4. docstrings 中 **Returns** ( 或者 **Yields** ) 的撰寫方式：
> - 描述返回值的類型和含義。如果函數返回None，這一部分可以省略。

:wrench: 5. docstrings 中 **Raises** 的撰寫方式：
> - 列出有關的所有異常。

**Docstrings 規範：Class**

:wrench: 6. 若 class 有公共屬性 ( Attributes )，則此 class 的 docstrings 中應該有個屬性( Attributes ) 段落。

> - Example
> ```python
> class SampleClass(object):
>     """Summary of class here.
> 
>     Longer class information....
>     Longer class information....
> 
>     Attributes:
>         likes_spam: A boolean indicating if we like SPAM or not.
>         eggs: An integer count of the eggs we have laid.
>     """
> 
>     def __init__(self, likes_spam=False):
>         """Inits SampleClass with blah."""
>         self.likes_spam = likes_spam
>         self.eggs = 0
> 
>     def public_method(self):
>         """Performs operation blah."""
> ```

**Block and Inline Comments(塊註釋和行註釋寫法)：**

 :pushpin: 7. 若在 code review 的時候必須解釋，則要寫上註釋。

:wrench: 8. 對於複雜的操作，要在其操作開始前寫上註釋。
> - Example
> ```python
> # We use a weighted dictionary search to find out where i is in
> # the array.  We extrapolate position based on the largest num
> # in the array and the array size and then do binary search to
> # get the exact number.
> ```
:wrench: 9. 對於不是一目瞭然的代碼，應在其行尾添加註釋。為了提高可讀性，註釋應該至少離開程式碼 8 個空格。

> - Example
> ```python
> if i & (i-1) == 0:        # true iff i is a power of 2
> ```

 :pushpin: 10. 絕對不要逐行解釋 code。( 要假設閱讀代碼的人比你更懂 python, 他只是不知道你的代碼要做什麼。 )

> - Example - Bad
> ```python
> # BAD COMMENT: Now go through the b array and make sure whenever i occurs
> # the next element is i+1
> ```

**TODO 規範**
說明：為臨時代碼使用 TODO 註釋，是一種短期解決方案。

:wrench: 11. TODO 註釋應該在所有開頭處包含"TODO"字串，接著用括號括起內容，包含名字、email、地址等等，然後是一個冒號。接著必須有一行註釋，解釋要做什麼。

> - Example
> ```python
> # TODO(kl@gmail.com): Use a "*" here for string repetition.
> # TODO(Zeke) Change this to use relations.
> ```

:wrench: 12. 若 TODO 是"將來做某事"的形式，那麼請確保包含了一個指定的日期("2018年11月解決")或者一個特定的事件("等到所有的客戶都可以處理XML請求就移除這些代碼")。


### <font color=#00AAAA> 七、 程式碼編寫原則

**可讀性**

:wrench: 1. 不要在行尾加分號，也不要用分號將兩條命令放在同一行。
 
:wrench: 2. 每行不超過 120 個字符。例外:
> - 導入module 語句
> - 註釋裡的 URL

:wrench: 3. 用 4 個空格來縮排代碼，不允許用 tab 替代。

 :pushpin: 4. Python 會將小括號、中括號和大括號中跨行的程式碼連接起來視為一個表達式。需要時，可以在跨行的表達式外圍增加一對額外的小括號來增加可讀性。
> - Example
> ```python
> foo_bar(self, width, height, color='black', design=None, x='foo',
>         emphasis=None, highlight=0)
> 
> if (width == 0 and height == 0 and
>     color == 'red' and emphasis == 'strong'):
> ```

:wrench: 5. 寧缺毋濫的使用括號。除非是為了連接跨行的程式碼，否則不要在返回語句或條件語句中使用括號。
> - Example - Good
> ```python
> if foo:
>     bar()
> while x:
>     x = bar()
> if x and y:
>     bar()
> if not x:
>     bar()
> return foo
> for (x, y) in dict.items(): ...
> ```
> - Example - Bad
> ```python
> if (x):
>     bar()
> if not(x):
>     bar()
> return (foo)
> ```

:wrench: 6. 連接跨行的程式碼時有以下兩種做法：
> (1) 垂直對齊換行的元素
>>
>> - Example - Good
>> ```python
>> # Aligned with opening delimiter
>> foo = long_function_name(var_one, var_two,
>>                          var_three, var_four)
>> ```
>>
>> - Example - Bad
>> ```python
>>  # Stuff on first line forbidden
>> foo = long_function_name(var_one, var_two,
>>     var_three, var_four)
>> ```
>
> (2) 使用4空格的縮進；此作法在第一行不應該有參數。
>> - Example - Good
>>
>> ```python
>> # Aligned with opening delimiter in a dictionary
>> foo = {
>>     long_dictionary_key: value1 +
>>                          value2,
>>            ...
>> }
>> 
>> # 4-space hanging indent; nothing on first line
>> foo = long_function_name(
>>     var_one, var_two, var_three,
>>     var_four)
>> 
>> # 4-space hanging indent in a dictionary
>> foo = {
>>     long_dictionary_key:
>>         long_dictionary_value,
>>            ...
>> }
>> ```
>> - Example - Bad
>> ```python
>>  # Stuff on first line forbidden
>> foo = long_function_name(var_one, var_two,
>>     var_three, var_four)
>> 
>> # 2-space hanging indent forbidden
>> foo = long_function_name(
>>   var_one, var_two, var_three,
>>   var_four)
>> 
>> # No hanging indent in a dictionary
>> foo = {
>>     long_dictionary_key:
>>     long_dictionary_value,
>>     ...
>> }
>> ```

**空格的使用**

:wrench: 7. 左右括號前後不要有空格。
> - Example - Good
> ```python
> spam(ham[1], {eggs: 2}, [])
> ```     
>
> - Example - Bad
> ```python
> spam( ham[ 1 ], { eggs: 2 }, [ ] )
> ``` 

:wrench: 8. 逗號、分號、冒號前不加空格，後加空格；但是在行尾時後不加空格。
> - Example - Good
> ```python
> if x == 4:
>     print x, y
> x, y = y, x
> ```
> - Example - Bad
> ```python
> if x == 4 :
>     print x , y
> x , y = y , x
> ```

:wrench: 9.  參數列表或索引的左括號前不應加空格。
> - Example - Good
> ```python
> spam(1)
> dict['key'] = list[index]
> ```
> - Example - Bad
> ```python
> spam (1)
> dict ['key'] = list [index]
> ```

:wrench: 10.  二元運算子兩邊都加上一個空格，比如=, (==, <, >, !=, <>, <=, >=, in, not in, is, is not), (and, or, not)。
> - Example - Good
> ```python
> x == 1
> ```
> - Example - Bad
> ```python
> x<1
> ```

:wrench: 11. 當 = 用於 default parameter values 時，不要在其兩側使用空格。
> - Example - Good
> ```python
> def complex(real, imag=0.0): return magic(r=real, i=imag)
> ```
> - Example - Bad
> ```python
> def complex(real, imag = 0.0): return magic(r = real, i = imag)
> ```

:wrench: 12.  不要用空格來垂直對齊多行間的標記，因為這會成為維護的負擔。
> - Example - Good
> ```python
> foo = 1000  # comment
> long_name = 2  # comment that should not be aligned
> 
> dictionary = {
>     "foo": 1,
>     "long_name": 2,
> }
> ```
> - Example - Bad
> ```python
> foo       = 1000  # comment
> long_name = 2     # comment that should not be aligned
> 
> dictionary = {
>     "foo"      : 1,
>     "long_name": 2,
> }
> ```

**空行**

:wrench: 13. top-level definitions 之間要空兩行。(例如：function 或 class definitions 之間)。

:wrench: 14. method definitions 之間要空一行。(像是method definitions and between the class line and the first method)。


**判斷布林值**

:wrench: 15. 不要用 `==` 或者 `!=` 來比較None，請使用 `is` 或者 `is not`。
(注意: 當寫下 `if x:` 時其實表示的是 `if x is not None`)

:wrench: 16. 禁止用 `==` 將一個布林值與 false 相比較。使用 `if not x:` 代替。當需要區分false和None，應該用像 `if not x and x is not None:` 這樣的語句。

:wrench: 17. 對於序列(字串，列表，元組)，要注意空序列是 false。因此 `if not seq:` 或者 `if seq:` 比 `if len(seq):` 或 `if not len(seq):` 要更好。

:wrench: 18. 處理整數時，使用隱式 false 可能會得不償失(即不小心將None當做0來處理)。 可以將一個已知是整數(且不是len()的返回結果)的值與0比較。

> - Example - Good
> ```python
> if not users:
>     print 'no users'
> 
> if foo == 0:
>     self.handle_zero()
> 
> if i % 10 == 0:
>     self.handle_multiple_of_ten()
> ```
> 
> - Example - Bad
> ```python
> if len(users) == 0:
>     print 'no users'
> 
> if foo is not None and not foo:
>     self.handle_zero()
> 
> if not i % 10:
>     self.handle_multiple_of_ten()
> ```

:pushpin: 19. 避免複合型的條件表達式Compound Conditional Expressions ，建議使用 Boolean Variables 分割成多個表達式較利於閱讀

> - Example - Good
> ```python
> num_1, num_2, num_3 = 5, 13, 4
> condiction_1 = num_1 > num2
> condiction_2 = num_1 > num3
> if condiction_1 and condiction_2:
>     print('num_1 is the max value')
> ```
>
> - Example - Bad
> ```python
> num_1, num_2, num_3 = 5, 13, 4
> if num_1 > num2 and cnum_1 > num3:
>     print('num_1 is the max value')
> ```

**導入(Import)**

:wrench: 20. `import` 只能用在 packages 跟 modules，不能使用在導入單一 classes 或 functions。

:wrench: 21. 所有 import 應該集中放在文件頂部，位於註釋和 docstring 之後。

:wrench: 22. 每個 import 應該獨佔一行。
> - Example - Good
> ```python
> import os
> import sys
> ```
> 
> - Example - Bad
> ```python
> import os, sys
> ```

:wrench: 23 導入 package 應該使用 package 的全路徑。

> - Example - Good
> ```python
> from sound.effects import echo
> ```

> - Example - Bad
> ```python
> import sound.effects.echo
> ```

:pushpin: 24. Import 的順序，相同的 packages 或是 modules 應放在一起，且按照從最通用到最不通用的順序分組:
> (1) 標準庫導入 (E.g. `import sys`)
> (2) 第三方庫導入 (E.g. `import tensorflow as tf`)
> (3) 應用程序指定導入 (E.g. `from other project`)

> - Example
> ```python
> import os
> import sys
> 
> from sklearn import preprocessing
> from sklearn import metrics
> import tensorflow as tf
> ```

:pushpin: 25.	每種分組中，應根據每個 module 的完整路徑按字母排序(忽略大小寫)。
```python
import foo
from foo import bar
from foo.bar import baz
from foo.bar import Quux
from Foob import ar
```

:wrench: 26. `from x import y` (e.g. `from sklearn import cross_validation`)。其中 x 是package 前綴，y 是不帶前綴的 module 名。

:wrench: 27. 以下任一種狀況下， import 時需要使用別名，且別名不可與原本的 package名稱相同：
> - 若有兩個 modules 都叫 y 的情況下
> - 或者是 y 名稱過長
> - import 的別名有標準縮寫時 (e.g. `import numpy as np`)

:wrench: 28. import 的別名有標準縮寫時，只能使用標準縮寫。
> ```python
> import numpy as np
> ```

**語句**

:wrench: 29. 通常每個語句應該單獨一行。但是，當測試結果與測試語句在同一行放得下，也可以放在同一行。若是 if 語句，只有在沒有 else 時才能這樣做。但是絕對不要對 try/except 這樣做，因為 try 和 except 不能放在同一行。
> - Example - Good
> ```python
> if foo: bar(foo)
> ```
> 
> - Example - Bad
> ```python
> if foo: bar(foo)
> else:   baz(foo)
> 
> try:               bar(foo)
> except ValueError: baz(foo)
> 
> try:
>     bar(foo)
> except ValueError: baz(foo)
> ```

:wrench: 30. mapping expression、for 、filter expression 等，每個部分應該單獨置於一行。禁止多重 for 語句或 filter expressions，複雜情況下使用 loops。
> - Example - Good
> ```python
> result = []
> for x in range(10):
>     for y in range(5):
>         if x * y > 10:
>             result.append((x, y))
> 
> for x in xrange(5):
>     for y in xrange(5):
>         if x != y:
>             for z in xrange(5):
>                 if y != z:
>                     yield (x, y, z)
> 
> return ((x, complicated_transform(x))
>         for x in long_generator_function(parameter)
>         if x is not None)
> 
> squares = [x * x for x in range(10)]
> 
> eat(jelly_bean for jelly_bean in jelly_beans
>       if jelly_bean.color == 'black')
> ```
> - Example - Bad
> ```python
> result = [(x, y) for x in range(10) for y in range(5) if x * y > 10]
> 
> return ((x, y, z)
>         for x in xrange(5)
>         for y in xrange(5)
>         if x != y
>         for z in xrange(5)
>         if y != z)
> ```

:pushpin: 31. 如果類型支持，就使用默認疊代器和運算子 (Default Iterators and Operators)。例如 lists，dictionaries 和 files。
> - Example - Good
> ```python
> for key in adict: ...
> if key not in adict: ...
> for line in afile: ...
> ```
> - Example - Bad
> ```python
> for key in adict.keys(): ...
> if not adict.has_key(key): ...
> for line in afile.readlines(): ...
> ```

**文件讀取**

:wrench: 32. 在文件和 sockets 結束時關閉它。原則上使用 "with"語句以管理文件:
> - Example
> ```python
> with open("hello.txt") as hello_file:
>     for line in hello_file:
>         print line
> ```

**其他**

:wrench: 33. 建立tuple 必須使用小括號：

> - Example - Good
> ```python
> x = (1, 2, 3)
> ```
> 
> - Example - Bad
> ```python
> x = 1, 2, 3
> ```

:wrench: 34. 縮排不得超過五層。

:wrench: 35. 裝飾器 (decorator) 應該遵守和函數一樣的導入和命名規則。裝飾器 (decorator) 的 python 文檔應該清晰的說明該函數是一個裝飾器 (decorator) 。


### <font color=#00AAAA> 八、 錯誤處理原則 </font>

:wrench: 1. 使用 `finally` 子句來執行那些無論 `try:` 塊中有沒有異常都應該被執行的代碼。
> - Example
> ```python
> file = open('demo.py', 'r', encoding='UTF-8')
> try:
>     for line in file:
>         print(line, end='')
> except:
>     print('讀取檔案發生錯誤')
> finally:
>     file.close()
> ```
> 

:wrench: 2.	module 或 package 應該定義自己的特定域的異常基類，這個基類應該從內建的Exception類繼承。module的異常基類應該叫做"Error"。
- Example
```python
class Error(Exception):
    pass
```

:pushpin: 3. 建議不要使用 `except:` 語句來捕獲所有異常，也不要捕獲 `Exception` 或者 `StandardErro`。使用`except:`很容易隱藏真正的 bug。

:pushpin: 4. 盡量減少 try/except 區塊中的代碼量。`try:` 區塊的體積越大，期望之外的異常就越容易被觸發。這種情況下，try/except 塊將隱藏真正的錯誤。


### <font color=#00AAAA> 九、 目錄架構原則 </font>

:wrench: 1.  AI研發雲檔案命名規範
> 
> | 資料夾命名                   |定義      |權限 | 備註 | 
> | --------------------------| ---------------   | ---------------   | ---------------  | 
> | /feature_engineering                   | 特徵工程  | 專案團隊 |  | 
> | /feature_engineering/dataset                   | 特徵工程資料集  | 專案團隊 | 此目錄下的資料不得上 git | 
> | /model                   | 模型/模型訓練的程式 |  專案團隊 |  | 
> | /utility                |工具包 (E.g. 放置團案團隊使用的method module )  |  所有人|   | 
> | /service                    | 服務 (for API 團隊或其他開發案)  |專案團隊 |                     |
> | /framework                  | 架構與環境變數 (for API 團隊或其他開發案)   |專案團隊 | |    
> | /log                  | 程式碼運行log記錄   |專案團隊 | 此目錄下的資料不得上 git |

:wrench: 2.  禁止將 dataset 放入TFS

:wrench: 3. 程式碼中不可 Hard Code 機密資訊(如資料庫連結字串、密碼或金鑰)，機密資訊應加密並儲存於設定檔案中


### <font color=#00AAAA> 十、 附件 </font>

:notebook: 參考 .pylintrc 中相關設定：
> | 項目                    | 上限       | 下限    |
> | ------------------ | -----------| -----------|
> | publish methods for a class | 20 | 0 | 
> | attributes for a class | 10 |  | 
> | arguments for a function / method |  10 |  | 
> | boolean expressions in if statement | 5 |  |
> | locals for a function / method  | 30 |  |
> | returns  | 6 |  |
> | statements in a function  | 100 |  |
> | lines in a module  | 1000 |  |
> | characters in a line  | 120 |  |