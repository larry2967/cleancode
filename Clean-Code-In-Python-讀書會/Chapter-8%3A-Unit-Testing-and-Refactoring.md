
## 「天下武功，唯快不破」

![image.png](.attachments/image-e208b1a9-d00c-49c2-a900-3e7be0059517.png)
現在的軟體開發中，速度永遠是最重要的。速度越快，越能快速反應市場，獲得價值。
在速度背後，反應的就是越早獲得回饋，問題越快發生，災害影響越小，越容易能改變。

假設今天有一個程式，遵守了『開閉原則』，程式碼簡潔，各種抽象化都做得很好，但我們仍然會有信心修改時不會造成問題? 功能正常? 上線沒有問題?

## 測試，讓問題提早出現
* 單元測試
  檢驗軟體基本組成單位的正確性，測試的物件是軟體設計的最小單位：函式。

* 整合測試
  程式模組採用適當的整合策略組裝起來，對系統的介面及整合後的功能進行正確性檢測的測試工作。

* 系統測試
   功能測試主要針對包括功能可用性、功能實現程度（功能流程&業務流程、資料處理&業務資料處理）方面測試。

* 驗收測試
  確認一系統是否符合設計規格或契約之需求內容的測試。

* 回歸測試
  檢驗軟體原有功能在修改後是否保持完整。

## 單元測試
程式的最基本模組: 函式，單元測試就是測試這些基本單位有沒有正確。

* 隔離性
   * 應與任何外部代理程式隔離，將焦點放在商業邏輯上。
   *  不連接資料庫，不執行 http 請求等
   * 測試順序要能任意執行，不依賴之前狀態。
* 效能
   * 要能快速執行，才能反覆測試
   * 越快獲得回應，開發迭代速度才能提升
   * 跑完全套單元測試不能超過10分鐘
* 自我驗證
   * 能正確的反應程式問題
   * 錯誤就是錯誤，不需要任何解釋

## Test-driven Development (TDD)
測試有分三種:
1. 不寫測試
2. 先寫程式再寫測試
3. 先寫測試再寫程式

TDD 就是使用第三種，先寫測試，再寫程式。

## TDD 流程

### 1. 寫測試
* 每當要開發新功能時，先把測試寫出。
* 寫測試過程中，思考與定義出功能的介面，輸入輸出是什麼?
* 設想要輸入的參數，預期的結果是什麼?
* 功能的程式此時都還未開始實作

### 2. 執行測試，取得失敗狀態
* 先跑一次測試，確認測試結果處於失敗狀態。
* 確保測試程式可以執行

### 3. 寫程式
* 先快速寫一版可以通過測試的程式
* 不求程式簡潔優雅，已通過為主

### 4. 執行測試
* 測試程式可以正確通過，確保程式邏輯無誤。
* 得到一版可以包含測試和正確的程式碼。

### 5. 重購
* 優化程式碼，包含產品程式和測試程式（測試程式也是專案需維護的一部份）。
* 提升程式的可讀性、可維護性、擴充性。
* 同時確保每次修改後，執行測試皆能通過。

每當有新功能要開發，重複以上流程。
確保每一階段的修改都是微小的改動，當有任一階段改動導致測試失敗，中斷開發並修復問題，確保問題解決再繼續進行下去。

![image.png](.attachments/image-244c09b8-5420-42d8-9e80-65ec4a3fa9cf.png)

## TDD 示範
撰寫簡單的版本控制工具，在有人提出合併請求時提供程式碼審查功能。
* 如果至少有一人反對，合併請求就會 rejected
* 如果沒有人反對，而且至少有兩位其他的開發者認可，就會 approved
* 在其他狀況下，狀態是 pending

### Step 1: 先寫測試

```
class TestMergeRequestStatus(unittest.TestCase):
    def test_simple_rejected(self):
        merge_request = MergeRequest()
        merge_request.downvote("maintainer")
        self.assertEqual(merge_request.status, MergeRequestStatus.REJECTED)
   
    def test_just_created_is_pending(self):
        self.assertEqual(MergeRequest().status, MergeRequestStatus.PENDING)

    def test_pending_awaiting_review(self):
        merge_request = MergeRequest()
        merge_request.upvote("core-dev")
        self.assertEqual(merge_request.status, MergeRequestStatus.PENDING)

    def test_approved(self):
        merge_request = MergeRequest()
        merge_request.upvote("dev1")
        merge_request.upvote("dev2")
        self.assertEqual(merge_request.status, MergeRequestStatus.APPROVED)
```

### Step 2: 跑測試
```
EEEE
======================================================================
ERROR: test_approved (__main__.TestMergeRequestStatus)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_merge.py", line 18, in test_approved
    merge_request = MergeRequest()
NameError: name 'MergeRequest' is not defined

======================================================================
ERROR: test_just_created_is_pending (__main__.TestMergeRequestStatus)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_merge.py", line 10, in test_just_created_is_pending
    self.assertEqual(MergeRequest().status, MergeRequestStatus.PENDING)
NameError: name 'MergeRequest' is not defined

======================================================================
ERROR: test_pending_awaiting_review (__main__.TestMergeRequestStatus)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_merge.py", line 13, in test_pending_awaiting_review
    merge_request = MergeRequest()
NameError: name 'MergeRequest' is not defined

======================================================================
ERROR: test_simple_rejected (__main__.TestMergeRequestStatus)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_merge.py", line 5, in test_simple_rejected
    merge_request = MergeRequest()
NameError: name 'MergeRequest' is not defined

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (errors=4)
```

### Step 3: 寫功能

```
from enum import Enum

class MergeRequestStatus(Enum):
    APPROVED = "approved"
    REJECTED = "rejected"
    PENDING = "pending"
    
class MergeRequest:
    def __init__(self):
        self._context = {
            "upvotes": set(),
            "downvotes": set(),
        }

    @property
    def status(self):
        if self._context["downvotes"]:
            return MergeRequestStatus.REJECTED
        elif len(self._context["upvotes"]) >= 2:
            return MergeRequestStatus.APPROVED
        
        return MergeRequestStatus.PENDING
    
    def upvote(self, by_user):
        self._context["downvotes"].discard(by_user)
        self._context["upvotes"].add(by_user)

    def downvote(self, by_user):
        self._context["upvotes"].discard(by_user)
        self._context["downvotes"].add(by_user)
```

### Step 4: 測試功能正常
```
....
----------------------------------------------------------------------
Ran 4 tests in 0.000s

OK
```

## 雙倍的開發時間，一樣的工作成果，不一樣的程式品質
* CI/CD 自動化的第一步
* 品質提升，技術債降低
* 短期工時增加，長期維護工時降低
* 寫測試的過程，就會思考介面要怎樣設計，然後思考程式的隔離性、獨立性，自然而然寫出高品質的程式。

~~潛伏三年，為了下一個十年~~