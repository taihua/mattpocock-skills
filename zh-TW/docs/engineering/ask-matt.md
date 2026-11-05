## 它做什麼

`ask-matt` 是這個 repo 中所有 skill 的路由器。你描述你所處的情況（一個不知如何開始的想法、一堆湧入的 bug 回報、一個進行了很久的 [session](https://www.aihero.dev/ai-coding-dictionary/session)），它就會指出適合的 skill 或 skill 序列，以及在那個序列中需要人類做決定的地方。

它給出建議，然後就停止。它不會拷問、不會寫 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)、不會打開檔案，也不會觸發它剛才指名的 skill；你拿回來的是下一個要輸入的東西，而由你去輸入。它也是一張手寫的、這個 repo 中 skill 的地圖，而不是對你已安裝之物的掃描，所以它不會為你的自己的 skill 或其他作者的 skill 指路。

## 何時使用

你透過輸入 `/ask-matt` 來觸發它；agent 不會自己去用它。

| 你的情況 | 路由器給你什麼 |
| --- | --- |
| 一個想法，卻不知道從哪裡開始 | 主流程的開頭，以及這次建置是否小到可以跳過 spec |
| 來自別人的 bug 與請求 | [triage](https://aihero.dev/skills-triage) 匝道，以及為什麼你自己產生的 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 不屬於那裡 |
| 兩個看起來可以互換的 skill | 它們之間的界線，而那通常是一個具體的判準，而不是品味問題。[grill-me](https://aihero.dev/skills-grill-me) 或 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 取決於你是否在一個工作目錄中；[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 取決於這項工作是否放得進一個 session |
| 一個很長的 session，以及一個關於[情境](https://www.aihero.dev/ai-coding-dictionary/context)的決策 | 階段邊界上五個選項的排序決策樹 |
| 一個你已經挑好的 skill | 沒什麼用。直接觸發那個 skill。 |

## 前置條件

路由器負責指名 skill；它不會安裝它們。它指向的每一個東西都必須已經安裝，建議才能付諸行動，而它只知道這個 repo 中被推廣的 skill。

依賴 tracker 的路線（triage、`to-spec`、`to-tickets`、`implement`）預設 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已經在 repo 中設定好 issue tracker。即使那件事還沒發生，路由器也會推薦它們。

## 是流程，不是 skill

這個 skill 的主導詞是**流程（flow）**，一條*穿過*各個 skill 的路徑，而不是單一個 skill。當你說出你的情況時，路由器會把你放在某個流程的某一步上。這與「這是符合你關鍵字的 skill」是不同的答案。路線有五種，而 skill 本身完整描述了它們：

- **主流程**，從想法到出貨。拷問、spec、ticket、實作（一次一張 ticket，或用 [implement-spec](https://aihero.dev/skills-implement-spec) 平行處理整張任務圖）、審查，然後是 [retro](https://aihero.dev/skills-retro)，它把建置學到的東西回饋到 agent 的環境中。它有兩個分支。一個是原型繞道，用於某個問題需要可執行的程式碼才能解決時。另一個是 spec 與 ticket 的拆分，只有在建置橫跨不只一個 session 時才值得它的成本。
- **匝道**，用於一個會產生工作、然後匯入主流程的情況：湧入的 bug 回報、某個東西壞了，或一項太迷霧重重、太大而無法在一個 session 中掌握的工作。
- **程式碼庫健康**，是維護而不是功能開發。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 勘查程式碼以尋找加深的機會，而它找到的每一個都會以想法的形式重新進入主流程。
- **獨立的 skill**，不在任何流程上，你單獨使用它們：原型、問卷、研究。
- **底層的詞彙層**，當問題出在用詞而不是流程時，其他 skill 會拉進來的兩份參考資料。

## 階段邊界

這個 skill 的另一個關鍵概念是**階段邊界（phase boundary）**。階段是 session 中的一段工作（[拷問](https://www.aihero.dev/ai-coding-dictionary/grilling)、實作、QA），而兩個階段之間的邊界，是「我該怎麼處理這個情境？」這個問題唯一該出現的地方。在階段中途沒有什麼好決定的：繼續，或把剩下的工作拆給[子代理](https://www.aihero.dev/ai-coding-dictionary/subagent)。

| 選項 | 何時採用 |
| --- | --- |
| **繼續（Continue）** | 下一個階段需要這個階段的逐字內容，或你還有[聰明區（smart zone）](https://www.aihero.dev/ai-coding-dictionary/smart-zone)的空間。它是唯一能讓 session 保持為[第一手資料](https://www.aihero.dev/ai-coding-dictionary/primary-source)的選項，所以要最先排除它 |
| **`/clear`** | 你身後的一切都可以丟掉。最便宜的選項，而如果你判斷錯了就無法撤銷 |
| **[handoff](https://aihero.dev/skills-handoff)** | 有東西必須移動：一個新的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)、一個新的目錄、一位同事、一個在階段中途分岔出去的支線任務 |
| **子代理（Subagent）** | 任務的範圍緊縮到可以在你[離開鍵盤](https://www.aihero.dev/ai-coding-dictionary/afk)時執行 |
| **`/compact`** | 以上皆非。它是預設值，而決策樹常常落在這裡 |

人們常常搞錯其中兩個，這就是為什麼路由器給的是順序，而不只是清單。`/handoff` 看起來像是在 context window 之間移動的通用方法，但它不是。它給你的只有可攜性。`/compact` 是決策樹中的最後一個選項，而不是第一個，因為它上面的四個選項每一個都更便宜或更精確。

## 常見問題

**難道沒有一份依正確順序排列的 skill 清單嗎？**

人們一直要求在 README 中放一份。這個 skill 就是那份清單。一張靜態的表會寫著 `wayfinder → to-spec → to-tickets → implement → code-review → retro`，而對大多數情況來說都是錯的。重要的部分是分支：有沒有程式碼庫、建置是否橫跨多個 session、這個問題能不能用談話解決。代價是路由器是手動維護的，所以它會落後於 repo。`/grilling` 發布了很久之後，路由器才指名它。

**它告訴我一半的 skill 都沒安裝。**

這是一個已知的 bug，而且還沒修正。路由器為你指路經過的大多數 skill 都設定了 `disable-model-invocation: true`，這代表 harness 會把它們排除在注入 agent 情境的 skill 清單之外。agent 把那份清單讀成完整的，於是回報它們不存在。在一個被回報的 session 中，它宣稱整個 spec 與 ticket 流程都不存在，並改為指向單純的 `/grilling` 與 `/tdd`。plugin 的二十七個 skill 中有十六個帶有這個旗標，所以這是常見的情況，而不是邊緣情況。它們都已經安裝了。無論如何都輸入那個 slash command，或檢查 `.claude-plugin/plugin.json`，那才是判斷有哪些東西存在的權威。

**它描述了某個 skill 的行為，但那個 skill 並沒有那樣做。**

這也是一個真正的 bug，也還沒修正。路由器是根據它自己對每個 skill 的一行摘要來回答，而不是根據 skill 本身。一份詳細的回報在單一 session 中追蹤到了三個實例，包括一個只根據「把討論串轉成一份 spec」這行摘要，就建議跳過 [to-spec](https://aihero.dev/skills-to-spec) 的情況。路由器從沒打開過 `to-spec/SKILL.md`。在每一個情況中，它都是在使用者反駁之後才去驗證，從不主動這麼做。在那裡跳過 `to-spec` 讓一次真正的接縫檢查被省略，而產出的 ticket 低估了工作量。當路由器對另一個 skill 做出一個你會據以行動的陳述時，請它先打開那個 `SKILL.md`。同樣的道理也適用於地圖完全沒涵蓋的問題，例如是否要使用[規劃模式（plan mode）](https://www.aihero.dev/ai-coding-dictionary/agent-mode)：那個答案是[模型](https://www.aihero.dev/ai-coding-dictionary/model)的推論，而不是寫在這裡的東西。

**為什麼它是文字，而不是一份編號的檢查清單？**

這是一個合理的抱怨。一個開放中的 issue 主張大部分的路由都是確定性的，而敘述式的寫法讓它難以快速瀏覽。沒有任何東西阻止你要求精簡版：「直接給我順序就好」就能得到順序。文字承載的是條件性的部分：分支、預期需要人類決定的地方，以及在步驟之間要在哪裡清空或壓縮。一份扁平的檢查清單恰恰會丟掉這些。

**它能為我自己的 skill 或其他作者的 skill 指路嗎？**

不能。有三個不同的提案要求一個會讀取你本地 `skills/` 目錄、並根據已安裝之物做推薦的路由器。`ask-matt` 不是那個東西。它是一套 skill 的地圖，手動維護，對你自己寫的或從其他地方安裝的 skill 一無所知。

**它叫我編輯一個 SKILL.md。**

這個建議常常是對的，但編輯很少能持續。有人問它如何讓 [implement](https://aihero.dev/skills-implement) 關閉 ticket，被告知在 skill 中加一行，然後立刻就看出了問題。`npx skills update` 會覆寫那個檔案，而 plugin 的安裝是唯讀的。把常設的行為放在你自己的 `CLAUDE.md` 或 `AGENTS.md` 中，或在觸發時說出來。你在提示中做的改變能撐過更新。人們就是用這種方式把流程指向 Linear 而不是 GitHub，或問它哪些開放的 ticket 可以平行執行。

**它指名了一個我沒有的 skill，或漏掉了一個我有的。**

在假設它不見了之前，先查看 changelog 有沒有改名。`writing-great-skills` 變成了 [writing-for-agents](https://aihero.dev/skills-writing-for-agents)，沒有別名；`to-prd` 變成了 [to-spec](https://aihero.dev/skills-to-spec)；`pathfinder` 變成了 [wayfinder](https://aihero.dev/skills-wayfinder)。有四個 skill 直接退役，併入吸收它們的 skill：`ubiquitous-language`、`design-an-interface`、`qa` 與 `request-refactor-plan`。如果它漏掉了一個你有的 skill，那就是上面描述過的、路由器本身的落後。

## 它有效的跡象

- 它最後指出要輸入什麼，然後就停在那裡，而不是自己開始工作。
- 它給回來的路線會提到在哪裡清空或壓縮情境、在哪裡預期你要審查，而不只是一串 skill 名稱。
- 當兩個 skill 很接近時，它會說是哪一個，以及為什麼另一個不適合你。
- 它對另一個 skill 的行為所做的任何陳述，都會在追蹤紀錄中顯示為它去讀了那個 skill 的 `SKILL.md`。
- 你在它給回來的東西中認出了你自己的情況，而不是最接近的通用情境。

## 它的定位

`ask-matt` 是一個位於整套 skill 之上的**獨立路由器**。它從來不是鏈中的一步。它指向每一條鏈，而其他文件頁面都會連結回它，所以它們都不需要重畫這張圖。從這裡你最常落在 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，也就是主流程的開頭，或 [triage](https://aihero.dev/skills-triage)，也就是給「別人送來的工作」而不是「你自己開始的工作」的匝道。

它是它所描述之 skill 的[二手資料](https://www.aihero.dev/ai-coding-dictionary/secondary-source)。當路由器與某個 `SKILL.md` 意見不同時，以 `SKILL.md` 為準。
