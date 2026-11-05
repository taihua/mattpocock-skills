## 它做什麼

`pr` 定義了 pull request 內文的形狀：一段呈現變更的 **Summary（摘要）**、證明它可運作的 **Evidence（證據）**，以及一個關於合併它有多危險的 **Merge Danger（合併風險）**判斷。它是一份格式參考，不是一套工作流程。它不會推送分支、開 PR，也不會決定 PR 裡要放什麼。它告訴 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 在撰寫內文時，內文應該長什麼樣子。

摘要是一個視覺化，而不是一段文字。預設的 PR 內文會用散文描述 diff。這個則挑選能把重點講清楚的**最小視圖**（虛擬碼、呼叫樹、元件樹、檔案樹、Mermaid 圖，或有形狀的 diff），並讓周圍的文字保持簡短。審查者已經打開了 diff，所以內文要在他們閱讀之前，先讓他們看到它的形狀。

## 何時使用

輸入 `/pr`，或者只要 agent 在撰寫 PR 內文，它就會自動使用它。

| 你的情況 | 使用 |
| --- | --- |
| 分支已經準備好，需要一份審查者能快速瀏覽的內文 | `pr` |
| 程式碼已經寫好，但還沒有人審查過 | 先用 [code-review](https://aihero.dev/skills-code-review)，再用 `pr` |
| PR 已經開了，審查意見陸續回來 | 這套 skill 中還沒有對應的；`pr` 只負責寫內文 |

## 範本

三個段落，依此順序：

- **Summary**：一個或多個小型視覺化，每一個都放在它所支撐的那段簡短文字旁邊。用一個，有時用好幾個，很少全部都用。只保留審查者需要的呼叫、檔案、props 與邊界。
- **Evidence**：之前與之後。當變更是視覺上的、而且[環境](https://www.aihero.dev/ai-coding-dictionary/environment)可以截圖時，截圖是最強的證據。否則，使用那個從失敗變成通過的確切測試（以虛擬碼寫出），或是有所改變的主控台輸出。
- **Merge Danger**：這個變更是**單向門**還是**雙向門**，以及它的**影響範圍（blast radius）**。雙向門很容易逆轉；單向門（破壞性的遷移、移除公開 API、難以逆轉的決策）則不然。影響範圍指出如果這個變更是錯的，什麼可能會壞掉：版面位移、API 的使用方、行動裝置的響應式表現。

「門」的判斷是主導概念。它把「這可以安全合併嗎？」從一種直覺，變成一個審查者可以反對的明確主張。它也告訴他們要把[人工審查](https://www.aihero.dev/ai-coding-dictionary/human-review)的心力花在哪裡。影響範圍小的雙向門快速瀏覽即可，單向門則要慢慢讀。

## 常見問題

**我能相信 agent 自己對「門」的判斷嗎？**

不能盲目相信，而這正是要把它寫出來的原因。寫這個變更的 agent 就是給它打分數的那一個，所以當它說「雙向、影響範圍小」時要最懷疑。diff 也常常隱藏了一個變更是否能被逆轉。正如一位使用者所說：「回滾一個 commit 並不會把一批已寄出的 email 收回來」，而一個用旗標控制的上線，只有在第一筆資料以新格式寫入之前才算是雙向的。這個 skill 給了 agent 一個定義（破壞性操作與難以逆轉的決策是單向的），而不是一份檢查清單，所以要最仔細檢查 Merge Danger 那一行。有兩件事有幫助：確保 agent 眼前有 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，而不只是 diff；並把你的 repo 一律視為單向的變更（schema 遷移、任何會對外發送或刪除的東西）寫在 agent 做判斷前會讀到的地方。

**它會幫我開 PR 嗎？**

不會。`pr` 只涵蓋內文。[implement](https://aihero.dev/skills-implement) 最後是 commit 到目前的分支。要求一個會開 PR 的 skill 或選項（一個 `/to-pr`，或讓 `implement` 開 PR 而不是 commit）的提案仍然開放中。一位使用者的變通方法，是對 `implement` 做一個一句話的本地覆寫，叫它開一個 PR。[implement-spec](https://aihero.dev/skills-implement-spec) 是例外：當你的 issue tracker 透過 PR 結案工作，或你要求開 PR 時，它會開一個 draft PR。因為 `pr` 是模型觸發的，所以只要你叫 agent 開 PR，它就會用這個形狀撰寫內文。

**它不會又產出一大堆文字和圖表嗎？**

這個 skill 的設計就是要防止這件事，但它仍然可能發生。使用者對 agent 寫的 PR 內文的抱怨很一致：「一大段摘要，但我只需要知道改了什麼、怎麼驗證的、什麼可能會壞，以及能不能安全合併。」這個 skill 告訴 agent 要省略開場白、文字保持簡短，並挑選最小的視圖，通常是一個視覺化，很少全部都用。如果你還是拿到一疊圖表，那是 agent 忽略了這一點。如果內文很龐大是因為 diff 很龐大，問題在於 PR 的大小，而 `pr` 不會替你拆分它。

**我的 repo 已經有 PR 範本了。哪一個優先？**

預設上兩者都不優先。`pr` 有它自己的範本，不會去找 `.github/pull_request_template.md` 或任何類似的東西。好幾位使用者要求這個 skill 使用 repo 的範本。如果你什麼都不做，agent 對同一份文件就會有兩個相互競爭的指示。在你 repo 的 agent 文件中解決它，例如填寫 repo 的範本，並把 Summary、Evidence 與 Merge Danger 放在它底下。

**輸出是 HTML 嗎？為什麼 Mermaid 圖沒有渲染出來？**

輸出是 markdown 格式的 PR 內文，不是 HTML 頁面。GitHub 與 GitLab 會在 PR 描述中渲染 Mermaid 區塊，但終端機不會，所以在 agent 還在本地給你看的時候，圖表看起來像原始文字。使用 CLI harness 的使用者用 ASCII Mermaid 渲染器，或讓 agent 在 PR 上附一個 HTML 版本來變通。Mermaid 只是六種視圖之一。呼叫樹、檔案樹或有形狀的 diff 在任何地方讀起來都一樣。

**它能整理回來的審查意見嗎？**

不能。它寫完內文就停止了。使用者不只一次要求一個方法來分類來自其他開發者或審查機器人的意見（哪些值得處理、哪些不是問題）。這個 skill 不做這件事。

**它會隨著 PR 的變動，讓內文保持最新嗎？**

不會。它在某一個時間點寫下內文，而一個在審查中變動的 PR 會讓那份內文過時。在一次大的變動之後，請 agent 重寫內文。agent 又在撰寫 PR 內文了，所以它會使用同樣的形狀。

**我的變更沒有 UI。Evidence 要放什麼？**

除了截圖之外的一切。只有在變更是視覺上的時候，截圖才是最強的證據。對於遷移、背景工作或重構，證據是那個之前失敗、現在通過的確切測試，或是有所改變的主控台輸出。單單一句「測試都是綠的」只是一個主張，而不是之前與之後。

**它能把內文標記為由 LLM 撰寫的嗎？**

它自己不會。一位使用者的做法，是在 repo 的 agent 文件中加入一條常設指示：agent 撰寫的每一個 issue、留言與 PR，都要以一行揭露聲明結尾。那條規則屬於 repo，它能涵蓋 agent 發布的一切，而不是放在某一種文件的範本中。

## 它有效的跡象

- 光看 Summary 的視覺化，在打開 diff 之前你就能知道這個 PR 改了什麼。
- 內文沒有開場白：它從 Summary 標題開始。
- Evidence 一節呈現了之前與之後，而不是聲稱測試都通過了。
- 每一個 PR 都說明了一扇門與一個影響範圍，而那些單向門正是你會放慢速度的地方。

## 它的定位

當建置以 pull request 的形式交付時，`pr` 位於審查與回顧之間：`to-spec → to-tickets → implement → code-review → pr → retro`。它是模型觸發的，所以在那條鏈之外，只要 agent 撰寫 PR 內文，它也會自己觸發。

- [code-review](https://aihero.dev/skills-code-review) 在它之前執行，因為 PR 內文應該描述一份已經審查過的 diff。
- [implement](https://aihero.dev/skills-implement) 產出內文所描述的 commit。

當你不確定當下情況需要哪個 skill 時，[ask-matt](https://aihero.dev/skills-ask-matt) 會在整套 skill 中為你指路。
