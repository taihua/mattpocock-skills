## 它做什麼

`retro` 回顧一次 coding [session](https://www.aihero.dev/ai-coding-dictionary/session)，並建議如何改善 agent 的**[環境（environment）](https://www.aihero.dev/ai-coding-dictionary/environment)**，讓下一次執行更順利。它讀取那個 session 本身的紀錄（預設是目前這一個，或你在 session 紀錄中指給它的那一個）。它找出 agent 掙扎的時刻，並給你一份候選修正清單，最嚴重的排在前面。

它改變的是環境，而不是程式碼。以 agent 交付出去的 bug、它花了二十次[工具呼叫](https://www.aihero.dev/ai-coding-dictionary/tool-call)才找到的檔案，或審查者漏掉的規則為例。`retro` 不會直接修正其中任何一個。它會問 repo 中是什麼讓這些事情得以發生，並提議能在下次預防它們的檢查、指標或規範。而且它只負責提議。在你挑選一個候選項目之前，什麼都不會改變。

## 何時使用

你透過輸入 `/retro` 來觸發它，agent 不會自己去用它。

在一個比應有的更艱難的 session 結束時使用它。例如，agent 花了太久去找某個東西、犯了一個工具本可抓到的錯，或需要一個它取得不了的資訊。順利的 session 沒什麼好學的，發現都來自困難的 session。如果你想要對這個 session 產出的程式碼做出評斷，請改用 [code-review](https://aihero.dev/skills-code-review)。

## 發現會落在哪裡

每個候選項目都屬於一個類別，而類別決定了修正要放在哪裡：

| session 中出了什麼問題 | 用什麼修正 |
| --- | --- |
| agent 花了很久才找到某個檔案或事實 | 從一個它本來就會讀的檔案放一個**導覽指標** |
| 它犯了一個工具本可抓到的錯 | 一個**[自動化檢查](https://www.aihero.dev/ai-coding-dictionary/automated-check)**：lint 規則、型別、測試、pre-commit hook、CI job |
| 審查者漏掉了一個屬於判斷題的錯誤 | 在 `CODING_STANDARDS.md` 中為審查 agent 加一條規則 |
| `AGENTS.md` 或 `CLAUDE.md` 很龐大 | 把其中的引導內容移出去，移到規範或檢查中 |
| 某次工具呼叫的成本相對於它的回傳來說很高 | 精簡這個工具，或替換它 |
| 某個引導檔案充滿了什麼都沒改變的句子 | 刪除這些 **no-op** |
| agent 需要一個它取得不了的資訊 | 擴大它的存取範圍：把 dev server 的 log 同時輸出到檔案、給予對某個服務的唯讀存取 |

主導概念是：規範屬於**審查者**，而不是實作者。實作的 agent 承受最大的情境壓力，因為它要探索、寫程式碼、除錯。審查的 agent 只拿到一份 diff，其他什麼都沒有。所以新規則要放在有空間套用它的地方，也就是審查。它絕不放在 [AGENTS.md](https://www.aihero.dev/ai-coding-dictionary/agents-md) 中，因為那個檔案不論相不相關，都會載入每一個 session 的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)。

在 `retro` 寫任何規則之前，它會先分類違規的性質。**機械性**的違規（被禁止的 API、import 的形式、檔案位置的規則）會得到一個確定性的檢查，因為檢查可以失敗，而規範檔中的一句話不會。只有真正的判斷題，也就是任何 linter 都無法強制執行的，才會變成文字。如果 repo 完全沒有護欄（沒有 pre-commit hook，也沒有執行 lint、型別檢查與測試的 CI job），`retro` 會把這件事本身回報為一項發現。

## 常見問題

**它會自己寫 lint 規則，還是等我說好？我能把它設定成在每個 session 結束後自動執行嗎？**

它會等。`retro` 只負責提議。在你挑選一個候選項目之前，什麼都不會改變，所以它不會手動編輯任何東西，也不會自動安裝任何 hook。這是刻意的。一位使用者在「被那些擋掉好變更的自動 hook 燙過」之後，正是要求這一點。決定什麼值得一個永久的檢查需要判斷力，所以這個 skill 保持[人在迴圈中（human-in-the-loop）](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)，並且是使用者觸發的。有些使用者確實在每次實作執行後都跑它。但順利的 session 沒什麼好學的，所以在每個 session 後都跑，大多只會產出沒人需要的規則。它沒有試跑（dry-run）模式。提議的檢查就跟其他程式碼一樣，所以在讓它擋住合併之前，先對 repo 試試看。

**這不會讓 lint 規則永遠越堆越多嗎？它有沒有建議過移除某條規則？**

部分會，而這是它最弱的地方。它可以移除文字：引導檔案中的 no-op，以及 `AGENTS.md` 或 `CLAUDE.md` 中應該屬於規範或檢查的引導內容。當這些檔案很龐大時，它會把它們標示為可刪除。它是以它讀到的那一個 session 來評斷它們，所以把每一項都當作刪除測試的候選，而不是定論。它不會稽核它上個月提議的 lint 規則、hook 或 CI job。它只看得到一個 session，所以它無法告訴你某條規則現在觸發得太頻繁，或它背後的 bug 已經不存在了。你仍然必須自己修剪檢查。一條在好的程式碼上不斷觸發的規則，就是該移除它的訊號。

**它不會為了填滿類別，就編出一些通用的建議嗎？**

這是對它最強烈的批評。一位使用者發現：「一旦工作完成，AI 往往會忘記 session 中間的掙扎，並編造通用的建議來滿足回顧的類別。」每一個候選項目都必須來自 session 本身的紀錄，所以建議是針對那個 session 的。這有好處也有代價。它很少編造無關的東西，但它可能會過度重視這一個 session 恰好處理的事情。丟掉任何你無法追溯到某個具體時刻的候選項目。也把嚴重程度的排序當作初稿，因為一個安靜但昂貴的錯誤，可能排在一個吵鬧但便宜的錯誤之後。

**我的 session 很長。現在就執行，還是重新開始？**

預設上它審查的是目前的 session。這是最理想的情況，因為那些掙扎還在 context window 中。如果 session 已經離開了[聰明區（smart zone）](https://www.aihero.dev/ai-coding-dictionary/smart-zone)，就[清空（clear）](https://www.aihero.dev/ai-coding-dictionary/clearing)它，然後在新的 session 中執行 `/retro`，把它指向 session 紀錄中的前一個 session。

**agent 一直犯同樣的錯。我應該在 `CLAUDE.md` 加一行嗎？**

通常不應該，而這是 `retro` 最常反對的事。`CLAUDE.md` 中的一行會載入每一個 session、降低檔案中其他所有內容的權重，而且會隨著程式碼改變而過時。如果錯誤是機械性的，修正方式就是一個會失敗的檢查。如果是判斷題，它就放進審查者會讀的程式碼規範中。`AGENTS.md` 與 `CLAUDE.md` 是給導覽指標用的，幾乎沒有其他用途。出於同樣的理由，`retro` 不是一個[記憶系統](https://www.aihero.dev/ai-coding-dictionary/memory-system)。它不儲存發生過什麼。它改變環境，讓錯誤不會再發生。

**我的設定提到了 `CODING_STANDARDS.md`，但我沒有這個檔案。它從哪裡來？**

沒有任何 skill 會附帶這個檔案。當某個 session 第一次為審查者找到一條判斷題規則時，`retro` 會提議建立它。在你接受之後，[code-review](https://aihero.dev/skills-code-review) 就會讀它。任何你已經在維護的其他規範文件，例如 `CONTRIBUTING.md`，運作方式都一樣。

**它和 `improve-codebase-architecture` 有什麼不同？**

輸入不同。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 只需要程式碼，並尋找它在結構上的改善。`retro` 需要一段 session 歷史，並改善 agent 工作的環境，而不是程式碼。你兩者都會用；兩者都無法取代對方。

## 它有效的跡象

- 每一個候選項目都能指回 session 中某個具體的時刻，而不是一個通用的最佳實務。
- 重複的錯誤會變成會失敗的檢查，而你的 `AGENTS.md` 會隨著時間變短，而不是變長。
- 如果某個檢查早已存在但沒有接上，發現的內容是把它接上，而不是提議建立一個新的。
- 下一個處理同類任務的 session 會更快找到它需要的東西。

## 它的定位

`retro` 是主鏈的最後一步，你在這裡回顧整條鏈進行得如何：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

在一次值得從中學習的建置之後執行它，在同一個 session 中，或指向那個 session 的紀錄。在順利的建置之後，你可以跳過它。

- [code-review](https://aihero.dev/skills-code-review) 是 `retro` 最常調整的審查 agent。新的程式碼規範會放在它的 Standards 軸會讀取的地方。
- [writing-for-agents](https://aihero.dev/skills-writing-for-agents) 為 `retro` 提議的每一個引導檔案與 skill 設定寫作風格，而 `retro` 在開始之前會先載入它。

當你不確定當下情況需要哪個 skill 時，[ask-matt](https://aihero.dev/skills-ask-matt) 會在整套 skill 中為你指路。
