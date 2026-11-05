> **已封存。** 這個 skill 已在 v1.3.0 從 plugin 中移除，不再維護。沒有任何東西取代它：agent 不需要專用的 skill 也能處理 merge 或 rebase 的衝突。這個頁面保留作為參考。

## 它做什麼

`resolving-merge-conflicts` 會逐個 hunk 地處理一個進行中的 git merge 或 rebase，然後執行專案自己的檢查，並以一個 commit 完成這次操作。

它拒絕把衝突當成文字問題。在動任何 hunk 之前，它會把兩邊各自追溯回它的**[第一手資料（primary source）](https://www.aihero.dev/ai-coding-dictionary/primary-source)**（commit 訊息、PR、原始 issue），所以它是在兩種意圖之間做選擇，而不是在兩段文字之間做選擇，並在兩者相容的地方同時保留兩者。在不相容的地方，它會選擇符合這次合併既定目標的那一邊，並說明其中的取捨。它不會發明新的行為來掩蓋衝突，也從不使用 `--abort`。它一定會把合併帶到一個完成的 commit。

## 何時使用

輸入 `/resolving-merge-conflicts`，或者當任務適合時，[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會自動使用它。

在 git 已經因為它自己無法解決的衝突而停下來時使用。它的範圍是你眼前的衝突，不包括衝突兩側的任何事：

| 你的情況 | Skill |
| --- | --- |
| merge 或 rebase 進行到一半，樹中有衝突標記 | 就是這個 |
| merge 完成了，但現在某個東西因為你看不出的原因出了問題 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| 正在規劃如何切分工作，讓分支之間比較不會撞在一起 | 都不是：參見下方關於平行工作的問題 |

## 第一手資料優先於 `ours` 與 `theirs`

這個 skill 存在的目的，是防止用旗標解決衝突：`--ours`、`--theirs`，或手動刪掉看起來比較不重要的那一段，讓標記消失、讓建置能編譯。這樣的解決方式在語法上可以完美無缺，卻仍然默默丟掉了某個人刻意做出的變更。

你無法保留一個你沒讀過的意圖。所以工作從歷史紀錄（commit、PR、[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)）開始，之後才轉到 diff。檢查步驟也是出於同樣的理由。這個 skill 會找出 repo 自己的[自動化檢查](https://www.aihero.dev/ai-coding-dictionary/automated-check)，並在 commit 之前執行它們，因為 merge 是 git 中最容易產生「同時滿足兩個分支、卻兩邊的測試都沒過」的程式碼的地方。

## 常見問題

**Claude Code 自己就能把衝突處理得不錯了。為什麼還需要一個 skill？**

它的附加價值是「找出第一手資料」與「執行回饋迴圈」這兩個步驟，否則你每次都得手動提示。沒有提示的 agent 通常只會根據 diff 產生一個看似合理的解決方式，然後就停在那裡。這個 skill 的價值，在於它不讓 agent 跳過的兩個步驟：了解兩邊各自存在的理由，以及事後執行檢查。相較於一個好的[模型](https://www.aihero.dev/ai-coding-dictionary/model)，這是一個小小的增益，而這是刻意的。至少有一位讀者預測，隨著模型進步，這整個 skill 會變成一個 no-op。

**我應該讓平行的 agent 不碰同樣的檔案，從一開始就避免衝突嗎？**

大多數時候不需要。在平行任務之間劃分檔案的成本比它省下的還多，因為 agent 處理 merge 衝突已經夠好了，這個取捨並沒有看起來那麼嚴苛。唯一值得保留的紀律是先做大型重構。在十個分支已經從某處分岔出去之後，才落地一次大型的重新命名，這種情況才會一直很昂貴。

一位使用者在平行 worktree 上回報了一個注意事項：當兄弟 [session](https://www.aihero.dev/ai-coding-dictionary/session) 各自在自己的樹中建置一張 ticket 時，合併回去最好由撰寫該變更的 session 來做，因為它就是那個已經知道意圖的 session。如果最後由一個 agent 解決所有人的衝突，它就會失去[情境（context）](https://www.aihero.dev/ai-coding-dictionary/context)，而這個 skill 的步驟 2 就必須把它重建回來。

**為什麼絕不 `--abort`？**

中止會丟掉解決衝突的工作，下次你再嘗試時，會回到同樣、完全沒變的衝突。這個 skill 是為「這次合併一定會發生」的情況而寫的。如果你已經決定它不應該發生，那是一個在觸發之前就該做的決定，而不是迴圈中的一個分支。

## 它有效的跡象

- agent 在解決衝突時會引用 commit 訊息、PR 或 issue 給你看，而不只是 diff 的 hunk。
- 每個 hunk 最後都同時保留了兩邊的行為，或附上一個明確的說明，指出丟掉了什麼以及為什麼。
- 結果中沒有出現任何兩個分支都沒有的東西。
- agent 找到了型別檢查、測試與格式化，並在 commit *之前*把它們跑到全綠，而不是在你發現有東西壞掉之後。
- 最後是一棵乾淨的樹，而且操作已經完成，包括多 commit 的 rebase 中剩下的每一個 commit。

## 它的定位

一個隨時都能使用、不依賴任何其他 skill 的獨立 skill：它在 git 停下來時開始，在樹乾淨且已 commit 時結束。它唯一真正的鄰居是 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)，在 merge 乾淨地解決了、但合併後的程式碼行為異常時接手：那是一個診斷問題，而不是衝突問題。它不在主要的「想法到出貨」流程上，所以 [ask-matt](https://aihero.dev/skills-ask-matt) 是了解它前後該執行什麼的地圖。
