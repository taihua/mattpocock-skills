## 它做什麼

`grill-me` 接收一個**粗略的想法**，並持續訪問你，直到你能對它做出承諾。你不需要一份想清楚的計畫才能開始，因為這個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 存在的目的就是產生一份計畫。它以**回合**的方式提問。每一回合就是整個**前緣（frontier）**，也就是所有前置條件你都已經確定的問題。所以它從不會問你一個取決於它還沒聽到的答案的問題。

它是**[無狀態的（stateless）](https://www.aihero.dev/ai-coding-dictionary/stateless)**。它不寫任何檔案，也不留下任何工作區。唯一的結果，是在你自己腦中一個更清楚的想法版本。

## 何時使用

你透過輸入 `/grill-me` 來觸發它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。在一個**全新的對話**中開始，而不是疊在一份你已經讓 agent 寫好的計畫之上。

只要你有一個值得認真對待的想法（一個功能、一個產品方向、一個商業決定、一篇文章），就使用它，而且要在你想清楚它涉及什麼之前很久就用。模糊不是等待的理由，因為這個 session 存在的目的就是消除模糊。如果你已經能精確地描述那件事，你就不需要拷問它。

三個拷問 skill 中你要用哪一個，取決於你眼前的情況：

- **任何東西、任何地方。** 使用 `grill-me`。它不需要 repo，也不寫任何檔案，而且主題不一定要是程式碼。
- **有一個要對齊的程式碼庫。** 使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。它是同樣的訪談，但它是[有狀態的（stateful）](https://www.aihero.dev/ai-coding-dictionary/stateful)：它會讀你的程式碼，並把學到的東西保存在 `GLOSSARY.md` 與 ADR 中。
- **大到一個 session 裝不下。** 使用 [wayfinder](https://aihero.dev/skills-wayfinder)。它把這項工作畫成一張地圖，並在其中執行拷問 session。

不要開啟[規劃模式（plan mode）](https://www.aihero.dev/ai-coding-dictionary/agent-mode)。規劃模式會讓 agent 急著產出一份計畫，但你要的是它一直問問題。

## 這是一場對話，不是一場面試

skill 負責提問，但範圍由**你**掌握。人們常常忽略這一點。這就是「把想法變成決策的 session」與「產出充滿自信的胡說八道的 session」之間的差別。

失敗模式是**被動**：連續四十題都回答「同意、同意、同意」，最後得到一份 agent 寫的、你只是點頭的計畫。它感覺很有生產力，因為它很長。但你什麼都沒決定，而結果看起來比實際上更確定。

主動代表要掌舵。對比你需要的還不夠細緻的問題提出反駁。在範圍開始漂移時說出來。回答「我不知道」，而且是認真的。這個 skill 是在幫助工程師，而不是取代工程師。結果的品質取決於你回答的品質，而不是問題的數量。

相反的錯誤確實存在，但比較少見：在訪談中待太久，永遠沒有開始寫程式碼。

## 可拷問與不可拷問

有些問題可以透過談話來回答。有些則不行，再怎麼拷問也到不了那裡。

「一個長表單還是三個頁面？」以及「這個互動應該給人什麼感覺？」是**不可拷問的**。你需要一個能讓你做出反應的東西，才能回答它們。碰到這種問題時，就停止拷問。用 [prototype](https://aihero.dev/skills-prototype) 做出一個用完即丟的版本，看看它，然後回來用一行字回答。

當你試圖用談話解決一個不可拷問的問題時，session 就會變得太長。agent 一直換句話說，你一直猜，而範圍隨著不確定性一起膨脹。

## 它有效的跡象

- 你對某件事表示不同意。一個你完全沒有反駁的 session，就是一個你不需要的 session。
- 問題分成幾個回合送來，而不是長時間一次一題，而且後面的回合明顯建立在你先前說過的話之上。
- 你最後走到一個意料之外的地方，因為某個問題讓你看見一個你一直在不知不覺中做出的決策。
- 在最後，你能向一個不在場的人為每一個選擇辯護。

## 常見問題

**我應該預期會有多少問題？我怎麼知道它什麼時候結束？**
算回合，而不是算問題。四個回合共四十六個問題是一個普通的 session。當前緣清空時它就結束了：它已經走訪過每一個分支，沒有任何東西被留下來當作未說明的假設。

**它問了我兩百個問題。哪裡出錯了？**
通常是範圍太大了。先請 agent 把工作拆成比較小的部分，然後逐一拷問。非常長的 session 也會漂進**[笨蛋區（dumb zone）](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**，此時 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) 已經滿到讓問題品質變差。

**我可以改回一次問一題嗎？**
可以。把這段加到你的全域 `CLAUDE.md`：

```
When grilling, ask one question at a time.
```

**如果我真的不知道答案怎麼辦？**
就直說。「我不知道」是一個真正的答案，而一個你回答不出來的問題，通常代表該做原型，而不是去猜。

**在撰寫 spec 之前要開一個新的 session 嗎？**
不用。這個 session 的價值就在於你剛建立的[情境（context）](https://www.aihero.dev/ai-coding-dictionary/context)。把同一段對話直接交給 [to-spec](https://aihero.dev/skills-to-spec)。

**模型有差嗎？**
比大多數 skill 更有差。拷問依賴的是[模型](https://www.aihero.dev/ai-coding-dictionary/model)自身對系統如何出錯的知識，所以使用你最好的模型。實作則大多是依循情境，所以在那裡用比較便宜的模型也沒問題。

## 它的定位

`grill-me` 是一個**可以在任何地方、針對任何東西執行的獨立 skill**。因為它是無狀態的，所以它是可攜的。它不需要 repo、工作區或設定，也不預設想法是關於軟體的。人們用它來做商業決策、寫作，以及決定下一步要做什麼：任何他們無法獨自想清楚的事情。

可攜性是它與 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 唯一的差別。那個 skill 執行同樣的訪談，但會讀取一個程式碼庫來對齊，並把學到的東西記錄成 `GLOSSARY.md` 與 ADR。兩者在底層都使用 [grilling](https://aihero.dev/skills-grilling) skill。`grill-me` 是不保存狀態的、使用者觸發的入口。

如果你拷問的東西最後證明是軟體，你可以把同一段對話交給 [to-spec](https://aihero.dev/skills-to-spec)，繼續進入建置流程（這是一個選項，而不是這個 skill 的重點）。當你不確定哪個流程適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
