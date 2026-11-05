## 它做什麼

`implement` 建置已經決定好的工作。你把它指向一張 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，或你們剛在對話中約定好的計畫，它就會寫程式碼、在接縫上驅動 [tdd](https://aihero.dev/skills-tdd)、一邊進行一邊做型別檢查、在最後執行 [code-review](https://aihero.dev/skills-code-review)，並 commit 到目前的分支。

它從不重新打開計畫。沒有訪談、沒有釐清的回合、沒有提出不同的做法。上游確定的東西就是輸入，而這個 skill 全部的工作就是把它變成一個 commit。這就是它與對一個全新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 輸入「把這個做出來」的差別，後者常常在建置的同時重新設計整個工作。

## 何時使用

你要自己輸入 `/implement` 來觸發它，agent 不會自己去用它。它帶有 `disable-model-invocation: true`，所以也沒有其他 skill 能呼叫它。凡是 [ask-matt](https://aihero.dev/skills-ask-matt) 或 [to-tickets](https://aihero.dev/skills-to-tickets) 說「然後每張 ticket 執行 `/implement`」的地方，那是給你的指示，而不是 agent 會自動去做的事。

工作目前存在哪裡，決定了這是不是正確的 skill：

| 工作是…… | 使用 |
| --- | --- |
| tracker 上的一張 ticket | `/implement #42`，每個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 一張 ticket，每張之間[清空](https://www.aihero.dev/ai-coding-dictionary/clearing)情境 |
| 一份還沒拆分的 spec，而建置會橫跨多個 session | 先用 [to-tickets](https://aihero.dev/skills-to-tickets)，再每張 ticket 執行 `/implement` |
| 一份 spec，而建置很小 | 直接對 spec 執行 `/implement` |
| 只存在於你剛進行的對話中，而且仍然很小 | 就在這裡、在同一個視窗中執行 `/implement` |
| 還沒有寫在任何地方 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，如果沒有程式碼庫，就用 [grill-me](https://aihero.dev/skills-grill-me) |
| 一個你想以測試先行方式建立的具體行為，沒有 spec | 直接用 [tdd](https://aihero.dev/skills-tdd) |
| 已經建好了，你想檢查它 | 直接用 [code-review](https://aihero.dev/skills-code-review) |

同一個 session 的情況值得特別提出，因為 skill 自己的第一行並沒有涵蓋它。`SKILL.md` 說的是「spec 或 ticket」，這會促使[模型](https://www.aihero.dev/ai-coding-dictionary/model)去找一個不存在的檔案。如果計畫只存在於討論串中，觸發時就說出來。

## 前置條件

`implement` 會 commit 到你目前所在的分支。它不會建立分支，也不會詢問。開始前請確認你在想要放這項工作的分支上。

如果 ticket 來自 [to-tickets](https://aihero.dev/skills-to-tickets)，它們所在的 tracker 是由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 設定的。`code-review` 在收尾時會讀取同一份設定，以找出原始的 spec。

## 一次執行做什麼

一次執行有五個步驟，依序是：

1. 閱讀 ticket 或 spec，找出接縫。
2. 在事先約定的接縫上驅動 [tdd](https://aihero.dev/skills-tdd)，一次一個紅燈、綠燈切片。
3. 經常做型別檢查，一邊進行一邊執行單一測試檔。
4. 在最後執行一次完整的測試套件。
5. 執行 [code-review](https://aihero.dev/skills-code-review)，然後 commit 到目前的分支。

一次執行涵蓋一張 ticket。[to-tickets](https://aihero.dev/skills-to-tickets) 產出的 ticket 是曳光彈式的垂直切片，大小剛好能放進一個全新的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)，所以預期的節奏是：清空情境、實作一張 ticket、commit、再清空。每張 ticket 都自成一體，所以你可以丟掉上一張 ticket 的情境。

## 事先約定的接縫

這個 skill 的核心概念是**接縫（seam）**，也就是你在不伸手進入內部的情況下觀察行為的公開邊界。測試存在於接縫上。當接縫在任何程式碼存在之前就約定好，測試就能長久存在，而你可以在不改動它們的情況下重寫底下的實作。

「事先約定」這一點很重要，而它也是這個 skill 最弱的地方。`implement` 內部沒有任何東西負責約定接縫。負責詢問的是 `tdd`，而它拒絕在未經確認的接縫上寫測試。所以實務上，約定要嘛發生在上游的 spec 中，要嘛發生在這次執行的第一次問答中。如果哪裡都沒有發生，這次執行就會變成「直接寫程式碼」，而沒有任何東西會警告你。在 spec 中指名接縫，就是阻止這件事的方法。

## 常見問題

**它完成了，但我的 ticket 還是開著，驗收條件也還沒打勾。**

正確，而且是預期中的。`implement` 沒有完成步驟。它在 commit 時結束，從不碰工作項目。這在 GitHub Issues 與本地 markdown tracker 上都一樣，所以它不是 tracker 整合的問題。它也不會處理 `code-review` 產出的發現，也不會勾選原始 issue 上的 `- [ ]` 方框。自己關閉 ticket 並核對條件。這在依賴鏈上最重要，因為 `to-tickets` 把前緣定義為所有阻擋者都已關閉的 ticket。如果什麼都沒有被關閉，就永遠不會有任何東西明顯地解除阻擋。

**我可以一次把它指向所有 ticket，或平行執行好幾個嗎？**

用 `/implement` 不行：一次觸發，一張 ticket。若要一次執行完整份 spec，請使用 [implement-spec](https://aihero.dev/skills-implement-spec)，它會把已就緒前緣上的每張 ticket 交給一個在自己 worktree 中的[子代理（subagent）](https://www.aihero.dev/ai-coding-dictionary/subagent)，然後把結果合併到同一個整合分支上。在同一個 checkout 中並排執行好幾個 `/implement` session，比不支援還糟。一份實地回報描述了在一個下午、橫跨三個 issue 的過程中：一個 session 中的 `git commit --amend` 落在另一個 session 的 commit 上、一個 stash 從 `refs/stash` 中消失，以及 commit 落在錯的分支上。這些 session 共用同一個工作目錄、同一個 index 與同一個 HEAD。使用者用 git worktree 來變通，但 `refs/stash` 在各個 worktree 之間也是共用的，所以單靠 worktree 並不能解決 stash 的問題。

**它能開一個 pull request，而不是直接 commit 嗎？**

沒有內建。它直接 commit 到目前的分支。好幾個人覺得這太急躁了，因為程式碼在他們能驗證它可運作之前就落地了。沒有設定旗標，也沒有 PR 模式。人們在觸發時覆寫它（「commit 到一個分支並開 PR」），或編輯他們本地的 skill 副本。當 agent 確實要寫 PR 時，[pr](https://aihero.dev/skills-pr) 會形塑它的內文。

**`code-review` 說它看不到我的變更。**

`code-review` 審查的是 `git diff <fixed-point>...HEAD`，這不包含已暫存（staged）與工作樹中的變更。`implement` 在 commit 之前執行它，所以除非已經存在一個中間的 commit，否則那個 diff 中沒有任何東西可以審查。好幾個人回報過這個問題，而兩邊都還沒修正。先 commit，再針對你分岔出來的那個點進行審查。

另外，有些人根本不想在執行中進行審查，因為一個審查自己剛寫的程式碼的 agent，會偏向它自己的解法。在一個新的 session 中針對一個固定點執行 [code-review](https://aihero.dev/skills-code-review)，是一個有效的替代方案。同樣的偏誤，也是那個 skill 把兩個軸放在不同子代理中執行的原因。

**一張 ticket 燒掉了 15 萬 token。是我用錯了嗎？**

大概不是。比較可能是 ticket 太大了。一次執行要探索程式碼庫、每個接縫一個紅燈、綠燈迴圈、一次完整的測試套件，以及一次審查，所以一張不簡單的 ticket 超過 10 萬個 [token](https://www.aihero.dev/ai-coding-dictionary/token) 是正常的，而不是有東西壞掉的跡象。修正方式在上游。在 [to-tickets](https://aihero.dev/skills-to-tickets) 中把 ticket 調整到合適的大小，讓每一張都放得進一個全新的視窗。如果某張 ticket 一直超過，就拆分它，而不是提高[推理強度（effort）](https://www.aihero.dev/ai-coding-dictionary/effort)等級。

**在一個新的 session 中執行 `/implement #2`，結果它處理了一件完全無關的事。**

agent 把 `#2` 解析成情境中另一份有編號的清單，例如一個待辦檔案或檢查清單，而不是設定好的 tracker。`implement` 現在會從 issue tracker 取得傳入的引用，並在開始之前說出它的標題，在引用有歧義時也會詢問。檢查那個標題是否符合你指的 ticket；傳入 issue 的 URL 或 `owner/repo#2` 就能完全消除歧義。

## 它有效的跡象

- session 一開始就閱讀 ticket 或 spec，並重述它將要建置什麼，而不是問你要建置什麼。
- 你可以在追蹤紀錄中看到一次實際的 `tdd` Skill 工具呼叫，而不只是在 diff 中看到測試出現。
- 在執行期間，型別檢查與單一測試檔會反覆執行，完整的測試套件則在接近結尾時執行一次。
- 執行會在不需要你提示它繼續的情況下，抵達目前分支上的一個 commit。
- diff 剛好是一張 ticket 份量的變更：一個穿過每一層的垂直切片，而不是好幾張 ticket 混在一起。

## 它的定位

`implement` 是主鏈的建置步驟：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它的鄰居是 [to-tickets](https://aihero.dev/skills-to-tickets)，後者產出它所消費的 ticket，並宣告決定它們順序的阻擋關係；[tdd](https://aihero.dev/skills-tdd)，它在每個接縫上於內部驅動它；以及 [code-review](https://aihero.dev/skills-code-review)，它在 commit 之前執行它。它位於規劃類 skill 的下游，並信任它們。它不會重新驗證交給它的東西的形狀，所以一張結構不良的地圖或一張水平分層的 ticket，會照著寫的樣子被建置出來。

這份信任就是為什麼 [wayfinder](https://aihero.dev/skills-wayfinder) 要在 [to-spec](https://aihero.dev/skills-to-spec) 匯入這條鏈，而不是把它的地圖直接接到 `implement`。只有在工作最後證明很小時，才從地圖直接進入 `implement`。

當你不確定自己處於哪個流程時，[ask-matt](https://aihero.dev/skills-ask-matt) 是整套 skill 的路由器。
