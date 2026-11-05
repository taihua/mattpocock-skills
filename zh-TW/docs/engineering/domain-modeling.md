## 它做什麼

`domain-modeling` 在你設計的同時，建立並磨利專案的**通用語言（ubiquitous language）**。它會質疑一個與詞彙表衝突的術語、在你用了模糊用詞的地方要求一個精確的詞，並用一個具體的情境測試一個關係，直到邊界變得精確。

它是**主動**的紀律，而不是被動的。任何 skill 都可以讀 `GLOSSARY.md` 來借用它的詞彙。這個 skill 是用在你要*改變*模型的時候，這就是為什麼它會打斷你。它在你確定一個術語的那一刻，就在對話中途把它寫進 `GLOSSARY.md`，而不是在最後產出一份整齊的詞彙表。在最後才寫的詞彙表，是一個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 的摘要；即時寫入的版本才是 session 的產出。

## 何時使用

輸入 `/domain-modeling`，或者當任務適合時，agent 會自動使用它。實務上，自動觸發是這個 skill 最弱的部分。當 `grill-with-docs` 或 `wayfinder` 說要載入它時，[模型](https://www.aihero.dev/ai-coding-dictionary/model)常常載入了 `grilling` 卻跳過這一個。如果一場[拷問（grilling）](https://www.aihero.dev/ai-coding-dictionary/grilling) session 執行完了，而 `GLOSSARY.md` 在最後沒有改變，就是發生了這種情況。和另一個 skill 一起指名觸發它。

當*用詞*是問題所在時使用它：

| 情況 | 做法 |
| --- | --- |
| 兩個人說「cancellation（取消）」時指的是不同的東西 | `domain-modeling`：挑選標準術語，把另一個列在 `_Avoid_` 下 |
| 「Account」在三個檔案中身兼三職 | `domain-modeling`：把它拆成 Customer 與 User |
| 你剛做了一個難以逆轉的架構選擇 | `domain-modeling`：如果這個選擇過了門檻，它會提議寫一份 ADR |
| 問題在於模組的*形狀*：接縫放在哪裡、介面有多深 | [codebase-design](https://aihero.dev/skills-codebase-design) |
| 你想在建置之前讓整個計畫被盤問 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它在底層驅動這個 skill |
| 你想查一個術語，而不是改變它 | 什麼都不用。讀 `GLOSSARY.md`。它是一個檔案。 |

## 前置條件

事先什麼都不需要。這個 skill 寫入兩個地方，並延遲建立兩者：

- repo 根目錄的 **`GLOSSARY.md`**。skill 在你確定第一個術語時建立它。在根目錄有 `GLOSSARY-MAP.md` 的 repo 中，術語改為寫入地圖所指向的、各 context 各自的 `GLOSSARY.md`。
- **`docs/adr/`**。skill 在第一份過了門檻的 ADR 出現時建立它。

開始之前不需要任何東西存在，而 skill 也不會事先建立任何東西。

## 兩種產出物，兩道門檻

詞彙表與 ADR 有不同的門檻。這個 skill 大部分的麻煩都來自把它們混為一談。

| | `GLOSSARY.md` | `docs/adr/NNNN-slug.md` |
| --- | --- | --- |
| 存放 | 術語。一個東西「**是**」什麼，用一到兩句話描述，被否決的同義詞放在 `_Avoid_` 下 | 一個決策，用一到三句話描述：背景、選擇、理由 |
| 寫入的門檻 | 一個模糊的術語變成了標準 | **三者全部**：難以逆轉、沒有背景就會令人意外、是真正取捨的結果 |
| 寫入方式 | 即時寫入，在術語確定的那一刻 | 提議，而不是預設 |
| 絕不存放 | 實作細節、[spec](https://www.aihero.dev/ai-coding-dictionary/spec)、草稿、一般程式設計概念 | 這個 session 中所做每一個選擇的日記 |

只要沒通過 ADR 三項測試中的任何一項，就沒有 ADR。一個容易逆轉的決策，直接逆轉就好；一個不令人意外的決策，沒有人會有疑問；一個沒有真正替代方案的決策，記錄的只是你做了顯而易見的事。

`GLOSSARY.md` 的規則最重要，因為它是實務上會被打破的那一條。**它是一份詞彙表，僅此而已。** 不加以控制的話，模型會把「寫入 `GLOSSARY.md`」當成可以保存你給的每一個答案的許可，於是這個檔案變成一份持續增長的 spec。這是這個 skill 在多個模型上被回報最多的問題。

## 交叉比對，以及它停在哪裡

當你說明某件事如何運作時，skill 會檢查程式碼，並指出任何矛盾。*「你的程式碼是取消整張 Order，但你剛才說可以部分取消，哪一個才對？」* 這個 skill 會大聲地讓語言與程式碼一致，然後才改變其中任何一個。

它交叉比對的是**程式碼**以及已 commit 的 `GLOSSARY.md` 與 ADR，此外什麼都不比對。它不會搜尋你的 issue tracker。如果你的團隊幾個月前在一個已關閉的 issue 中爭論並解決了一個命名衝突，skill 會把它當成新問題再提一次。有一個[開放中的請求](https://github.com/mattpocock/skills/issues/717)要修正這一點。在那之前，變通方法是把指示放在你自己的 `docs/agents/domain.md` 中，這些 skill 本來就會讀它。

## 常見問題

**我的 `GLOSSARY.md` 有 500 行。1,000 行。3,000 行。我該怎麼辦？**
大小只是症狀。原因是這個檔案吸收了從來就不屬於詞彙表的實作細節與決策。修正方式是一個直接的指示：`/grill-with-docs make my GLOSSARY.md more concise and remove any implementation details from it`（讓我的 GLOSSARY.md 更精簡，並移除其中所有的實作細節）。對一個臃腫的檔案執行它，大部分內容都會消失。只有在檔案已經精簡、但仍涵蓋讀者不會想同時掌握的兩個領域時，才用 `GLOSSARY-MAP.md` 拆分它。拆分一個臃腫的檔案，只會給你好幾個臃腫的檔案。這個 skill 在這方面的指引還不夠強，無法從一開始就防止膨脹，而追蹤這件事的 issue 仍然開放中。

**為什麼是 `GLOSSARY.md` 而不是 `CONTEXT.md`？**
在 1.3.0 之前它叫做 `CONTEXT.md`（以及 `CONTEXT-MAP.md`）。這個檔案是一份詞彙表，僅此而已，所以 `GLOSSARY.md` 就直說了，而且正如一位讀者所說：「對 AI agent 來說，所有東西都是[情境（context）](https://www.aihero.dev/ai-coding-dictionary/context)」。如果你的 repo 還有 `CONTEXT.md`，重新執行 `/setup-matt-pocock-skills`：它會提議用 `git mv` 改名，並更新 `docs/agents/domain.md`。這套 skill 中的每一個都會尋找 `GLOSSARY.md`，所以保留舊名稱就代表要修補它們全部。

**`/ubiquitous-language` 去哪了？**
它被移除了，而不是被標為棄用。它的工作移到了 `domain-modeling`，後者持續維護整個模型，而不是從一段對話中傾倒出一份詞彙表。詞彙的強制執行現在更重要，而不是更不重要。它在拷問、分類與繪製地圖的底層執行，而不是一個你必須記得的獨立步驟。

**一個完全沒有詞彙表的程式碼庫，我要怎麼得到一份？**
明確地要求它，而不是等它自己累積。`/grill-with-docs help me scaffold my existing repo with a GLOSSARY.md`（幫我為既有的 repo 建立一份 GLOSSARY.md 骨架）是有文件記載的路徑。預期會有一場很長的盤問。一位使用者回報在檔案成形之前被問了超過 50 個問題。在一個既有（brownfield）的 repo 上，靠順帶使用來建立詞彙表實在太慢了。

**我能保留領域模型，但使用我自己的 ADR 格式嗎？**
目前無法乾淨地做到。詞彙表那一半與 ADR 那一半是放在同一個 skill 中發布的，所以一個已經有既定 ADR 慣例的團隊（不同的範本、不同的位置、不同的命名），會得到與它內部風格衝突的指示。目前的選項是把 skill 複製到本地並編輯它，或在你 repo 自己的 agent 文件中覆寫 ADR 的慣例。把兩者拆開是一個[開放中的請求](https://github.com/mattpocock/skills/issues/557)。

**詞彙表真的值得它的成本嗎？它是又一個要審查的產出物，而且可能會過時。**
有時候不值得。DDD 越接近實作就越沒用。回報在上游，在命名與概念的對齊上，而不是在聚合（aggregate）與分層的儀式上。同義詞的控制在命名邊界上很重要：模組名稱、資料表名稱、狀態列舉、issue 標題、CLI 指令。在一般的文字中它就沒那麼重要了。還有一個反對意見是，領域術語壓縮的是已經共用它們的*人類之間*的溝通，而 agent 對白話描述的反應是一樣的。照這個解讀，詞彙表的價值是讓你和你的審查者與 agent 正在做的事保持一致，而不是讓 agent 變得更好。在一個一天的建置上，跳過它。而且一份未經審查、由 agent 撰寫的詞彙表比沒有還糟：它會充滿充滿自信的主張，而之後的 session 會把它們當成真的。

**它能替我把模糊的提示轉成領域語言嗎？**
不能，而且也沒有計畫做這樣的 skill。一套你自己都不理解的領域語言，一旦寫下來就毫無意義。這個 skill 在你有了理解之後強制精確；它不會發明你沒有的詞彙。相關的陷阱是使用領域用詞卻沒有做建模。在錯誤的概念上套用正確的名詞，會產出讀起來正確、實際上卻不是的結果。

## 它有效的跡象

- 它會在你說到一半時打斷你，問你指的是兩件事中的哪一件，而不是挑一個就繼續。
- `GLOSSARY.md` 在對話**進行中**改變，而不是在最後一次爆發。
- 它拒絕為一件你明天就能撤銷的事寫 ADR，並說明三項測試中哪一項沒通過。
- 新條目用一到兩句話定義一個東西*是*什麼，並在 `_Avoid_` 下列出你放棄的用詞。
- 當你的程式碼與你說的話不一致時，它會把你的程式碼引用給你看。
- `GLOSSARY.md` 變短的頻率和變長一樣高。

## 它的定位

`domain-modeling` 是一份**模型觸發的參考資料**，在其他 skill 的*底層*執行的次數，比它單獨執行的次數更多。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 透過一場拷問 session 驅動它，[wayfinder](https://aihero.dev/skills-wayfinder) 在繪製地圖時載入它，[triage](https://aihero.dev/skills-triage) 用它讓 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 使用專案自己的用詞，而 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在決策確定時呼叫它。它最近的兄弟是 [codebase-design](https://aihero.dev/skills-codebase-design)。它們一起構成其他一切底下的詞彙層，這一個負責*領域*，那一個負責模組的*形狀*。當你想要這套紀律、又不想投入通常會載入它的那個 skill 的步驟時，也可以直接使用它。當你不確定哪個 skill 適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
