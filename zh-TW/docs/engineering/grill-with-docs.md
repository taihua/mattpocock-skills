## 它做什麼

`grill-with-docs` 針對一個計畫或設計訪問你，直到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 對它有一致的理解，並在過程中把詞彙與困難的決策寫進你的 repo。它就是 [grill-me](https://aihero.dev/skills-grill-me) 所執行的同一場訪談（一回合問題、等待、然後下一回合），只是對準了一個程式碼庫。

它是**[有狀態的（stateful）](https://www.aihero.dev/ai-coding-dictionary/stateful)**。其他每一個拷問 skill 都把 [session](https://www.aihero.dev/ai-coding-dictionary/session) 留在你的腦中；這一個則在磁碟上留下檔案。當一個術語確定時，skill 會立刻把它寫進 `GLOSSARY.md`，而不是在最後一次批次處理。當一個決策通過三道關卡時，skill 會把它寫成一份 ADR。這就是全部的差別，而它也造成了人們使用這個 skill 時大部分的困擾。這些產出物是真實 repo 中的真實檔案，所以它們可能在你預期會出現時不見了，也可能在不只一個人寫入時逐漸偏離。

## 何時使用

你透過輸入 `/grill-with-docs` 來觸發它，agent 不會自己去用它。

在一個 repo 中、在一次變更的開頭使用它，此時計畫還很模糊，而描述那件事的用詞也還沒確定。它是單一 session 的工具。你要用哪一個拷問 skill，取決於你眼前的情況：

| 你手上有什麼 | 使用 |
| --- | --- |
| 你根本不是在一個工作目錄中工作 | [grill-me](https://aihero.dev/skills-grill-me) |
| 一個 repo，以及一個能在一個 session 內確定的變更 | `grill-with-docs` |
| 一項大到一個 session 裝不下的工作（全新建置、大型功能） | [wayfinder](https://aihero.dev/skills-wayfinder) |
| 一個完全沒有領域文件的 repo，而且心中沒有特定的功能 | `grill-with-docs`，對準 repo 本身而不是某個變更 |
| 一個卡在別人腦中知識的決策 | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

與 wayfinder 的區分歸結於 session 數量：單一 session 的規劃用 `/grill-with-docs`，多 session 的規劃用 `/wayfinder`。

## 前置條件

這個 skill 會寫入你的 repo，所以你需要在一個可以安全寫入的地方。確定的術語會寫到根目錄的 `GLOSSARY.md` 詞彙表，或者，如果根目錄的 `GLOSSARY-MAP.md` 把 repo 標記為多 context，就寫到相關 context 的 `GLOSSARY.md`。決策會寫到 `docs/adr/`。skill 只有在需要時才會建立兩者。在第一個術語或決策確定之前什麼都不存在，所以你不需要事先設定任何東西。

它也需要另外兩個 skill 存在，因為它自己的 `SKILL.md` 只有一行，就是把工作委派給它們。[grilling](https://aihero.dev/skills-grilling) 提供訪談，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供寫入。只安裝 `grill-with-docs`，你會得到一個無法運作的 skill。

## 書面紀錄

一個 session 會產出三種東西，而它們並不對等。

| 確定了什麼 | 落在哪裡 |
| --- | --- |
| 一個術語：專案自己對某個東西的用詞 | `GLOSSARY.md`，在它確定的當下即時寫入 |
| 一個難以逆轉、沒有背景就會令人意外，而且是真正取捨的決策 | `docs/adr/` 下的一份 ADR |
| 你決定的其他所有事情 | 對話中，而且只在那裡 |

第三列正是讓人們措手不及的地方。`GLOSSARY.md` 只是一份詞彙表。它不包含實作細節、不包含 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，也不包含草稿筆記。一份 ADR 需要三個條件同時成立，所以大多數決策都不符合，大多數 session 也不會產出任何 ADR。一個產出更精確的詞彙表、卻沒有任何 ADR 的 session 是按照設計在運作，但這代表你們約定的大部分內容，只存在於你們約定它時的那個 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) 中。把同一段對話交給 [to-spec](https://aihero.dev/skills-to-spec)，而不是[清空（clear）](https://www.aihero.dev/ai-coding-dictionary/clearing)它。

詞彙表是主要的產出。這個 skill 建立的是領域語言：專案自己的用詞，約定一次，讓你、agent 與你的同事都不必再推敲一遍。並不是每個人都同意這能改善 agent 的表現。最強的反對意見是，一個術語與它的白話展開，從[模型](https://www.aihero.dev/ai-coding-dictionary/model)那裡得到的結果是一樣的，而這套詞彙主要縮短的是共用它的人類之間的溝通。從這個角度來看，詞彙表仍然有價值，但價值歸於人類。

## 常見問題

**我應該用這個還是 `/wayfinder`？**
範圍決定了答案。任何你能在一個 session 內確定的事都用這個；當工作大到一個 session 裝不下時，用 [wayfinder](https://aihero.dev/skills-wayfinder)，它會先把工作畫成一張由決策 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 組成的地圖。Wayfinder 更慢、更密集，在一個範圍明確的功能上使用它是常見的錯誤。它不會取代這個 skill，而且對於地圖上適合拷問的部分，它可以啟動一場拷問 session。

**它執行了，但沒有出現 `GLOSSARY.md`，也沒有 ADR。**
有兩個已知的原因。第一個是沒有任何東西符合條件。ADR 需要通過全部三道關卡，而一個關於變更、卻沒有新詞彙的 session 就沒有東西可寫。第二個是一個真正的 bug。當這個 skill 在另一個編排層內執行時（一個 spec 驅動開發的包裝層、一個多 agent 框架、一條把它當作別人管線中某一步來觸發的規則），使用者回報寫入檔案的那一半會默默地不發生，而訪談仍然照常進行。這個 bug 已經回報，但尚未修正。如果你處於那種設定中，在相信 session 的產出之前，先檢查工作目錄。

**它一次把所有問題都丟出來，沒有建議答案，也從沒提到 `GLOSSARY.md`。**
這是 skill 沒能載入它的兩個依賴。因為 `SKILL.md` 只是一行委派，一個沒有載入 [grilling](https://aihero.dev/skills-grilling) 與 [domain-modeling](https://aihero.dev/skills-domain-modeling) 的 agent 只能猜測拷問是什麼意思，於是你會一次拿到所有問題，沒有任何結構。只載入一部分更令人困惑。`grilling` 載入了，`domain-modeling` 沒有，你就會得到一場很好的訪談，卻沒有任何書面紀錄。這種情況多常發生取決於模型與[推理強度（effort）](https://www.aihero.dev/ai-coding-dictionary/effort)等級，而它是這個 skill 被回報最多的問題。如果你懷疑發生了這種情況，直接問 agent 它載入了哪些 skill。

**我其他的決策都跑到哪裡去了？**
只在對話中。這是對這個 skill 最嚴重、仍未解決的抱怨。詞彙表不是 spec，大多數答案也夠不上一份 ADR，而且沒有任何紀錄把每個已確定的答案連結到 spec、ticket 與測試。後續的步驟會把精確的答案（順序保證、否定式的需求、數值預設值）軟化成比較弱的文字，結果可能看起來很完整，卻遺漏了你所決定的事。目前，請保留這個 session，並直接把它餵給 [to-spec](https://aihero.dev/skills-to-spec)。然後拿你自己的答案重新檢查 spec，而不是假設它已經記錄下來了。

**我可以把它指向一個完全沒有文件的既有 repo 嗎？**
可以。對於沒有 ADR、沒有領域語言、沒有設計原則的程式碼庫，這就是正確的 skill：觸發它並說「help me document my repo（幫我為我的 repo 建立文件）」。使用者常常把它與 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 搭配使用，來建立或修復一份 `GLOSSARY.md`。預期你需要引導它。它會讀程式碼並就它發現的東西問你，而由你決定程式碼庫中已有的哪些用詞才是正確的。

**session 結束時我該做什麼？**
這個 skill 的結尾訊息常常沒有明確指示，這是一個已知的問題。在主流程中，答案是在同一段對話中執行 [to-spec](https://aihero.dev/skills-to-spec)。如果變更小到可以立刻建置，就改為直接進入 [implement](https://aihero.dev/skills-implement)。

**它為什麼叫這個名字？**
沒有人對這個名字滿意。有一個開放中的建議要把它改名為 `grill-domain-model`，那更準確地描述了它的行為。這件事沒有任何進展。如果有一天真的改名了，文件頁面會跟著移動，URL 也會改變。

## 它有效的跡象

- `GLOSSARY.md` 在 session *進行中*逐個術語地改變，而不是在最後一次全部出現。
- 詞彙表讀起來是純粹的詞彙（你專案的用詞與精簡的定義），不包含任何實作細節或像 spec 的文字。
- 程式碼庫能回答的問題，是透過閱讀程式碼庫來回答，而不是拿來問你。
- 你得到很少或沒有 ADR，而你得到的那些，是你會很不想再爭論一次的決策。
- 它會因為你既有的詞彙表對某個詞有不同的定義，而質疑你用的那個詞。

## 它的定位

`grill-with-docs` 是主要建置鏈的開頭：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它在任何東西被寫成 spec 之前出現。它產出共同的理解與確定的詞彙，然後由 [to-spec](https://aihero.dev/skills-to-spec) 在不再次訪問你的情況下加以綜整。它的近鄰是 [grill-me](https://aihero.dev/skills-grill-me)（同樣的訪談，沒有 repo 也沒有檔案），以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)（它所驅動的詞彙表與 ADR 紀律）；兩者都使用 [grilling](https://aihero.dev/skills-grilling) 基本元件來進行訪談。在它的上游，[wayfinder](https://aihero.dev/skills-wayfinder) 為一個 session 裝不下的工作繪製地圖，並可以把地圖的某些部分交回給它。當你不確定哪個 skill 或流程適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
