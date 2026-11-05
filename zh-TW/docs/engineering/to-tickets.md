## 它做什麼

`to-tickets` 接收一份計畫、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，或你所在的這段對話，並把它拆成你 issue tracker 上的一組 **[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)**。每張 ticket 都宣告它的**阻擋關係（blocking edges）**：在它能開始之前必須先完成的其他 ticket。

每張 ticket 都是一顆**曳光彈（tracer bullet）**：一條窄但完整、穿過變更每一層（schema、API、UI、測試）的路徑，只要一落地，你就能單獨展示它。這就是這個 skill 與「顯而易見的拆分方式」的差別，後者是一次切一層、最後再整合。它也把每張 ticket 的大小調整到能放進一個全新的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)，因為接手這張 ticket 的會是一個從未看過你 spec 的 [session](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何時使用

你透過輸入 `/to-tickets` 來觸發它。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。

| 你所在的位置 | 要執行什麼 |
| --- | --- |
| 你有一個 spec issue，而建置會橫跨好幾個 session | `/to-tickets`，或 `/to-tickets #<spec_issue>` |
| 計畫只存在於對話中，從沒寫下來 | `/to-tickets` 會直接讀取討論串，不需要 spec |
| 整個變更放得進一個 context window | [implement](https://aihero.dev/skills-implement)，跳過 ticket |
| 什麼都還沒決定 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然後 [to-spec](https://aihero.dev/skills-to-spec) |
| 一張 [wayfinder](https://aihero.dev/skills-wayfinder) 地圖已經清楚了 | 先用 [to-spec](https://aihero.dev/skills-to-spec) 收斂地圖，再用 `/to-tickets` |

`to-tickets` 產出的 ticket 在構造上就是 agent 可以直接處理的。不要對它們執行 [triage](https://aihero.dev/skills-triage)。分類是給從別人那裡來的工作用的。

## 前置條件

`to-tickets` 會發布到一個 tracker，所以必須先用 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 為這個 repo 設定好 tracker，以及分類標籤詞彙。兩種都可以：像 GitHub 或 Linear 這樣真正的 tracker，或是 `.scratch/` 下的本地 markdown 檔案，後者不需要額外設定。

## 曳光彈，而不是分層

**水平**切片交付的是變更的一層。在每一層都落地之前什麼都無法運作，而每張 ticket 的驗收條件都必須伸手觸及另一張 ticket 所擁有的工作。**垂直**切片（曳光彈）一次交付一條穿過所有層的細路徑，所以它可以單獨驗證，也擁有它所評分的一切。

這是人們最常違反的規則。一個團隊執行了一疊依層切分（語料庫、生產者、聚合器、選擇器）的 26 張 ticket，結果每關閉一張 ticket 大約要跑二十次 agent，其中約四分之三是重做。他們自己的事後檢討把每一類失敗都追溯到水平切片，而不是實作本身。

在任何東西發布之前，會發生兩件事。`to-tickets` 會尋找預先重構（prefactoring）的機會（「先讓改變變得容易，再做那個容易的改變」的原則），並把那些工作排在最前面。接著它會以編號清單呈現拆分結果，並就此向你提問：粒度對嗎、阻擋關係是真的嗎、有沒有什麼應該合併或拆分。在你核准之前，什麼都不會進入 tracker，而那次提問就是你提出反駁的地方。

## 阻擋關係

阻擋關係是這個產出物的重點。依 tracker 的不同，它們以兩種方式運作：

| Tracker | 阻擋關係存在哪裡 | 你如何處理它們 |
| --- | --- | --- |
| 本地 markdown | 文字，寫在 `.scratch/<feature>/issues/<NN>-<slug>.md` 下每張 ticket 一個的檔案中，依阻擋者優先的順序編號 | 由上到下，手動處理 |
| 真正的 tracker（GitHub、Linear） | 原生的阻擋連結，或在 tracker 支援時使用子 issue | 任何阻擋者都已完成的 ticket 都在**前緣**上，可以接手 |

不論哪一種，阻擋關係都存在於 ticket 中。tracker 只決定是否有任何東西能平行地據以行動。`to-tickets` 產出這個產出物；執行它（一次一個 session，或一整個艦隊）是你的工作，而不是 skill 的。

## 大範圍重構的例外

有一種形狀會打破曳光彈規則。**大範圍重構（wide refactor）**是一個單一的機械性變更（改一個欄位名稱、改變一個共用符號的型別），它的**影響範圍（blast radius）**涵蓋整個程式碼庫。一次編輯就會弄壞數千個呼叫處，所以沒有任何垂直切片能以綠燈落地。

`to-tickets` 改為以**擴張再收縮（expand–contract）**的順序來排列它：

- **擴張（Expand）**：在舊形式旁邊加入新形式，讓什麼都不會壞。
- **遷移（Migrate）**：依影響範圍決定批次大小（每個套件、每個目錄），分批把呼叫處遷移過去，每一批一張 ticket，每一張都被「擴張」所阻擋。因為舊形式仍然存在，CI 保持綠燈。
- **收縮（Contract）**：在沒有任何呼叫者之後刪除舊形式，這張 ticket 被每一個遷移批次所阻擋。

當連批次都無法單獨保持綠燈時，它們會共用一個整合分支，並全部阻擋一張最後的「整合並驗證」ticket。CI 只需要在那張 ticket 上是綠燈。

## 常見問題

**它為一個三行的變更產出了十二張 ticket。**
過度拆解是這個 skill 被回報最多的問題，很多使用者都碰過。[模型](https://www.aihero.dev/ai-coding-dictionary/model)預設會切成原子單位，並失去能讓它們有意義的分組。提問那一步就是你修正這件事的地方。請它合併 ticket，它就會合併。還有一個下限。如果整個變更放得進一個 context window，你根本不需要這個 skill。直接進入 [implement](https://aihero.dev/skills-implement)。

**ticket 出來是每層一張：所有 schema 在一張，所有 API 在另一張。**
這正是垂直切片規則所要對抗的失敗，而這個 skill 有時仍然會產出它。在提問那一步，對每張 ticket 問一個問題來抓出它：這張完成時我能展示什麼？答不出來的 ticket 就是水平切片。有些人因此在每張 ticket 加一行「demo path（展示路徑）」，並回報這能把模型推向垂直切片。

**在 GitHub 上，ticket 沒有被建立成 spec issue 的子 issue。**
這在十幾次執行與好幾個模型上都被回報過，[在 issue #554 中描述得最完整](https://github.com/mattpocock/skills/issues/554)。已修正：`to-tickets` 現在會把每張 ticket 設為其來源 issue 的子 issue，而 GitHub tracker 範本提供了指令（`gh issue create --parent <n>`，`gh` 2.94 以上）。重新執行 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 來更新較舊的 `docs/agents/issue-tracker.md`。

**「Blocked by」被寫進 issue 內文，而不是一條真正的阻擋連結。**
這是同一類問題，[在 issue #513 中被回報](https://github.com/mattpocock/skills/issues/513)，其中 agent 甚至聲稱 GitHub 根本沒有原生的阻擋關係。它有：`gh issue create --blocked-by 12,15`。因為 skill 會先發布阻擋者，所以在建立時它們的編號一定可以取得。內文中的文字本應是給沒有原生關係之 tracker 的退路，而不是預設。現在當阻擋關係是原生的時，ticket 範本會省略 `## Blocked by`。

**本地的 ticket 放在哪裡？v1.1 的說明說是根目錄的 `tickets.md`。**
以前是，而那是一個 bug。一個共用的單一檔案，在平行的 agent 寫入時也會造成競態條件。本地模式現在依相依順序，在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下每張 ticket 寫一個檔案，與本地 tracker 範本早已描述的配置一致。`NN` 前綴是一個真正的 ticket ID，所以 `/implement 03` 就能運作，不用再打一長串標題。

**它在試圖讀取我的 spec 時一直被截斷。**
一份非常大的 spec，可能會超過 tracker issue 能乾淨回傳的量。沒有本地副本可以退回去用，所以 agent 會花費[工具呼叫](https://www.aihero.dev/ai-coding-dictionary/tool-call)一再擷取片段，卻永遠讀不到結尾。不要在 `/to-spec` 與 `/to-tickets` 之間[清空](https://www.aihero.dev/ai-coding-dictionary/clearing)或[壓縮](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一個 context window 中執行它們，agent 就永遠不需要再去擷取 spec。

**驗收條件什麼都沒評到：有些在任何工作完成之前就已經通過了。**
範本要求寫條件，卻沒說它們是否能失敗，所以這種情況會發生。有三種形狀反覆出現：在基準 commit 上就已經成立的條件、只有另一張 ticket 的工作才能滿足的條件，以及只是重述請求、而不是從產出物推導出來的條件。垂直切片能防止大部分的情況，因為一個交付新行為的切片，在構造上就會在基準 commit 上失敗。但這個檢查仍然值得手動做。對每個條件，說出能證明它不成立的觀察，並確認它在實作者開始的那個 commit 上會失敗。

**ticket 已經發布了。我實際上要怎麼執行它們？**
這個 skill 在產出物處就停止了，沒有自動派發模式。派發是手動的：看看看板，數一數沒有開放阻擋者的 ticket，然後開那麼多個 agent session。給每張 ticket 一個全新的情境，並在每張之間清空。[implement](https://aihero.dev/skills-implement) 在完成時並不會可靠地關閉或勾選 ticket，不論是在 GitHub 還是本地 markdown，所以你要自己更新 ticket 的狀態。

## 它有效的跡象

- 每張 ticket 都能回答「這張完成時我能展示什麼？」，而答案是一個行為，而不是一層。
- 在任何東西發布之前，清單會以編號的形式回到你手上，每張都有一行「Blocked by」。
- 最上面的 ticket 沒有阻擋者，可以立即開始。
- ticket 內文中沒有任何檔案路徑或行號，除了原型產出的程式碼片段。
- 每張 ticket 讀起來都像是一個全新的 session 在你不在場的情況下也能完成的東西。
- 如果找到了預先重構，它會排在順序的最前面，而不是混在功能 ticket 中。

## 它的定位

`to-tickets` 是主要建置鏈中的一步：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，它交給這個 skill 一份已確定的 spec 來切分。讓兩者留在同一個 context window 中，中間不要清空。下游是 [implement](https://aihero.dev/skills-implement)，它在每個全新的 session 中建置一張 ticket，以 [tdd](https://aihero.dev/skills-tdd) 驅動測試，並以 [code-review](https://aihero.dev/skills-code-review) 收尾。[implement-spec](https://aihero.dev/skills-implement-spec) 是另一條往下走的路。它把同樣的阻擋關係讀成一張任務圖，並在一個整合分支上平行建置每一張已就緒的 ticket。當你不確定哪個 skill 或流程適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
