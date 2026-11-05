## 它做什麼

`grilling` 是一個訪談迴圈，在任何人據此行動之前，對一個計畫、一個決策或一個想法進行壓力測試。它把主題畫成一棵**設計樹（design tree）**：每一個決策都會分支出依附在它之下的決策，並逐個分支地訪問你，直到沒有任何假設是沒說出口的。

它不會一次只問一個問題，也不會一次問完所有問題。每個**回合（round）**問的是整個**前緣（frontier）**：每一個前置條件都已確定的決策，此外什麼都不問。如果一個問題依賴另一個，這兩個問題絕不會在同一回合；一個取決於仍未回答之答案的問題，屬於之後的回合。你的回答確定了決策，前緣往外推進，下一回合就問那些因此解鎖的問題。十三個問題通常會在大約三個回合內問完，而不是十三個回合。

## 何時使用

輸入 `/grilling`，或者當任務適合時，[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會自己使用它。它是拷問系列中唯一一個模型觸發的 [skill](https://www.aihero.dev/ai-coding-dictionary/skill)，所以你很少會輸入它。通常是你*確實*輸入了的某個 skill 替你執行它。

直接輸入 `/grilling` 只會得到單純的訪談，沒有其他東西。當你想要更多時：

| 你手上有什麼 | 使用 |
| --- | --- |
| 你不是在一個工作目錄中工作 | [grill-me](https://aihero.dev/skills-grill-me)：同樣的 [session](https://www.aihero.dev/ai-coding-dictionary/session)，但用一個 agent 永遠不會自己觸發的名稱 |
| 你在一個工作目錄中 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：同樣的 session，而且它會在進行中寫入 `GLOSSARY.md` 與 ADR |
| 一項大到一個 session 裝不下的工作 | [wayfinder](https://aihero.dev/skills-wayfinder)：它繪製一張地圖，並在決策 ticket 中執行拷問 |
| 一個談話無法解決的問題：某個東西應該長什麼樣子或給人什麼感覺 | [prototype](https://aihero.dev/skills-prototype)：做出用完即丟的版本，然後回來 |
| 你自己的某個 skill 需要一場訪談 | 從它觸發 `/grilling`，而不是再寫一套訪談 |

## 回合、前緣，以及誰做決定

這個 skill 建立在三個概念之上。

**設計樹**是主題的模型：決策下面掛著決策。**前緣**是所有前置條件都已確定的決策集合：也就是 agent 目前唯一能問的問題。一個**回合**就是一個前緣，完整地問、完整地回答。

在一個回合中，每個問題都有固定的格式：在 `❓` 後面編號並加上標題，然後是內容，接著是 agent 建議的答案，單獨放在一行 `➡️` 上。這個格式讓你可以用編號回答整個回合（「1 是，2 第二個選項，3 不是，原因如下」），而不需要把問題引用回去。這個格式有一個已知的問題。建議的答案有時會*反對*問題原本的措辭，所以同意建議就等於對問題回答「否」。當這種情況發生時，就針對建議回答，並說明這一點。

設計的另一半是事實與決策的區分。事實是 skill 自己的工作。當一個前緣問題需要某個[環境](https://www.aihero.dev/ai-coding-dictionary/environment)能確定的東西時，agent 會派出一個[子代理（sub-agent）](https://www.aihero.dev/ai-coding-dictionary/subagent)去查，而不是問你。回合不會等那個研究；只有依賴它的問題才會等。決策屬於你，agent 必須等待它們。一個執行 `grilling` 卻自己回答決策的 agent，就破壞了這個 skill。當前緣清空時，session 就結束了。接著 agent 會等你確認你們已有共同的理解，然後才根據你們約定的內容採取行動。

前緣有一個限制。agent 是憑判斷選擇前緣的；它不是從一張圖計算出來的。所以它可能把兩個問題放在同一回合，之後才發現其中一個答案本該改變另一個。唯一的防護就是告訴它。這會在下一回合重新打開受影響的分支。

## 這裡涵蓋什麼，包裝層涵蓋什麼

這一頁涵蓋的是機制。包裝 skill 的頁面回答人們最常問的問題。

| 問題 | 在哪裡回答 |
| --- | --- |
| 設計樹、前緣、回合、問題格式、事實 vs 決策 | 這裡 |
| 一個 session 應該持續多久、碰到談話無法回答的問題該怎麼辦、如何避免一路點頭 | [grill-me](https://aihero.dev/skills-grill-me) |
| 什麼會被寫進 `GLOSSARY.md`、什麼會變成 ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |

## 常見問題

**我可以改回一次問一題嗎？**
可以，而且很大一部分的使用者都這麼做。把這段加到你的全域 `CLAUDE.md`：

```
When grilling, ask one question at a time.
```

不是每個人都偏好以回合為基礎的預設方式。閱讀速度較慢、使用第二語言工作，或靠依序的格式來幫助專注的使用者，回報一次一題對他們比較有效。這個退出選項是一個被支援的設定，而不是變通方法。

**`/batch-grill-me` 去哪了？**
併入了這個 skill。以回合為基礎的提問曾短暫作為一個獨立的 skill 發布，之後移進了 `grilling` 本身，所以所有建立在這個基本元件上的東西（`grill-me`、`grill-with-docs`、`triage`、`wayfinder`）都一次得到了它。沒有 `batch-grill-me` 可以安裝，也沒有獨立的依序 skill；上面那行 `CLAUDE.md` 就是回到一次一題的方法。

**一次問完整個回合，一定會漏掉那些我先前的答案本來會引出的問題吧？**
這是對回合最常見的反對意見，而前緣回答了它。一個回合只包含彼此不互相依賴的問題，所以一個回合中的任何答案都不可能讓同一回合的另一個問題失效。你的答案仍然會改變它們之後的內容，因為 agent 會從頭推算每一個下一回合。你確實會失去一些東西，但比「一次問完所有問題」所暗示的要少。參見上面提到的前緣的限制。

**它問完了問題，就開始動手建置了。**
確認關卡就是為此而存在的。這個 skill 是在你說理解已經一致時才算完成，而不是在前緣清空時。比較弱、比較快的[模型](https://www.aihero.dev/ai-coding-dictionary/model)仍然會跳過這道關卡。使用者最常在低推理強度或非前沿模型上回報這個問題，這些模型會把「訪談直到達成共同理解」壓縮成幾個問題加上一份大綱。如果你的模型會這樣，可靠的修正方式是在你自己的 `AGENTS.md` 或 `CLAUDE.md` 中加一行，告訴 agent 未經允許不要實作。

**它自己回答了自己的問題，而不是問我。**
那是這次執行中的一個 bug，也是 skill 的文字要區分事實與決策的原因。最常發生在另一個 skill 執行 `grilling` 來解決一張 ticket 時，agent 把那個任務當成可以繼續往前走的許可。同樣的限制也是為什麼沒有非同步模式。有人要求一個讀取 GitHub issue 並發布一份整合決策備忘錄的變體，那是另一個 skill，因為一場沒有人回答的拷問 session，產出的是 agent 的意見，而不是你的。

**我能限制問題的數量嗎？**
不能，而且上限是刻意排除在範圍之外的。有些計畫需要三個問題，有些需要五十個；一個固定的上限要嘛會截斷困難的情況，要嘛在簡單的情況下顯得武斷。預期的控制方式是白話。叫它收尾，或停下來接受計畫目前的樣子。如果一個 session 進行得非常久，原因通常是範圍太大了；把工作拆開，逐一拷問各個部分。

**我單獨安裝了 `grill-me`，結果什麼都沒發生。**
`grill-me` 是一個只有一行的 skill，它的全部內文就是「執行一場 `/grilling` session」，所以它也需要安裝這個 skill。`grill-with-docs` 也是一樣，而它還額外需要 [domain-modeling](https://aihero.dev/skills-domain-modeling)。安裝整套就能避免這個問題；選擇性安裝則代表也要安裝那些基本元件。

**`grill-with-docs` 執行了，但它從沒載入 `grilling`。**
這是一個真正的 bug，至今仍未修正，而使用者在各種 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 與模型上都回報過。一個指名另一個 skill 的 skill，並不能可靠地讓那個 skill 被載入，而 `grill-with-docs` 指名了兩個。徵兆是一個一次問完所有問題、沒有任何建議的 session。這代表模型是在即興發揮一場訪談，而不是執行這一個。直接問 agent 它是否載入了 `grilling` 與 `domain-modeling`，通常就能恢復。

## 它有效的跡象

- 一個回合以編號清單的形式出現，每個問題的建議答案都放在單獨一行 `➡️` 上，而你可以用編號回答整個回合。
- 一個回合中的任何問題，都不需要先回答同一回合的另一個問題。
- 之後的回合會問第一回合不可能問的東西。
- 它會去查事實（讀檔案、派出子代理），而不是問你它自己就能查到的東西。
- 在背景執行的研究不會讓回合停下來；只有依賴它的問題才會等。
- 它在最後會停下來，請你確認理解是一致的，而不是開始工作。
- 問題數量保持很高，而回合數量保持很低。

## 它的定位

`grilling` 是一個**基本元件（primitive）**，而不是一個你要排進流程的步驟：它是訪談技巧的單一事實來源，保存在同一個地方，讓每一個需要訪談的 skill 都使用它，而不是自己發明一套。[grill-me](https://aihero.dev/skills-grill-me) 與 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 是它的兩個使用者觸發的入口，而 `grill-with-docs` 是主要建置鏈的起點，在 [to-spec](https://aihero.dev/skills-to-spec) 之前。[wayfinder](https://aihero.dev/skills-wayfinder) 執行它來解決決策 ticket，[triage](https://aihero.dev/skills-triage) 用它把一份模糊的回報拷問成一份可處理的回報，而 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在你挑選了一個要加深的候選項目後，用它走過決策樹。當你不確定哪個入口適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
