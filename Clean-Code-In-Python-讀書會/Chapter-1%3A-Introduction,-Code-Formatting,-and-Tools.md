<font color= black size=3 face="微軟正黑體" cellpadding=1000000  style="line-height: 1.5">

# Clean Code 
與以下項目息息相關
- 易讀性
- 容易維護
- 減少技術債
- 有效地溝通理念讓別人了解我們最初的想法用意

# 什麼是 PEP
Python Enhancement Proposals，用來規範與定義 Python 的各種加強與延伸功能的技術規格，好讓 Python 開發社群能有共同遵循的依據。

https://www.python.org/dev/peps/

![image.png](.attachments/image-74162b8e-fb0d-42cf-ba97-738d28d01f55.png)

# PEP-8 的好處
- 方便使用 grep

找參數 --> location=current_location
```
grep -nr "location="
```
找變數 --> current_location = get_location()
```
grep -nr "location ="
```
- 一致性：方便閱讀
-  程式品質：易於除錯

# PEP-8 風格指引
1. 縮排-使用4個空格
2. 將最高層的 function 和 class 定義以兩個空白行分隔
3. 在參數列表裏的 = 兩端不需要空格
4. Class 的 method 之間一個空行
```PYTHON
def add(num1=1, num2=2):
    return num1 + num2


class Mytest(object):
    def my_function1(self):
        return 1

    def my_function2(self):
        return 2
```


# 避免使用註解
- 代表程式無法表達想法
- 可能產生誤導，常會修改程式忘記修改註解


# 使用 docstring
- 文件化，必須要維護
- 盡可能加入 docstring
- 解釋資料型別、提供範例、註解變數


# 註釋
- 提示函式引數型別
- 但 python 是動態型態語言，不會強制限定型別
- 可使用 __annotations__ 來文件化、強制檢查型別
```
e.g.
def locate(latitude: float, longitude: float):

則 locate.__annotations__ 內有 dict
{'latitude': float, 'longitude': float}
```
 - 什麼是動態型態語言

C++
```
int a = 1, b = 2;
int sum=a+b;
string s = "I'm the example";
```
  Python
```
a = 1
b = 1
sum = a+b
s = "I'm the example"
```
