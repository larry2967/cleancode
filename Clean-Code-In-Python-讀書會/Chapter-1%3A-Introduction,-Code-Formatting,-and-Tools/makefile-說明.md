# 說明
- make 是一個工具程式，經由讀取一個叫做 makefile 的檔案，自動化建構軟體。
- 編譯小型程式可用簡單的命令編譯或 shell script 編譯，但當程式很大且包含大量標頭檔和函式庫時，就需要使用 makefile。
- makefile 會將程式分成好幾個模組，根據裡面的目標、規則和檔案的修改時間進行判斷哪些需要重新編譯，省去大量重複編譯的時間。


# Makefile 裡主要包含了五個東西：
- 顯式規則：顯式規則表示如何生成一個或多個目標文件。
- 隱式規則：比較簡略地書寫 Makefile 規則
- 變量定義：定義的變數都會置換到引用位置上。
- 文件指示：一個 Makefile 中引用另一個 Makefile
- 註釋：用 # 符號；換行則是使用 \ 符號 。

# 顯式規則
- 格式
```
target: dependencies
    命令1
    命令2
    命令3
      .
      .
      .
    命令n
```
- 目標 (Target) : 可以是 Object 檔，也可以是執行檔，還可以是一個標籤。
- 依賴 (Dependency, Prerequisites) : 要產生目標檔 (target) 所依賴的檔案。
- 命令 (Command) : 建立專案時需要執行的 shell 命令。( 命令前使用 Tab 而非多個空格 )

# 變量
```
objects = program.o foo.o utils.o
program : $(objects)
    cc -o program $(objects)
$(objects) : defs.h
```
- 使用 = 給予初始值
- 使用 (變數) 或 {變數} 調用

# 使用 .PHONY
- .PHONY 會將目標設成偽目標，使 make 目錄下沒有目標檔案或目標檔案為最新時，仍可執行 make <target>。
- .PHONY 寫法也可以讓程式設計師知道哪些工作目標不是針對檔案，增加可讀性。