---
name: to-tickets
description: 把一份計畫、spec 或目前的對話拆成一組曳光彈式的 ticket，每張都宣告它的阻擋關係，並發布到設定好的 tracker（在本地時，每張 ticket 一個檔案，以文字寫出阻擋關係；在真正的 tracker 上則使用原生的阻擋連結）。
disable-model-invocation: true
---

# 轉成 Ticket（To Tickets）

把一份計畫、spec 或對話拆成一組 **ticket**：曳光彈（tracer-bullet）式的垂直切片，每一張都宣告**阻擋（block）**它的 ticket。

issue tracker 與分類標籤詞彙應該已經提供給你了。如果沒有，請告訴使用者執行 `/setup-matt-pocock-skills`。

## 流程

### 1. 蒐集背景

從對話情境中已有的內容著手。如果使用者以參數傳入一個引用（spec 路徑、issue 編號或 URL），就取得它，並閱讀完整的內文與留言。

### 2. 探索程式碼庫（選用）

如果你還沒有探索過程式碼庫，就去探索，以了解程式碼的現況。Ticket 的標題與描述應該使用專案領域詞彙表的用語，並尊重你所觸及領域中的 ADR。

尋找預先重構（prefactor）程式碼的機會，讓實作變得更容易。「先讓改變變得容易，再做那個容易的改變。」

### 3. 草擬垂直切片

把工作拆成**曳光彈** ticket。

<vertical-slice-rules>

- 每個切片切出一條窄但「完整」的路徑，穿過每一層（schema、API、UI、測試）：是垂直的，「不是」某一層的水平切片
- 完成的切片本身就能展示或驗證
- 每個切片的大小都要能放進一個全新的 context window
- 任何預先重構都應該最先做

</vertical-slice-rules>

為每張 ticket 標出它的**阻擋關係**：在它能開始之前必須先完成的其他 ticket。沒有阻擋者的 ticket 可以立即開始。

**大範圍重構是垂直切片的例外。** **大範圍重構（wide refactor）**是指一個機械性的變更（改一個欄位名稱、改變一個共用符號的型別），它的**影響範圍（blast radius）**擴散到整個程式碼庫，所以一次編輯就會同時弄壞數千個呼叫處，沒有任何垂直切片能以綠燈落地。不要硬把它塞進曳光彈；改用**擴張再收縮（expand–contract）**的順序。先擴張：在舊形式旁邊加入新形式，讓什麼都不會壞。接著依影響範圍決定批次大小（每個套件、每個目錄），分批把呼叫處遷移過去，每一批都是自己的 ticket，並被「擴張」所阻擋，因為舊形式仍然存在，所以每一批之間 CI 都能保持綠燈。最後收縮：在沒有任何呼叫者之後刪除舊形式，這張 ticket 被每一個遷移批次所阻擋。當連批次都無法單獨保持綠燈時，保留這個順序，但讓它們共用一個整合分支，並全部阻擋一張最後的「整合並驗證」ticket；只有在那裡才保證綠燈。

### 4. 向使用者提問確認

以編號清單呈現提議的拆分方式。每張 ticket 顯示：

- **標題**：簡短的描述性名稱
- **被誰阻擋**：哪些其他 ticket（如果有的話）必須先完成
- **交付什麼**：這張 ticket 讓哪個端到端的行為能運作

詢問使用者：

- 粒度感覺對嗎？（太粗 / 太細）
- 阻擋關係正確嗎：每張 ticket 是否只依賴真正卡住它的 ticket？
- 有沒有 ticket 應該合併或進一步拆分？

反覆修改，直到使用者核准這個拆分方式。

### 5. 把 ticket 發布到設定好的 tracker

發布已核准的 ticket。**怎麼發布**取決於 `/setup-matt-pocock-skills` 設定的 tracker；兩種情況下 ticket 本身都一樣，只有阻擋關係的形式不同：

- **本地檔案** → 每張 ticket 寫一個檔案，放在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，依相依順序從 `01` 開始編號（阻擋者在前）。每個檔案的「Blocked by」列出它所依賴的編號或標題。使用下方每張 ticket 一個檔案的範本：一個檔案一張 ticket，絕不合併成單一個檔案。
- **真正的 issue tracker（GitHub、Linear……）** → 依相依順序（阻擋者在前）每張 ticket 發布一個 issue，讓每張 ticket 的阻擋關係都能引用真實的識別碼。如果平台有原生的阻擋關係就使用它；否則把每張 ticket 的「Blocked by」設為阻擋它的 issue。如果來源是一個既有的 issue，就把每張 ticket 設為它的子 issue（依 tracker 文件中的操作方式）。除非另有指示，否則套用 `ready-for-agent` 分類標籤；這些 ticket 在構造上就是 agent 可以直接接手的。

處理**前緣（frontier）**：所有阻擋者都已完成的 ticket。對於純線性的鏈，就是由上到下。

*不要*關閉或修改任何父 issue。

<local-ticket-template>

# <NN>: <Ticket 標題>

**What to build（要建立什麼）：** 這張 ticket 讓哪個端到端的行為能運作，從使用者的角度描述，而不是一層一層的實作清單。

**Blocked by（被誰阻擋）：** 卡住這張 ticket 的 ticket 編號或標題，或「None (can start immediately)（無，可立即開始）」。

**Status（狀態）：** ready-for-agent

- [ ] 驗收條件 1
- [ ] 驗收條件 2

</local-ticket-template>

<issue-template>

## Parent（父 issue）

指向 tracker 上父 issue 的引用（如果來源是既有的 issue；否則省略這一節）。

## What to build（要建立什麼）

這張 ticket 讓哪個端到端的行為能運作，從使用者的角度描述，而不是一層一層的實作。

## Acceptance criteria（驗收條件）

- [ ] 條件 1
- [ ] 條件 2

## Blocked by（被誰阻擋）

- 指向每張阻擋 ticket 的引用，或「None (can start immediately)（無，可立即開始）」。如果阻擋者已設定為原生的關係，就省略這一節。

</issue-template>

不論哪種形式，都避免具體的檔案路徑或程式碼片段：它們很快就會過時。例外：如果某個原型產出了一段比文字更精確表達某個決策的程式碼（狀態機、reducer、schema、型別形狀），就把它內嵌進來，並簡短註明它來自原型。只保留富含決策的部分，不是一個可運作的 demo，只要重要的部分。
