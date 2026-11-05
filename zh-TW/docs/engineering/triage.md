## 它做什麼

`triage` 處理你專案 tracker 上的 issue。它讓每一個 issue 在一個由**分類角色（triage roles）**組成的小型狀態機（一個類別角色與一個狀態角色）中流轉。每個 issue 最後會變成一份 agent 可直接使用的簡報、一個給回報者的具體問題，或一個附有記錄理由的已關閉 issue。

它只用於**不是你自己建立的** issue。也就是原始的 bug 回報、外部進來的功能請求，或一個不請自來的外部 pull request：從外部進入 tracker、保持著回報者留下時的模樣的工作。來自 [to-tickets](https://aihero.dev/skills-to-tickets) 的 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 已經是 agent 可直接處理的，所以如果你對它們執行 `triage`，就是在浪費工作。規則很簡單：`/triage` 只用於外來的 issue，而不是你自己建立的 issue。

它也與手動加標籤不同，因為它會先給出建議，然後等待。它把它的類別與狀態判斷連同理由，以及它在程式碼庫中發現的東西告訴你，並在你指示之前不套用任何東西。

## 何時使用

你透過輸入 `/triage`，然後用白話描述你想要什麼來觸發它。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。範例：「給我看任何需要我關注的東西」、「我們來看看 #42」、「把 #42 移到 ready-for-agent」。

| 你手上有什麼 | 去哪裡 |
| --- | --- |
| 一個充滿別人送來之原始回報的 tracker | `/triage` |
| 一個你自己粗略的想法，什麼都還沒寫下來 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 一段已經有結論、要轉成 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 的對話 | [to-spec](https://aihero.dev/skills-to-spec) |
| 一份要拆成 agent 可直接處理之 ticket 的 spec | [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一個已確認、需要找出根本原因而不是標籤的 bug | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 前置條件

`triage` 會讀寫你的 issue tracker，所以必須先用 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 設定好那個 tracker 及其標籤詞彙。下方的角色名稱是**標準**名稱。你 tracker 中的標籤字串可能不同，而設定會提供對應關係。如果你的 tracker 已經完全使用標準名稱，你就沒有任何東西需要對應，也沒有任何東西需要設定。

tracker 的設定也決定了外部 pull request 是否算是一種請求管道，以及誰算是外部的。那個旗標預設是關閉的，而設定已經不再詢問這件事。要把 PR 納入範圍，就在 `docs/agents/issue-tracker.md` 中把它打開。

## 狀態機

每一個被分類的項目最後都剛好有一個類別角色與一個狀態角色。有兩個類別：`bug`（某個東西壞了）與 `enhancement`（新功能或改善）。有五個狀態：

| 狀態 | 意思 |
| --- | --- |
| `needs-triage` | 你需要評估它。沒有標籤的 issue 通常會先落在這裡。 |
| `needs-info` | 等待回報者。當他們回覆時，回到 `needs-triage`。 |
| `ready-for-agent` | 已完整定義，並附上一份 agent 簡報。一個 [AFK（離線）](https://www.aihero.dev/ai-coding-dictionary/afk) agent 可以接手。 |
| `ready-for-human` | 同樣的簡報，加上為什麼 agent 做不了：需要判斷、需要外部存取權、需要手動測試。 |
| `wontfix` | 已關閉，並記錄理由。 |

這就是全部的詞彙。每個項目剛好一個狀態角色的規則，讓查詢保持簡單。這些狀態也是這個 [skill](https://www.aihero.dev/ai-coding-dictionary/skill) 被要求最多的部分。使用者要求過一個給「已定義但被另一個 issue 阻擋」之工作的狀態、一個給「等待未來觸發條件」之工作的 `deferred` 狀態，以及一個終止性的 `implemented` 狀態。這些都還沒有發布。參見下方的問題。

`wontfix` 有三種情況。這個差別很重要，因為其中只有一種會寫入知識庫：

| 你為什麼要關閉它 | 會發生什麼 |
| --- | --- |
| 已經實作了 | 一則指出該功能已經存在於哪裡的留言。不會寫入 `.out-of-scope/`，因為它是一個已建好的功能，而不是被拒絕的功能，而在那裡放一個檔案會破壞去重複的檢查。 |
| 被拒絕的 bug | 禮貌的說明，然後關閉。 |
| 被拒絕的功能增強 | 在 `.out-of-scope/` 中放一個檔案，從關閉留言連結過去，然後關閉。 |

`.out-of-scope/` 為每一個被拒絕的**概念**存放一個 markdown 檔，而不是每個 issue 一個。每個檔案都是一份簡短的設計文件，而不是一筆資料庫紀錄。它說明拒絕了什麼、為什麼，並列出每一個要求過它的 issue。`triage` 在評估任何東西之前會讀取整個目錄。它依概念比對，而不是依關鍵字，所以「night theme（夜間主題）」與 `dark-mode.md` 相符。當它找到相符的項目時，它會給你看舊的決策，並問你是否仍然同意，而不是從頭再爭論一次這個請求。

## 寫簡報之前先驗證

在任何[拷問](https://www.aihero.dev/ai-coding-dictionary/grilling)之前，`triage` 會先檢查主張是否站得住腳。對 bug 而言，它會依照回報者的步驟重現它。對 PR 而言，它會 checkout 分支並執行相關的測試。然後它回報三種結果之一：

- 已確認，附上程式路徑。
- 無法重現。
- 細節不足以嘗試。這是最強烈的 `needs-info` 訊號。

在同一輪中，它還會對程式碼庫執行另外兩項檢查。**重複性**檢查會問這個功能是否已經實作，並依領域概念搜尋，而不是依回報者的用詞。**先前的拒絕**檢查會問 `.out-of-scope/` 是否已經說過不。兩項檢查都很便宜，而其中任何一項命中都會產生一個 `wontfix`。

這一切工作都是為了做好一個產出物：**agent 簡報（agent brief）**。這是 `triage` 在一個 issue 移到 `ready-for-agent` 時發布的結構化留言。`triage` 發布它之後，簡報就是合約，而原始回報只是背景。簡報是**耐久的**而不是精確的，因為一個 issue 可能在 `ready-for-agent` 中停留好幾週，而程式碼在此期間會改變。所以簡報指名的是型別、簽章與行為合約，絕不是檔案路徑或行號。一個經過確認的重現，會比一個猜測產出強得多的簡報。

## PR 就是附帶程式碼的 issue

如果 tracker 把外部 pull request 視為一種請求管道，它們就會經過同一個狀態機，使用相同的類別、狀態與轉換。狀態套用在 diff 上。`ready-for-agent` 代表已附上簡報，agent 應該對程式碼進行下一步。`ready-for-human` 代表一個人可以合併它。PR 上的簡報描述的是對既有 diff 還剩下哪些要做的事，而不是如何從零建立那個東西。

探索時只會顯示*外部*的 PR，因為協作者進行中的分支不是分類的工作。那個過濾只適用於探索。如果你明確指名一個 PR，不論是誰寫的，`triage` 都會處理它。

## 常見問題

**我執行了 `/to-spec` 與 `/to-tickets`，現在那些 ticket 都還沒分類地放在那裡。我要對它們執行 `/triage` 嗎？**
不用。它們已經是 agent 可直接處理的。`to-tickets` 在發布它們時會套用 `ready-for-agent` 標籤，所以一個 AFK 執行器不需要再過一輪就會接手它們。碰到這個問題的使用者執行了 spec 流程，卻在產出上看到 `needs-triage`，而他們的 AFK 執行器忽略了所有東西。`triage` 是給從外部送來之工作的匝道。spec 流程是給你自己開始之工作的車道。兩者在 `ready-for-agent` 交會，而不是之前。

**既然已經有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 還有用嗎？**
只有在你有外來的工作時才有用。`triage` 比那個流程更早出現，做的是不同的工作：它處理別人提出的回報。如果你 tracker 中的一切都來自你自己的規劃，你很少會用到它。如果你維護任何公開的東西，或你的團隊會向你回報 bug，那裡就是那些工作的起點。主要的用途是接收外部貢獻者 issue 的開源 repo。

**agent 試著套用 `ready-for-agent`，而 `gh` 說這個標籤不存在。**
這是一個已知的開放 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 會把標籤詞彙寫進 `docs/agents/triage-labels.md`，但不會在你的 tracker 中建立這些標籤。自己用 `gh label create` 或 tracker 的 UI 建立五個狀態標籤與兩個類別標籤，一次就好，錯誤就會停止。這個 issue 連結到一個尚未合併的社群修正分支。

**五個狀態不夠用。那 blocked、deferred 或 implemented 呢？**
這是這個 skill 被提出最多次的缺口。它有三種形式：

- 一個已完整定義、但要等另一個 issue 關閉的 issue（[#139](https://github.com/mattpocock/skills/issues/139)）。回報者說 `ready-for-agent` 在那裡「技術上是正確的」但具有誤導性，所以 agent 會接手它，然後卡住。
- 預定要做、但要等某個觸發條件的未來工作，所以它還無法處理（[#297](https://github.com/mattpocock/skills/issues/297)）。
- 一個代表「已實作，等待驗證」的終止狀態。沒有它，一個 AFK 執行器可能會把完成的 ticket 再次排入佇列。

被阻擋的情況被接受為真實存在，但名稱尚未決定（`blocked` 還是 `paused`）。這些都還沒有發布。作為變通方法，人們會在類別旁邊加一個 repo 本地的額外標籤。這樣狀態欄位就能保持一個準確的值，但 skill 並不知道那個額外的標籤。一個社群 fork 走得更遠，加入了 `needs-slicing`、`tracking` 與工作量標籤。那可以運作，但它屬於那個 fork，而不屬於這個 skill。

**這和 `/diagnosing-bugs` 有什麼不同？**
這裡的驗證步驟是刻意做得很淺的。它回答的是「這是真的嗎，大概在哪裡」，而不會去找根本原因。如果一個 bug 無法在幾分鐘內依照回報者的步驟重現，就用 `needs-info`，或者如果你想現在就調查它，就用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。兩個 skill 的文字目前都沒有提到對方。一位使用者回報了這個缺口，而它仍然開放。

**我可以把它指向我整個待辦清單，然後讓它跑嗎？**
你可以要求，但要注意它讀了什麼。「顯示需要關注的項目」那一輪是一個用於*挑選*的便宜清單。你挑一個 issue，然後 `triage` 才會為那個 issue 蒐集完整的[情境](https://www.aihero.dev/ai-coding-dictionary/context)。如果你一次對二十個 issue 執行它，agent 可能會在不告訴你的情況下，把那份便宜的清單當成唯一的證據。那份清單會回傳 issue 內文，但不包含留言。一位使用者正是碰到了這個問題。三個 issue 已經有一則留言說「已經修好了，建議關閉」，結果三個都得到了新的 agent 簡報。對於批次處理，要明確說明它必須讀取每個 issue 的留言。

**它能用在 Linear，或 GitHub Issues 以外的任何東西上嗎？**
可以。tracker 是設定，而不是寫死的假設。人們用它搭配 Linear（透過 `linear` CLI）、GitLab，以及 `.scratch/` 下的純 markdown 檔案。一種常見的分工是用 Linear 處理 issue 與規劃，用 GitHub 處理程式碼與 PR。這樣說「issue tracker」的 skill 會對應到 Linear，說「PR」的 skill 會對應到 GitHub。本地 markdown tracker 有一個開放中的範本 bug：產生的檔案可能會包含兩次驗收條件，一次在最上層，一次在 agent 簡報裡面（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 它有效的跡象

- 它碰過的每個項目最後都剛好有一個類別角色與一個狀態角色，從不是零個，也從不是兩個互相衝突的狀態。
- 它給你一個附理由的建議，然後停下來。它不會重新加標籤然後就繼續下去。
- 在任何東西進入 `ready-for-agent` 之前，它重現了 bug，或 checkout 並執行了 PR。
- 它寫的簡報指名的是型別與行為，而且不包含任何檔案路徑或行號。
- 當一個你六個月前拒絕的請求又回來時，它會告訴你並引用舊的理由，而不是再分類一次。
- 它發布的每一則留言都以 `> *This was generated by AI during triage.*` 開頭。

## 它的定位

`triage` 是一條**匝道**，而不是主鏈中的一步。主流程從你的一個想法開始（拷問、spec、ticket、實作、審查）。`triage` 是給來自別人之工作的平行車道。兩條車道終點相同：一個標上 `ready-for-agent`、附有簡報的 issue。[implement](https://aihero.dev/skills-implement) 接手它的方式，就和接手一張來自 [to-tickets](https://aihero.dev/skills-to-tickets) 的 ticket 一樣。當一個請求在 `triage` 能為它寫簡報之前需要更多細節時，`triage` 會一起執行 [grilling](https://aihero.dev/skills-grilling) 與 [domain-modeling](https://aihero.dev/skills-domain-modeling)，一次一回合問題，所以它會在你做出決策時，把它們記錄在 `GLOSSARY.md` 與 ADR 中。當你不確定自己在哪一條車道時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
