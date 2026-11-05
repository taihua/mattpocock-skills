## 它做什麼

`implement-spec` 接收一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 與它的 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)，並在一次執行中把整件事落地。負責統籌的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 把每張 ticket 交給一個在自己 git worktree 中工作的實作[子代理（subagent）](https://www.aihero.dev/ai-coding-dictionary/subagent)，把每個完成的分支合併進單一個**整合分支（integration branch）**，對結果執行 [code-review](https://aihero.dev/skills-code-review)，並解決這些 ticket。

它把 ticket 讀成一張**任務圖（task graph）**，而不是一份清單。阻擋關係決定了什麼可以開始，所以在任何時刻都有一個**前緣（frontier）**，由阻擋者都已落地的 ticket 組成，而前緣上的每張 ticket 會同時執行。這就是它與逐張處理 ticket 的差別。決定步調的是圖的形狀，而不是 ticket 在 tracker 上的順序。

## 何時使用

你透過輸入 `/implement-spec` 來觸發它，agent 不會自己去用它。

| 你的情況 | 使用 |
| --- | --- |
| 一份已拆成 ticket 且有阻擋關係的 spec，你想一次執行就落地 | `/implement-spec` |
| 一次一張 ticket，在你自己的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) 中，每張之間[清空（clear）](https://www.aihero.dev/ai-coding-dictionary/clearing) | [implement](https://aihero.dev/skills-implement) |
| 一份還沒拆成 ticket 的 spec | 先用 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一小塊沒有真正圖結構的工作 | 直接用 [implement](https://aihero.dev/skills-implement) |

## 前置條件

- **一個 issue tracker。** 這個 skill 從 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 設定好的 tracker 讀取 ticket，也在那裡解決它們。如果還沒有設定任何 tracker，它會停下來並告訴你先執行設定，而不是用猜的。
- **帶有阻擋關係的 ticket**，就像 [to-tickets](https://aihero.dev/skills-to-tickets) 所寫的那樣。沒有阻擋關係，圖就是扁平的，每張 ticket 都會同時開始。
- **一個能在背景執行子代理、並為每個子代理提供 git worktree 的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)。** 這個 skill 存在的目的就是同時執行多張 ticket，所以在一個一次只能執行一個子代理的 harness 上，它只是一個比較慢的 `implement`。

## 整合分支

一切都落在同一個分支上。每個實作者：

1. 在開始之前確認自己的 worktree 是以整合分支為基礎，
2. 用 [tdd](https://aihero.dev/skills-tdd) 建置它的 ticket，一次一個紅燈、綠燈切片，
3. 在回報完成之前，把整合分支的最新狀態合併進自己的分支，讓落地成為一次快轉（fast-forward）。

tracker 決定是否要有 pull request。如果你的 tracker 透過 PR 結案工作，或你要求開一個，skill 會在第一次合併之後開一個 draft PR，並在最後把它標記為可審查。否則，執行會停在整合分支上，每張 ticket 都以你 tracker 結案工作的方式被解決，這在本地 markdown tracker 上可以完全離線運作。

實作者與統籌者之間透過[情境指標（context pointers）](https://www.aihero.dev/ai-coding-dictionary/context-pointer)溝通（spec、ticket、共用的探索筆記、先前的 commit），而不是貼上摘要。這讓每個子代理的提示保持精簡，並在統籌者的視窗中留下空間給這張圖。

## 常見問題

**這和我自己對每張 ticket 執行 `/implement` 有什麼不同？**

這正是這個 skill 存在要回答的問題。在它發布之前，人們一直在建立自己的版本，一位使用者這樣描述這個需求：他們想要「讓子代理實作 ticket」，而不是「當一份 spec 可能包含超過 5 張 ticket 時，得一張一張地個別建立新 session 並叫它們實作」。使用 `implement` 時，你就是調度者：每張 ticket 一個 [session](https://www.aihero.dev/ai-coding-dictionary/session)，中間清空，還要自己追蹤哪些 ticket 已經解除阻擋。`implement-spec` 把這份工作交給一個統籌 session。代價是你不再逐一閱讀每張 ticket 落地時的工作；你在最後審查整合分支。要開始一次執行，清空情境，然後輸入 `/implement-spec` 並附上指向 spec 的指標（issue 編號或檔案路徑）。對於一個沒有真正圖結構的小變更，就跳過它，直接用 `implement`。

**它需要 GitHub 嗎？我希望它停在分支上。**

不需要，已經不需要了。一位喜歡開發中版本的使用者正是有這個抱怨：「它在最後會建立一個 PR，這需要像 GitHub 這樣的線上 repo。我希望它能離線完成同樣的工作，並停在所有工作都已合併的那個分支上。」現在執行會結束在整合分支上。只有在設定好的 tracker 透過 PR 結案工作，或你要求開 PR 時，才會開 PR，所以在本地 markdown tracker 上，執行結束時每張 ticket 都已解決，而工作已合併在分支上。

**它的審查與修正迴圈跑了好幾個小時，或一直在「修正」還沒建置的 ticket。**

兩者都發生在 `code-review` 在 skill 指定的那個時間點之外執行時。它把程式碼與整份 spec 比較，所以只有在每張 ticket 都落地之後才有意義。如果它在執行中途執行，每一張還沒建置的 ticket 都會被讀成失敗。agent 接著就去建置那張 ticket，而這又觸發另一次審查。在最後，skill 會執行一次 `code-review`，並把每一項發現送給一個修正子代理，但它還沒有說明修正之後何時該停止。一位使用者回報一個五張 ticket 的功能，「審查與修正迴圈大約花了四個小時」。如果你看到第二次大範圍的審查開始，就告訴它針對已修正的發現執行有焦點的檢查，然後停止。要預期第一次審查會找到真正的問題。這次執行的產出是一份由審查來完成的草稿，而不是可以單獨交付的東西。

**它會像 implement 一樣驅動 tdd 嗎？**

現在會，雖然一開始不會。執行開發中版本的使用者注意到「實作子代理不會繼承 /tdd 指令」，所以一從一張 ticket 擴大到整份 spec，紅燈、綠燈就停了。現在每個實作者都會用 `tdd` 建置它的 ticket。但仍然沒有像 `implement` session 那樣互動式約定接縫的步驟，所以如果你想把接縫固定下來，就在 spec 或 ticket 中指名它們。

**兩個平行執行的實作者在同一個檔案上撞在一起，或對同一個東西取了不同的名字。**

worktree 並不會消除衝突；它們只是把衝突延後到合併的時候。從 ticket 文字寫出的阻擋關係，只是對每張 ticket 會碰到哪些檔案的一種猜測，而兩張處理「程式碼庫不同部分」的 ticket 仍然會共用一份訊息目錄、一個設定註冊表或一個型別。每個實作者只看得到自己的 ticket 與共用筆記，從看不到另一個人進行中的工作，所以一位使用者的 web 與行動版 ticket，把同一個字串分別加成了 `blockedSince` 與 `blockedOn`。當前緣上的兩張 ticket 會碰到同一個共用檔案時，要嘛在它們之間加一條阻擋關係讓它們依序執行，要嘛讓探索筆記把每張 ticket 要新增的確切名稱定下來。

**被阻擋的 ticket 永遠不會開始，即使它的阻擋者已經合併了。**

這是 GitHub 上的已知問題。tracker 的「被阻擋數」只有在阻擋者*關閉*時才會減少，而 ticket 通常是在 PR 合併時才關閉，也就是執行結束的時候。tracker 是起始圖的正確來源，但在執行中途它是過時的。告訴統籌者自己追蹤哪些 ticket 已經合併進整合分支，並據此計算前緣。

**這會取代 Sandcastle 或 AFK 腳本嗎？**

不會。人們會問，是因為這些 skill 現在已經延伸到實作了：「Sandcastle 還有用嗎？你的 skill 現在似乎也能處理實作了。」`implement-spec` 讓一個 agent 在一個 harness session 內負責統籌，這不需要任何基礎設施，而且讓你可以觀看與引導。對於真正 [AFK（離線）](https://www.aihero.dev/ai-coding-dictionary/afk)的工作，一個確定性的迴圈（[Sandcastle](https://github.com/mattpocock/sandcastle)、一個 shell 腳本、一個 CI job）更快、更便宜、更可靠，因為沒有任何 agent 在做統籌決策。

**某張 ticket 的關鍵測試在它的 worktree 中被跳過了，而它回報為綠燈。**

worktree 只包含 git 追蹤的東西。讀取被 gitignore 的測試資料、本地資料庫或憑證的測試，可能會在那裡默默被跳過。對於驗證依賴未追蹤資料的 ticket，告訴統籌者改在主要的 checkout 中執行它。

## 它有效的跡象

- 只要圖允許，就會有好幾個實作者同時在執行，而不是一個接一個。
- 一張 ticket 在它最後一個阻擋者落地到整合分支時就立刻開始，而不是等整次執行結束。
- 每張 ticket 的追蹤紀錄都顯示 `tdd` 有在執行，在程式碼之前先有一個失敗的測試。
- 合併到整合分支都是快轉，而不是衝突解決。
- 執行結束在一個分支上，每張 ticket 都已解決，只有在你的 tracker 需要時才有 PR。

## 它的定位

`implement-spec` 是主鏈的建置步驟，作為每張 ticket 各執行一次 [implement](https://aihero.dev/skills-implement) 的平行替代方案：

```txt
grill-with-docs → to-spec → to-tickets → implement-spec → retro
```

它的鄰居是 [to-tickets](https://aihero.dev/skills-to-tickets)，後者宣告它讀成任務圖的阻擋關係；以及 [code-review](https://aihero.dev/skills-code-review)，它在收尾前對整合分支執行後者。當你不確定自己處於哪個流程時，[ask-matt](https://aihero.dev/skills-ask-matt) 是整套 skill 的路由器。
