## 它做什麼

`research` 透過閱讀擁有答案的來源來回答一個問題，然後在 repo 中留下一份附引用的 Markdown 檔。它只根據**[第一手資料（primary sources）](https://www.aihero.dev/ai-coding-dictionary/primary-source)**工作：官方文件、原始碼、規格、第一方 API。它把每一個主張都追溯回擁有它的來源，所以當 API 自己的文件可以取得時，它不會轉述某篇部落格文章對那個 API 的說法。

它不會在對話中回答你。產出是一個檔案，寫在 repo 原本就存放這類筆記的地方，每一個主張都附有連結。你得到的是一份可以回應、可以交給另一個 agent，或可以丟掉的文件，而不是一個在 [session](https://www.aihero.dev/ai-coding-dictionary/session) 結束後就消失的答案。

## 何時使用

輸入 `/research`，或者當任務變成閱讀的跑腿工作時，[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會自動使用它。

當下一步是從工作目錄之外*找出某件事*（某個第三方 API 怎麼運作、某份規格怎麼說、某個版本聲明是否成立），而你不想讓自己的討論串因為閱讀而停下來時，就使用它。你需要什麼決定了用哪個 skill：

| 你需要什麼 | 使用 |
| --- | --- |
| 一個某個決策正在等待的外部事實 | `research` |
| 一個透過訪談、*與你一起*做出的決策 | [grilling](https://aihero.dev/skills-grilling) |
| 一個持久的架構決策，寫進 `GLOSSARY.md` 與 ADR | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 找出某個做法在你的程式碼庫中是否可行 | [prototype](https://aihero.dev/skills-prototype) |
| 一個大到一個 session 裝不下的計畫 | [wayfinder](https://aihero.dev/skills-wayfinder) |

`research` 與 `grill-with-docs` 之間的界線，在於**回來的東西的保存期限**。研究產出的是短命的事實，例如這個函式庫的認證機制在這一週是怎麼運作的。ADR 記錄的是你要保留的決策。如果你產出的是一個決策而不是一個事實，那你就是在[拷問（grilling）](https://www.aihero.dev/ai-coding-dictionary/grilling)，而不是在研究。

## 委派出去的跑腿工作

閱讀以一個**背景 agent** 的形式執行。在它把每個主張追溯到第一手資料、寫出一份 Markdown 檔並回報的同時，你可以繼續工作。研究是你委派出去的跑腿工作，而不是外包出去的思考。你得到一份可以拿來拷問、規劃或設計的文件，而決策仍然由你來做。

沒有任何東西能阻止背景 agent 自己再派出另一個背景 agent。這是這個 skill 記錄最完整的問題。

檔案放在哪裡由 repo 決定，而不是 skill。它遵循任何已經存在的筆記慣例。如果沒有，它會挑一個合理的地方並告訴你放在哪裡。每次執行寫一個檔案。

## 常見問題

**它派出了第二個研究 agent。這是應該發生的嗎？**

不是。這是一個開放中的 bug，[issue #530](https://github.com/mattpocock/skills/issues/530)。skill 告訴它的呼叫者去啟動一個背景 agent，但沒有限制 agent 的類型。所以呼叫者派出一個 `general-purpose` agent，而它有 `Agent` 工具和同樣的指示，於是又照做一次。一位回報者量測到，一個研究任務在三次重疊的執行中花了大約 45 萬個 [token](https://www.aihero.dev/ai-coding-dictionary/token)，而重複的那個在半小時後才在使用者看不到的地方完成。它在 Claude Code 之外也會發生。使用者在 Codex 搭配 GPT-5.6-sol 時也確認了同樣的巢狀問題。目前沒有已發布的修正。使用者在自己安裝的副本中加了一行，告訴已經是[子代理（subagent）](https://www.aihero.dev/ai-coding-dictionary/subagent)的 agent 自己完成工作。這有幫助，但它只是一條指示，而不是結構性的修正。觸發這個 skill 之後，留意你的背景任務清單，並停止重複的那一個。

相反的失敗也會發生。如果你自己的全域指示禁止 agent 再次委派工作，背景 agent 就會拒絕這個任務，而這個 skill 會在不告訴你的情況下什麼都不做。

**檔案應該放在哪裡？我應該 commit 它嗎？**

skill 把檔案放在 repo 原本存放筆記的地方，除此之外沒有其他意見。社群的看法大致已經確定：保留 ADR，而不是研究檔案。Discord 上一個關於這個問題的討論串說得最清楚：「ADR 要留。其他東西做完之後就封存或刪除。不然它會變成工作的殘渣，如果你已經偏離了 spec 或研究，它還可能汙染之後對 repo 的讀取。」研究檔案記錄的是它被寫下那天的真相，所以一份過時的研究檔案比沒有還糟。整體而言，這些檔案不屬於 git，而且沒有標準的存放位置。人們改用 Obsidian、一個獨立的知識 repo，或 issue tracker。

**什麼算是「高可信度」的第一手資料？由誰決定？**

由[模型](https://www.aihero.dev/ai-coding-dictionary/model)決定。skill 指名了符合資格的來源*種類*（官方文件、原始碼、規格、第一方 API），但沒有允許清單、沒有網域關卡，也沒有驗證程序。這是這個 skill 剛被提出時最大聲的反對意見，而至今沒有人公開回答過：「五個指向垃圾資料的研究子代理，只會讓你更快得到五個充滿自信的錯誤答案。你要怎麼把關什麼才算高可信度的來源？」你唯一的保護是每個主張上的引用。追兩三個看看。如果它們指向的是某樣東西的摘要，而不是那樣東西本身，這次執行就失敗了。

**之後的 session 會重複利用先前執行找到的東西嗎？**

不會。沒有任何東西會自動載入過去的研究檔案。檔案會留在 repo 中，直到有人或某個 skill 指向它。這是這個設計早期遇到的最強挑戰：「價值在於 markdown 成為 agent 之後會重新讀取的情境，而不是擷取本身。一個寫完就沒人理的死檔案，只是一個比較花俏的搜尋。」已發布的 skill 並沒有解決這個問題。實務上，只有當你自己把檔案餵進下一步時，它才有用：把它附在一份 spec 上、在拷問 session 中引用它、讓一張 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 指向它。

**為什麼不直接叫 agent 去讀文件就好？**

你可以，而一個剛好這麼說的兩行提示，正是這個 skill 所取代的做法。這個 skill 給你兩件提示給不了的東西。它在背景執行，所以你 session 的[情境](https://www.aihero.dev/ai-coding-dictionary/context)保持乾淨。而且第一手資料的限制與附引用的檔案輸出每次都一樣，而不是取決於你怎麼寫提示。與某個 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 自己的深度研究模式相比，差別在於檔案與第一手資料規則，而不是搜尋本身。如果一個兩行提示就能在一個小問題上給你需要的東西，就用那個兩行提示。

**它什麼時候會停止閱讀？**

這個 skill 沒有停止條件。這表現為兩種看似相反、但原因相同的抱怨：agent 鑽得太深，以及 agent 廣泛涵蓋了一個主題，卻漏掉了真正重要的那一個細節。一位實踐者這樣說：「深度研究的 skill 有時候有點太深了。而叫 agent 去研究，通常會漏掉關鍵細節。」你必須自己設定範圍。一個狹窄、可回答的問題（一個 API、一個行為、一個版本聲明）回來的結果，會比「研究一下 X」好得多。

**`/wayfinder` 建立了研究 ticket。我要自己解決它們嗎？**

不用，它現在會替你觸發它們。在 v1.1 之後尚未發布的變更中，一次繪製地圖的 session 會為每張研究 ticket 派出一個 `/research` 子代理，並平行執行它們。每一個都把它的發現記錄在一個用完即丟的 `research/<name>` 分支上，並從 ticket 放一個[情境指標](https://www.aihero.dev/ai-coding-dictionary/context-pointer)指向它。研究 ticket 是 wayfinder「每個 session 只處理一張 ticket」規則的唯一例外，因為它們是 [AFK（離線）](https://www.aihero.dev/ai-coding-dictionary/afk)的：沒有任何東西在等你。一個已知的問題：之後刪除那個分支，會讓 ticket 中的情境指標失效。

## 它有效的跡象

- 你自己的 session 持續進行。如果你坐在那裡看它閱讀，就代表委派沒有發生。
- 剛好出現一個新的背景任務。第二個名稱幾乎相同的任務就是巢狀 bug。
- 出現一個新的 Markdown 檔，在 repo 原本存放筆記的資料夾中，而且 agent 告訴你路徑。
- 其中每一個主張都附有連結，隨機追兩個，會落在官方文件、規格或原始碼檔案本身，而不是某人對它的整理文章。
- 光靠這個檔案，你就能做出那個讓你卡住的決策，不需要自己回頭去查來源。

## 它的定位

`research` 是一個隨時都能使用的獨立 skill。它為思考類的 skill 提供養分，而不是建置鏈中的一步。你把它的檔案帶*進*流程中。[grilling](https://aihero.dev/skills-grilling) 與 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 在已經掌握事實時會問出更銳利的問題，而 [to-spec](https://aihero.dev/skills-to-spec) 可以根據它來綜整。[wayfinder](https://aihero.dev/skills-wayfinder) 是唯一直接觸發它的 skill。它用一個 `/research` 子代理來解決地圖上的每一張研究 ticket。整張地圖請見 [ask-matt](https://aihero.dev/skills-ask-matt)。
