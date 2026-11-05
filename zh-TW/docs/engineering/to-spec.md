## 它做什麼

`to-spec` 把你剛才進行的對話轉成一份 **[spec（規格）](https://www.aihero.dev/ai-coding-dictionary/spec)**，並以單一個 issue 發布到你的 issue tracker。

它不會訪問你。當你使用它時，決策已經做完了。所以它綜整已知的內容（來自討論串、程式碼庫、你的 `GLOSSARY.md` 與 ADR），而不會開啟新一輪的提問。spec 記錄的是你已經做出的決策。它不是做出新決策的地方。

## 何時使用

你透過輸入 `/to-spec` 來觸發它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。

當建置大到一個 agent [session](https://www.aihero.dev/ai-coding-dictionary/session) 裝不下、必須拆到好幾個 session 時使用它。這就是唯一的觸發條件：

| 你所在的位置 | 要執行什麼 |
| --- | --- |
| 你還什麼都沒決定 | 先用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 已經決定了，而且工作放得進一個 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement)：跳過 spec |
| 已經決定了，而且工作橫跨好幾個 session | `/to-spec`，然後 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一張 [wayfinder](https://aihero.dev/skills-wayfinder) 地圖已經清楚了 | `/to-spec #<map_issue>` |

## 前置條件

`to-spec` 會把 spec 發布成一個 issue，所以必須先用 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 為這個 repo 設定好 tracker 與分類標籤詞彙。兩種 tracker 都可以：像 GitHub 這樣真正的 tracker，或是 `.scratch/` 下的本地 markdown 檔案，後者不需要額外設定。

## Spec 是一份決策紀錄

spec 之所以存在，是因為 context window 會結束。你在[拷問（grilling）](https://www.aihero.dev/ai-coding-dictionary/grilling)時確定了很多事：解決方案的形狀、你爭論過的選擇，以及你刻意拒絕的東西。這一切都在一段你即將清空的對話中。spec 把它保存下來。

所以 spec 不驗證、也不決定任何事。它用你專案自己的詞彙記錄你所做的決定，讓一個新的 session 不用你再解釋一次就能接手工作。如果 spec 寫了你從沒說過的東西，那就是一個瑕疵。

## 先有接縫，再有文字

在寫任何東西之前，`to-spec` 會先草擬這個功能將在哪些**接縫（seam）**上被測試，並與你確認。它偏好既有的接縫勝過新的，並挑選它能找到的最高層接縫。一次變更理想的接縫數量是一個。

其他 skill 之後會使用這些約定好的接縫。[tdd](https://aihero.dev/skills-tdd) 只在你事先約定的接縫上工作。[code-review](https://aihero.dev/skills-code-review) 會拿 spec 審查 diff，所以一個沒有人同意過的接縫會以審查發現的形式出現。這兩條連結都經過這份文件。這就是為什麼你應該在這裡認真對待關於接縫的討論，而不是把它留到實作時。

## 常見問題

**`/to-prd` 去哪了？**
就是這個 skill，在 v1.1 改了名。現在通篇只使用「spec」這一個術語，舊的 `to-prd` slug 已經無法使用，所以請用新名稱重新安裝。舊的詞彙被 *spec* 與 *tickets* 這一對取代了。spec 是目的地以及確定它的那些決策。[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 則是抵達那裡的步驟。如果你改變方向，就刪掉未完成的 ticket，保留 spec。

**為什麼 spec 會被加上 `ready-for-agent` 標籤？我不想讓 agent 直接根據它來實作。**
這個標籤的意思是「不需要進一步分類」：這份文件已經完整到 agent 可以據以工作。它標記的是一份輸入，而不是一張工作單。但輪詢 `ready-for-agent` 的 [AFK（離線）](https://www.aihero.dev/ai-coding-dictionary/afk) agent 看不出這個差別。它們會試圖一次建完整份 spec，而不是接手 ticket 切片。這是這個 skill 被回報最多的問題。在它改變之前，請在你的 AFK agent 提示中明確排除父 spec，或在 `/to-tickets` 執行後移除這個標籤。

**為什麼不從拷問直接進入 `/to-tickets`，跳過 spec？**
很多時候你確實應該這樣做。只有在多 session 的工作上，spec 才值得它這一步。它的價值在於 ticket 是可拋棄的，而 spec 不是。每張 ticket 的大小是一個全新的 context window，之後就會被刪除或關閉，而 spec 則留下來，作為記錄它們背後理由的唯一地方。在單一 session 的變更上，這什麼都沒給你，你卻要付出一個額外的綜整步驟，讓[模型](https://www.aihero.dev/ai-coding-dictionary/model)有機會漂移。從拷問直接進入 `/implement`。

**我剛完成一張 wayfinder 地圖。我該餵給它什麼？**
給它主要的地圖 issue，`/to-spec #<map_issue>`，而不是個別的決策 ticket。[wayfinder](https://aihero.dev/skills-wayfinder) 產出的是散布在地圖上的決策，而不是成品。`to-spec` 把它們收斂成一份你可以據以建置的文件。如果你把地圖直接接到 `/implement`，你就失去了這一步。

**spec 是給我審查的，還是只給 agent 看的？**
主要是給 agent 看的，而它讀起來也是如此：完整、密集、充滿引用。讀接縫與範圍外這兩節。在這兩個地方，錯誤的決策現在抓到最便宜，之後才發現最昂貴。確實有人抱怨要讀完整份，而且沒有摘要模式。但如果 spec 讓你感到意外，那是拷問太淺了；不是 spec 太長。

**ticket 開始之後，我要把 spec 凍結，還是讓 agent 改寫它？**
沒有任何東西會讓它保持同步。實務上它是你在那個時刻所知道之事的一張快照，而實作第一次教會你某件事時，它就過時了。在工作交付之後，把它當作可拋棄的。你的 `GLOSSARY.md` 與 ADR 才是要長久保存的檔案。如果你在實作期間學到了應該長久保存的東西，就放到那裡，而不是放進一份被編輯過的 spec。

**我的工作是一次重構或一個模組邊界，而不是一個功能。這個範本適用嗎？**
比較不適用，這是已知的限制。範本大量依賴使用者故事，而使用者故事並不適合架構性的工作。你最後會圍繞著其實是關於介面與不變式的決策，寫出沒人要的故事。改用「實作決策」與「測試決策」這兩節。把長期的架構決策透過 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 記錄成 ADR，而不是寫在 spec 中。

**它會檢查 tracker 中的相關工作，或引用它所遵循的 ADR 嗎？**
兩者都不會。它會閱讀並遵循它所觸及領域的 ADR，但不會連結它們。它也不會在撰寫前搜尋 tracker 中重疊的 issue，所以一份 spec 可能會重複某人已經提出的工作，而且沒有任何東西會警告你。如果該領域很熱鬧，請先自己搜尋 tracker。

**`/to-tickets` 讀不到我的 spec：它一直被截斷。**
tracker 的 issue 可能無法完整回傳一份非常大的 spec，而且沒有本地副本可以替代。要解決這個問題，不要在 `/to-spec` 與 `/to-tickets` 之間[清空（clear）](https://www.aihero.dev/ai-coding-dictionary/clearing)或[壓縮（compact）](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一個視窗中執行它們，`/to-tickets` 就永遠不需要再去取得 spec。

## 它有效的跡象

- 它開始寫，而不是問你新一輪的問題。
- 它在寫之前先給你看接縫，並盡可能提議最少的接縫。
- 它使用你專案的名詞，而不是通用的產品管理樣板文字。
- 你記得其中的每一個決策都是你做的。它沒有為了填滿某一節而發明任何東西。
- 範圍外一節列出的是真實的東西。你所拒絕的事，通常是這一頁上最有用的幾行。

## 它的定位

`to-spec` 是主要建置鏈中的一步，但只在其中多 session 的分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

在上游，[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 做出這個 skill 只負責記錄的決策，而一張完成的 [wayfinder](https://aihero.dev/skills-wayfinder) 地圖會在這裡匯入這條鏈。在下游，[to-tickets](https://aihero.dev/skills-to-tickets) 把 spec 切成曳光彈式的 ticket，交給 [implement](https://aihero.dev/skills-implement) 建置。當你不確定哪個 skill 或流程適合時，[ask-matt](https://aihero.dev/skills-ask-matt) 會幫你指路。
