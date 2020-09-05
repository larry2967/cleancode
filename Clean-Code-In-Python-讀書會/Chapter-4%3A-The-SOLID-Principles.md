## 單一功能原則 (Single Responsibility Principle, SRP)

<font color='blue'>你是否有遇過...?</font>
1. 某些程式越來越大、邏輯越來越複、越來越難維護它
2. 想繼承其他人寫好的程式，但又有許多功能用不到的，可能會影響程式效能，我要不要自已寫一支新的

我們有時會聽到其他人: 誰啦，這誰寫的，在寫什麼啦，我要先下班了

<font color='red'> 準則：</font>
一個 Class 只負責一個功能，只有一種原因引起 Class 的改變。 如果我們將不同的功能聚在一起，會使我們難以維護它。

<font color='blue'> 效益是什麼? </font>
1. 程式簡潔，容易維運
2. 程式效能比較好，因為減少不必要的程式

情境: Claas A 負責功能 1和功能 2，在正常運行中沒有出現任何問題，但當功能 1改變時，需要修改 Claas A ，可能引起正常運行的功能 2發生問題

解決方案：遵循單一功能原則，分別創建Claas A和Claas B來負責功能 1和功能 2。這樣設計當修改其一，並不會影響其二工作

 * **優化前**
 ```python
class Animal(object):
    def __init__(self):
        pass
 
    def Terrestrial(self, name):
        print('%s 陸地動物' %name)
 
    def Aquatic(self, name):
        print('%s 水中動物' %name)
 
# caller
class Client(object):
    def __init__(self):
        pass
 
    def work(self):
        animal = Animal()
        animal.Terrestrial('牛')
        animal.Terrestrial('羊')
        animal.Terrestrial('猪')
        animal.Aquatic('鱼')
 
if __name__ == '__main__':
    client = Client()
    client.work()
```
執行結果
 ```python
1.	牛 陸地動物
2.	羊 陸地動物
3.	豬 陸地動物
4.	鱼 水中動物
```
但如果又增加了一種動物-----------鳥，很明顯鳥不是陸地動物與水中動物，因此要對類Animal進行修改

**優化後: 直接把 Animal 類別拆分，拆分為三個類分別實現陸地動物、水中動物、飛行動物.**
 ```python
class Terrestrial(object):

     def __init__(self):
        pass
 
    def breathe(self, name):
        print('%s 陸地動物' %name)
 
class Aquatic(object):

     def __init__(self):
        pass
 
    def breathe(self, name):
        print('%s 水中動物' %name)

class Flight(object):

     def __init__(self):
        pass
 
    def breathe(self, name):
        print('%s 飛行動物' %name)
 
# caller
class Client(object):
    def __init__(self):
        pass
 
    def work(self):
        Terrestrial= Terrestrial()
        Terrestrial.breathe('牛')
        Terrestrial.breathe('羊')
        Terrestrial.breathe('猪')
 
        aquatic = Aquatic()
        aquatic.breathe('鱼')

        flight= Flight()
        flight.breathe('鳥')
 
if __name__ == '__main__':
    client = Client()
    client.work()
```
執行結果
 ```python
1.	牛 陸地動物
2.	羊 陸地動物
3.	豬 陸地動物
4.	鱼 水中動物
5.	鳥 飛行動物
```

## 關閉原則 (The Open/Closed principle, OCP)

<font color='blue'>你是否有遇過...?</font>
1. 它之前明明是好的，我沒有修改那段程式邏輯，為什麼上線後，它發生異常
2. 這支程式大家很夯，很熱門，大家一直 check out 它

**<font color='red'> 準則：</font>**
理想情況下，當需求改變時，我們只要用新行為來擴展模組以滿足新的需求，但不用修改現有程式 

<font color='blue'> 效益是什麼? </font>
這解決了軟體工程的一個重大目標: 可維護性。不遵守 OCP 原則會造成連鎖反應，也有可能修改一個地方就會觸發整個程式的變動或損壞其他部份

簡單來說，當對原有程式進行擴展時，在不更動到原始碼的前提完成擴展。當我們加入新程式時也修改了既有程式，這就代表那個邏輯是不良的設計。理想情況下，當需求改變時，我們只想要用新行為來擴展模組以滿足新的需求，但不想要修改程式。

舉例：

 * **優化前**
 ```python
class Triangle(object):
    
    @staticmethod
    def Show():
        print ("這是一個 Triangle")

class Square(object):
    
    @staticmethod
    def Show():
        print ("這是一個 Square")

class Display(object):

    def ShowTheSharp(self,sharp):
        if sharp == 'Triangle':
            triangle=Triangle()
            triangle.Show()
        elif sharp == 'Square':
            square=Square()
            square.Show()

display=Display()
```

 ```python
class Triangle(object):
    
    @staticmethod
    def Show():
        print ("這是一個 Triangle")

class Square(object):
    
    @staticmethod
    def Show():
        print ("這是一個 Square")

class Circle(object):
    
    @staticmethod
    def Show():
        print ("這是一個 Circle")

class Display(object):

    def ShowTheSharp(self,sharp):
        if sharp == 'Triangle':
            triangle=Triangle()
            triangle.Show()
        elif sharp == 'Square':
            square=Square()
            square.Show()
        elif sharp == 'Circle':
            circle=Circle()
            circle.Show()

display=Display()
display.ShowTheSharp("Circle")
```
上面的例子不符合關閉原則，也不符合單一功能原則。如果想增加功能，類別必須被修改，這就違法了關閉原則。同時類別功能過多，也不符合單一功能原則。

實現符合關閉原則的代碼

 ```python
import abc

class Sharp(abc.ABC):
    @abc.abstractmethod
    def show(self):
        pass

class Triangle(Sharp):
    def show(self):
        print('this is a triangle')

class Square(Sharp):
    def show(self):
        print('this is a square')

class Circle(Sharp):
    def show(self):
        print('this is a circle')

class Display:
    def __init__(self):
        self.sharp = None

    def set_sharp(self, sharp):
        self.sharp = sharp

    def show_the_sharp(self):
        self.sharp.show()

display = Display()
display.set_sharp(Square())
display.show_the_sharp()
```

這樣就符合了關閉原則原則。同時增加功能只需增加就行了。避免了通過修改現有來增加功能

## 里氏替換原則 (Liskov Substitution Principle, LSP)
- Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T. 
---
**概述**
- 在繼承方面，當子類別替換替換掉父類別時，其功能不受影響
```python
class Calculator():
    def calculate(self, a, b): # returns a number
        return a * b

class DividerCalculator(Calculator):
    def calculate(self, a, b): # returns a number or raises an Error
        return a / b           

calculation_results = [
    Calculator().calculate(3, 4),
    Calculator().calculate(5, 7),
    DividerCalculator().calculate(3, 4),
    DividerCalculator().calculate(5, 0) # 0 will cause an Error, 但是TEST CASE就會救了你......嗎?
]

print(calculation_results)
```
---
**為什麼要遵循**
- 繼承的特性導致高耦合，子類別對於方法修改(Override, Overload) 必須依照父類別行為方向，否則會對整體的繼承體系照成破壞，會有產生不可預測的行為與不好察覺 Bug
- 如果遵循後可以減少Reuse上的Bug並且擴展子類別以達到新增功能，對於 開放封閉原則 Open-Closed Principle 來說可以安全的新增子類別
---
**如何降低違反LSP情形**
- 合約設計 DBC (Design By Contract) : 利用前置條件與後置條件來約束子類別，確保本質不變
- 避免繼承 : 盡量利用組合(Composition over inheritance)
---
**如何規範地遵從里氏替換原則**
- 子類必須完全實現父類的抽象方法，但不能覆蓋父類的非抽象方
- 子類可以實現自己特有的方法
- 當子類覆蓋或實現父類的方法時，方法的前置條件（即方法的形參）要比父類方法的輸入參數更寬鬆。
- 當子類的方法實現父類的抽象方法時，方法的後置條件（即方法的返回值）要比父類更嚴格。
- 子類的實例可以替代任何父類的實例，但反之不成立
- ...
```python

import abc
 
class Abstract_bird(object):
 
    __metaclass__ = abc.ABCMeta
 
    @abc.abstractmethod
    def fly(self):
        pass
 
    @abc.abstractmethod
    def tweet(self):
        pass
 
    def eat(self):
        print('bird is eating')
 
 
class Eagle(Abstract_bird):
 
    def fly(self):
        print('eagle is flying')
 
    def tweet(self):
        print('eagle is tweeting')
 
    def drink(self):
        print('eagle is drinking')
 
 
if __name__ == '__main__':
    bird = Abstract_bird()
    bird.fly()
    bird.tweet()
    bird.eat()
    print('把所有父類Abstract_bird替換為子類Eagle如下')
    eagle = Eagle()
    eagle.fly()
    eagle.tweet()
```
---
**善用工具檢查**
- mypy
```python
class Event:
    def meets_condition(self, event_data: dict) -> bool:
        return False

class LoginEvent(Event):
    def meets_condition(self, event_data: list) -> bool:#怎麼偷改input的屬性呢, 有沒有思考根本問題?
        return False
```
lsp_mypy.py:7: error: Argument 1 of "meets_condition" is incompatible with supertype "Event"; supertype defines the argument type as "Dict[Any, Any]"
- pylint
```python
class Event:
    def meets_condition(self, event_data: dict) -> bool:
        return False

class LoginEvent(Event):
    def meets_condition(self, event_data: dict, override: bool) -> bool:#怎麼新增參數呢, 有沒有思考根本問題?
        return False
```
lsp_pylint.py:6:4: W0221: Parameters differ from overridden 'meets_condition' method (arguments-differ)

---
**請待在safe的區域**
- safe
1. Composition over inheritance
- unsafe
1. Design By Contract
1. Pylint and Mypy
1. Code Review
1. Test Case and Test Case
![1528970642055.png](.attachments/1528970642055-a01d6aa6-3032-4ad8-ace5-56580ff1542a.png)
---
**最後...**
- 領養代替購買
- 新增代替修改
- 組合代替繼承
Prefer composition over inheritance as it is more malleable / easy to modify later, but do not use a compose-always approach.
- 拱手代替握手
---
**不斷的思考**
當你想讓B繼承A的時候 先問自己每個B都確實是A嗎 如果這個問題是不確定的 那就不應該用繼承 應該用復合
即使每個B都確實是A 再問最後一個問題 A是不是沒有缺陷的一個類 如果A有缺陷的話 你願不願意讓你的B有同樣的缺陷 因為繼承會把所有的缺陷都傳播到子類上 但復合可以允許設計新的API來隱藏這些缺陷
![ilxzO.jpg](.attachments/ilxzO-e81be7e6-5940-41ae-a75e-19ca842a229c.jpg)


## 介面隔離原則 (Interface Segregation Principle, ISP)

<font color='blue'>你是否有遇過...?</font>
1. 繼承後，空實作
2. 並非全部方法使用到

**<font color='red'>準則：介面(Interface)必須是小型的。使用者不應該被強迫依賴不使用的方法</font>**

<font color='blue'>介面(Interface)是什麼?</font>

1. 介面代表物件公開的一組方法，是以它可以接收或解釋的所有訊息構成它的介面
3. 介面就是使用方可以請求的東西
4. 介面可將類別公開的行為的定義與實作隔開。

在 Python 中，介面是透過`類別`與`方法`來定義的。
```python
class Person(object):

    @abc.abstractmethod
    def duty(self):
        """ 職責 """
        pass


class Coder(Person):

    def duty(self):
        """ Coder 的職責是... """
        return "Coding"

    def sleep(self):
        time.sleep(5)
```

**<font color='red'>本小結重點！</font>**
當我們定義一個有許多方法的介面時，最好拆成多個介面，讓每一個介面有較少的方法(最好只有一個)，並擁有一個非常明確且準確的範圍。

藉由將介面拆成最小的單位，可以提升程式重用性，想要實作其中一個介面的類別都極可能具備高度的內聚力，因為他有相當明確的行為與功能。



<font color='blue'>範例</font>

假設有些動物會走、會飛、會跳。
```python
class Animal:
  def walk(self):
    raise NotImplementError
  def fly(self):
    raise NotImplementError
  def jump(self):
    raise NotImplementError
```
用 raise NotImplementError，可以迫使每個實作 Animal 介面的 class 們，都必須要實作 method 才可以，不然會有 NotImplementError
```python
class Dock(Animal):
  def walk(self):
    pass
  def fly(self):
    pass
  def jump(self):
    pass

class Dog(Animal):
  def walk(self):
    pass
  def fly(self):
    pass
  def jump(self):
    pass
...
```
可是!!並不是每個動物都一定同時會走、會飛又會跳啊，但是我又想要強迫子類一定要實作這些 method，該怎麼辦呢?
還有一種情況，就算是每一種動物都同時會走、會飛又會跳，現在假設已經有十種動物都符合這條件，如果突然又發現，原來他們不是同時都會走、會飛又會跳，甚至還會游泳，那又要再 Animal 新增一個 def swim(self)，其他已經有實作 Animal 的子類，也要再實作 swim() 才可以。現在只有十種需要個別實作 swim()。

```python
class Animal:
  def walk(self):
    raise NotImplementError
  def fly(self):
    raise NotImplementError
  def jump(self):
    raise NotImplementError
  # 新增游泳功能
  def swim(self):
    raise NotImplementError
```
```python
class Dock(Animal):
  def walk(self):
    pass
  def fly(self):
    pass
  def jump(self):
    pass
  # 每個子類都要跟著實作
  def swim(self):
    pass

class Dog(Animal):
  def walk(self):
    pass
  def fly(self):
    pass
  def jump(self):
    pass
  def swim(self):
    pass
...
```
==那麼解法呢 !!==
```python
class Animal():
  def behavior(self):
    pass

class Dock(Animal):
  def behavior(self):
    pass

class Dog(Animal):
  def behavior(self):
    pass
```
其實就是把原本定得很細的 method，給它再抽象一層，不要把功能給寫死，實際作的事就交給實作的 class 去決定就好。

<font color='blue'>ISP的精神</font>
1. 不要讓介面包山包海
2. 避免提供過多功能的介面
3. 不應該強迫實作沒用到的方法
4. 沒用到的方法，另外建一個介面
5. 比較好的作法是每一個介面有一個方法，


<font color='blue'>介面應該要多小？</font>

還是要看使用情境啦！可以從內聚性的角度來理解
 

## 依賴反轉 (Dependency Inversion Principle, DIP)

依賴反轉原則，什麼是依賴(Dependency)？又要反轉？你可以把`依賴`想像成，我每天早上都要喝一杯咖啡，那麼就是我`依賴`咖啡。
那麼用程式的邏輯來解釋依賴，就是有一個 Coffee Class 和 Person class，來看範例。
```python
class Coffee:
  def __init__(self, name):
    self.name = name
    
class Person:
  def behavior_in_morning(self, coffee):
    print("Drink", coffee.name, ".")

Person().behavior_in_morning(Coffee("City Cafe"))
# 輸出為 Drink City Cafe .
```

這樣瞭解了依賴的關係了嗎 ? 其實再講得更簡單點，可以把`依賴`想像成 在你的 method 裡有用到某個物件，就是一種依賴，不論是取得物件的屬性、修改物件的屬性，還是執行物件的功能等等，都是一種依賴，還有繼承與實作也是一種依賴。

那麼今天有另一個人它習慣早上喝豆漿呢 ? 我這個 Person class 就必須得修改，這樣就違反了 OCP，所以這樣不是個好設計。

```python
class Coffee:
  def __init__(self, name):
    self.name = name

class SoyMilk:
  def __init__(self, name):
    self.name = name
    
class Person:
  def behavior_in_morning(self, coffee):
    print("Drink", coffee.name, ".")

Person().behavior_in_morning(Coffee("City Cafe"))
# 輸出為 Drink City Cafe .
```

==那麼解法呢 !!==
可以再抽象一層 Drink class，然而 Person 的 behavior_in_morning 的參數改成 Drink，這樣身為一個人，就可以自由地選擇他早上想喝的飲料了

```python
class Drink:
    def __init__(self, name):
        self.name = name

class SoyMilk(Drink):
    def __init__(self, name):
        self.name = name

class Coffee(Drink):
    def __init__(self, name):
        self.name = name

class Person:
    def behavior_in_morning(self, drink):
        print("Drink", drink.name, ".")

Person().behavior_in_morning(SoyMilk("義美"))
# 輸出為 Drink 義美 .

Person().behavior_in_morning(Coffee("City Coffee"))
# 輸出為 Drink City Coffee .
```

講那麼多，到底依賴反轉是啥，原文寫得很繞口，翻譯也很繞口= =

原文:

Dependency should be on abstractions not concretions

A. High-level modules
should not depend upon low-level modules. Both should depend upon abstractions.

B. Abstractions should not depend on details. Details should depend upon
abstractions.

翻譯
高階不依賴低階，高階與低階都應該依賴抽象，抽象不依賴細節，細節應該依賴抽象

**白話文**
在需要有彈性的地方，不應該直接定死細節，而是需要再抽象一層
就像不論是咖啡或是豆漿，再抽象一層就是喝東西

**再一個白話文例子**
假設有台電腦，它包含了CPU、硬碟、主機板、記憶體、電源等，當電腦的任何一部份壞掉、或是要更新的時候，我們直接買新的重新接上就可以了，不需要做其他事．
為啥！
因為他的設計中有考量到DIP
如果電腦直接依賴CPU(想像是黏死上去的)，那CPU壞了整台就要報廢，其他零件也是，電腦依賴CPU、電腦依賴主機板...等
但他試圖讓電腦依賴`接口這個抽象`(CPU插槽、記憶體插槽等等)，電腦只需要依賴抽象類接口，我們可以隨時替換接口的功能

不過也不要無限地抽象，就會達到 OverDesign，你必須自己考量**擴充彈性**與**開發效益**，來決定你怎麼設計這支程式。

## 結論
SOLID設計原則，心法啦！
還是要多寫程式，遇到不同的使用場景和使用方式，就能夠多加體會了ㄅ
