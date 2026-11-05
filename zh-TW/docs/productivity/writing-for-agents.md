## 它做什麼

`writing-for-agents` 是撰寫面向 agent 之文件的參考資料：一個 skill、一份 `AGENTS.md` 或 `CLAUDE.md`、一份 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)、一段執行期提示、一份 README，任何 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 會讀的文件。格式不同，寫法則相同。同樣的槓桿能讓每一種都可預測，所以 agent 每次執行時都會走相同的*流程*（不一定產出相同的結果）。

它預設的修正方式是刪除，而不是解釋。叫一個 agent 為另一個 agent 寫指示，它會把大部分的字花在解釋[模型](https://www.aihero.dev/ai-coding-dictionary/model)早就知道的事情上。每一行這樣的內容都是一個 **no-op（無作用指令）**：它花費[情境](https://www.aihero.dev/ai-coding-dictionary/context)，卻不改變任何行為。這份參考資料幫助你找出它們，所以它對你已經寫好的文件，和對一個空白檔案一樣有用。

它在 v1.1 之前叫做 `writing-great-skills`。新名稱更符合它一直以來的本質。它的內容幾乎沒有任何部分是 skill 專屬的。只屬於 skill 的機制（frontmatter、模型觸發與使用者觸發的選擇、路由 skill）放在一份連結過去的 `SKILL-MECHANICS.md` 中，只有在你眼前的文件是一個 skill 時才需要讀。

## 何時使用

輸入 `/writing-for-agents`，或者在你建立或編輯 skill、修改 `AGENTS.md` 或 `CLAUDE.md` 時，agent 會自己使用它。

對其他所有 agent 會讀的東西，手動使用它：你的文件、spec 與 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)、系統提示與 [AFK（離線）](https://www.aihero.dev/ai-coding-dictionary/afk)提示。判準只有一個問題：agent 會讀這個嗎？agent 如何拿到這份文件並不重要：可能是指標指名了它、人類貼上了它，或它就在 repo 裡。若要找出一個程式碼庫包含什麼，請使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。這份參考資料控制的是一份文件讀起來如何，而不是它知道什麼。

## 兩種負擔

核心概念是每一份文件與每一個指標都會花費的兩種預算：

- **情境負擔（Context load）**：永遠載入的資料對 agent 視窗造成的成本：`AGENTS.md` 中的一行、一個 skill 的 description，任何每一個[輪次（turn）](https://www.aihero.dev/ai-coding-dictionary/turn)都留在情境中的東西，不論它是否被觸發。
- **認知負擔（Cognitive load）**：你記住有哪些文件存在、以及何時該使用每一份的成本。你就是索引。不要試圖把這個成本最小化，因為它是人類主導權的代價。

一旦你用這兩種負擔來思考，大多數撰寫上的決策（要不要拆分、要內嵌還是揭露、要用指標還是推入）都會變成同一種取捨，只是發生在不同的地方。

## 槓桿

- **[情境指標（Context pointers）](https://www.aihero.dev/ai-coding-dictionary/context-pointer)**：存在於情境中的一個引用，它指名某份不在情境中的資料，並編碼了何時該去讀取它。skill 的 description 與 `AGENTS.md` 中指名某份文件的一行字是同樣的東西。決定 agent 多可靠地依循它的，是指標的*措辭*，而不是它指向的目標。
- **資訊層級（Information hierarchy）**：從檔案內的步驟，到檔案內的參考，再到放在指標後面的揭露式參考的階梯。**[漸進式揭露（Progressive disclosure）](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)**就是把資料沿著這個階梯往下移，讓頂層保持容易閱讀。
- **完成條件（Completion criteria）**：每個步驟的完成條件有多清楚、要求有多高，以及這種要求所帶來的**跑腿工作（legwork）**。它們是對抗**過早完成**的防線。
- **主導詞（Leading words）**：一個已經存在於模型預訓練中的精簡概念（*tight（緊密）*、*red（紅燈）*、*tracer bullet（曳光彈）*），agent 在執行文件時會用它來思考。它在兩個地方發揮作用。在本文中它引導執行，在指標中它觸發調用。
- **刪減（Pruning）**：單一事實來源、相關性，以及逐句套用的 no-op 測試，用來對抗**重複**、**沉積**與**蔓生**。

## 常見問題

**`/writing-great-skills` 去哪了？**
就是這個 skill，在 v1.1 改了名。早在改名之前，使用者就已經拿它來處理 `AGENTS.md`、文件、spec、ticket 與執行期提示了。結構、主導詞與刪減適用於任何 agent 會讀的文字。沒有別名。請用新名稱重新安裝。

**「為 agent 寫作」：所以是 agent 在寫？**
相反。你是作者；agent 是讀者。這正是它困難的地方。你是在為一個已經讀過所有東西的讀者寫作，所以解釋是浪費，而精確就是全部的工作。

**我不能直接叫 agent 幫我寫嗎？**
可以，但它會產出囉嗦的東西。放著不管的話，模型會解釋它早就知道的事，而且它不會自己套用 no-op 測試，也不會主動去找主導詞。用這份參考資料來審查草稿。它大部分的價值都來自那一輪審查。

**我叫 agent 精簡一份文件，結果它把功能也刪掉了。**
被叫去「精簡」的 agent 會針對長度做最佳化，因為長度是它們看得到的東西。no-op 測試檢查的是行為，而不是風格：刪掉那一行，然後問 agent 的行為有沒有改變。當一句話不合格時，刪掉整句，而不是刪減其中幾個字；對它有意見分歧時，透過實際執行文件來解決，而不是靠爭論。

**我怎麼知道什麼時候算完成？**
當它能運作，而且你再也找不到重複、沉積或 no-op 時。沒有自動化的評估。你透過手動執行文件來檢查，並用失敗模式的詞彙來診斷問題。當一份文件表現異常時，同一套詞彙能幫你修正它：先說出失敗模式的名稱，再修正那個問題。

**這應該放在 `CLAUDE.md` 還是其他地方？**
問問自己你想付哪一種負擔。`CLAUDE.md` 會無條件載入每一個 [session](https://www.aihero.dev/ai-coding-dictionary/session)。放在指標後面的資料，在被觸發之前只花費指標本身那一行。任何十個 session 中只有一個會用到的東西，都會在另外九個 session 中花費情境負擔。

**每次有新模型，我都需要重寫我的文件嗎？**
大多數時候不需要，而且過度貼合某一個模型本身就會造成問題。為新模型更新，通常只是再做一輪 no-op 檢查，而不是重寫。

**我的 skill 只在我當初建立它的那個任務上有用。**
常見的方法（先把工作做一次，再讓 agent 把它寫成 skill）會太貼合那一次執行，而且範例會變得太具體。把那次執行當作證據保留下來，然後刻意地通用化：去掉屬於那個 repo 和那些檔案的東西，為這一類任務而寫。

**英文不是我的母語。我會失去主導詞的優勢嗎？**
不會。找出能把最多行為塞進最少 [token](https://www.aihero.dev/ai-coding-dictionary/token) 的詞，是這份參考資料替你做的工作。

## 它有效的跡象

- 文件越改越好的同時也越來越短，而你會驚訝於剩下的東西有多少。
- 你能指著一個主導詞，看到它在不只一個地方改變了 agent 的行為。
- 沒有任何東西以任何形式被說了兩次。重複是一份文件從未被測試過最可靠的跡象。
- 只有某一個分支需要的參考資料，放在指標後面，而不是放在主檔案中。

## 它的定位

這是一份隨時都能使用的獨立參考資料。它適用於整套 skill，而不只是某一個。這裡的每一個 skill 都是用它寫成的，而且一旦 agent 必須去讀其他 skill 產出的文件（一份 `GLOSSARY.md` 及其 ADR、一份 spec、一張 ticket），它也涵蓋這些文件。它唯一的直接呼叫者是 [retro](https://aihero.dev/skills-retro)，後者在提議任何引導檔案或 skill 之前都會先載入它。當你不確定哪個 skill 或流程適合某個任務時，[ask-matt](https://aihero.dev/skills-ask-matt) 會在整套 skill 中為你指路。
