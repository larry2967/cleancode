# Mypy
- 是 Python 靜態型態檢查的工具
- 可以檢查型態的使用是否有不一致的狀況
- 安裝方式
```
pip install mypy
```

- 現有一個 test.py
```PYTHON
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return '{name} {age}'.format(name=self.name, age=self.age)


student1 = Student('小玉', 20)
student2 = Student('大山', '21')

print(student1)
print(student2)
```
- 執行方式
```
$ mypy test.py
```
- 執行結果
發現沒有問題！
![mypy01.PNG](.attachments/mypy01-23bb6bfd-9478-45e4-bbf0-cbcfa5e32f92.PNG)

- 加上類型標記後
```
class Student:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

    def __str__(self):
        return '{name} {age}'.format(name=self.name, age=self.age)


student1 = Student('小玉', 20)
student2 = Student('大山', '21')
```
- 執行結果
發現有一個輸入參數的類型不符合原先的設定！
![mypy02.PNG](.attachments/mypy02-33909c3e-129e-4dfa-879f-a2fb659a7bca.PNG)

- 幫助訊息
使用以下語法能查詢定義檢查方法的參數，例如使用 --ignore-missing-imports 來忽略 import 的錯誤。
```
mypy -h
```

# Pylint
- 可用來檢查整份 py檔的架構
- 檢測是否有符合 PEP-8 等 Coding Style，並給予一個分數。
- 執行方式
```
$ pylint test.py
```
- 執行結果(僅有-1.22分！)
![pylint.png](.attachments/pylint-bf81c40d-532b-4141-9645-0ff4aa884121.png)

- Black
由作業"apply_check_pack.py"之範例可發現有許多空格、縮排之問題，因此可先使用"black"工具自動處理上述問題。
black 可自動修改 py檔排版及並使其符合 PEP-8 之 Coding Style，執行範例如下：
```
$ black apply_check_pack.py
All done! ✨ 🍰 ✨
1 file reformatted, 0 files left unchanged.
```
使用black修正檔案之格式問題後，再用 pylint 偵測一次程式，可得出以下結果：
![pylint.png](.attachments/pylint-ccb6e37e-100b-4881-9127-7ccdfaf3f486.png)
改善大部分格式及縮排問題，分數提升為3.88分！

# makefile
使用 makefile 來完成自動檢測整合，步驟如下：
1. 建立檔案 makefile，內容如下： ( 使用 Tab，而非空格 )
```
typehint:
    mypy src/ tests/

pytest:
    pytest tests/

lint:
    - pylint src/ tests/

checklist: lint typehint pytest

.PHONY: typehint pytest lint checklist
```
2. 使用以下語法，執行 makefile 腳本
```
make checklist
```
3. 結果如下：
![image.png](.attachments/image-5e126f2d-1950-47c2-8b6a-667eaec64213.png)