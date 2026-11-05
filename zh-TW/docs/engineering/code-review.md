## 它做什麼

`code-review` 沿著兩個軸，審查 `HEAD` 與你指定的某個固定點（一個 commit、一個分支、一個 tag、`main`、`HEAD~5`）之間的 diff。**Standards（規範）**問的是程式碼是否遵循這個 repo 寫程式碼的方式。**Spec（規格）**問的是程式碼是否做到了原始 issue 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 所要求的。每個軸都在自己的[子代理（sub-agent）](https://www.aihero.dev/ai-coding-dictionary/subagent)中執行，所以兩者都看不到對方的推理。

這個 skill 從不合併或重新排序兩個軸。報告最後會列出*每個軸*最嚴重的問題，並拒絕在兩軸之間選出單一個贏家。一個變更可能通過其中一軸、卻沒通過另一軸。遵循每一條慣例、卻實作了錯的東西的程式碼，會通過 Standards、沒通過 Spec。完全做到 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 要求、卻破壞了 repo 慣例的程式碼則相反。一個混合的判決會讓通過的那一軸掩蓋失敗的那一軸。

## 何時使用

輸入 `/code-review`，或者當你要求審查一個分支、一個 PR、進行中的工作，或任何「自 X 以來」的東西時，agent 會自動使用它。

| 你的情況 | 使用 |
| --- | --- |
| 有一份 diff，你想知道它是否建得對、*而且*是否是對的東西 | `code-review` |
| 你想在 diff 中找 bug：null 路徑、競態條件、差一錯誤 | Claude Code 自己內建的審查，而不是這個（參見下方的名稱衝突） |
| 什麼都還沒寫，你想以測試先行的方式寫 | [tdd](https://aihero.dev/skills-tdd) |
| 需要建置一整份 spec，包含審查 | [implement](https://aihero.dev/skills-implement)，它會自己呼叫這個 skill |
| 漂移的是整個程式碼庫，而不是一份 diff | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| 某個東西壞了，而你不知道為什麼 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

你必須提供固定點。如果你沒有，skill 會向你要一個，而不是用猜的。在派出任何東西之前，它會檢查 ref 可以被解析、diff 不是空的，所以一個打錯的分支名稱會在你面前失敗，而不是在兩個子代理裡面才失敗。

## 前置條件

Standards 軸什麼都不需要。它讀取 repo 所記錄的任何東西（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 之類的），而當 repo 什麼都沒記錄時，就退回一個內建的基準。

Spec 軸需要一份存在且找得到的 spec。它依以下順序尋找：

1. commit 訊息中的 issue 引用（`#123`、`Closes #45`、GitLab 的 `!67`），透過 tracker 文件取得。
2. 你以參數傳入的路徑。
3. 在 `docs/`、`specs/` 或 `.scratch/` 下，與分支或功能名稱相符的 spec 檔案。
4. 問你。

步驟 1 依賴 tracker 文件，而那份文件是由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 寫的。沒有它，只要你給一個路徑，這個軸仍然可以運作。如果完全沒有 spec，skill 會跳過 Spec 子代理，而報告會說「no spec available（沒有可用的 spec）」，而不是編造需求。

## 兩個軸

| | Standards | Spec |
| --- | --- | --- |
| 問題 | 它建得對嗎？ | 它是對的東西嗎？ |
| 讀取 | repo 文件化的規範，加上壞味道基準 | 原始的 issue 或 spec |
| 回報 | 違反文件化規範之處（可以是硬性的），以及壞味道（永遠是判斷題） | 缺漏或只做了一部分的需求、範圍蔓延、實作錯誤的需求 |
| 每一項發現都引用 | 規範檔案與規則，或指名的壞味道加上 hunk | spec 中的那一行 |

這個設計是為了避免一個不了解你規範的通用審查 skill。那樣的 skill 會標記你程式碼庫中刻意為之的東西，並漏掉你程式碼庫所依賴的不變式。所以 repo 自己的文件是 Standards 軸上的[第一手資料](https://www.aihero.dev/ai-coding-dictionary/primary-source)，而且**永遠以 repo 為準**。

**壞味道基準（smell baseline）**位於 repo 的規範之下。它是 Fowler《_Refactoring_（重構）》第 3 章中的十二種程式碼壞味道：Mysterious Name（神秘命名）、Duplicated Code（重複程式碼）、Feature Envy（依戀情結）、Data Clumps（資料泥團）、Primitive Obsession（基本型別偏執）、Repeated Switches（重複的 switch）、Shotgun Surgery（散彈式修改）、Divergent Change（發散式變化）、Speculative Generality（夸夸其談的通用性）、Message Chains（過長的訊息鏈）、Middle Man（中間人）、Refused Bequest（被拒絕的遺贈）。每一個都是一個帶標籤的經驗法則（「possible Feature Envy（可能有依戀情結）」），從不是硬性違規。每一個都說明了這個壞味道是什麼以及如何修正，所以一項發現會附帶一個修正方式，而不只是一個抱怨。兩個軸都會略過任何你的 linter 已經強制執行的東西。

## 常見問題

**它與 Claude Code 自己的 `/code-review` 衝突。我該怎麼辦？**

這是這個 skill 被回報最多的問題，而且還沒修正。Claude Code 附帶了它自己的 `/code-review`，做的是不同的事：它在 diff 中找 bug，而這個檢查的是 spec 符合度與 repo 規範。當你安裝這個函式庫時，其中一個會勝出，而哪一個勝出取決於你的安裝方式：

- **Plugin marketplace。** 每個 skill 都會有一個 `mattpocock-skills:` 前綴，而內建的那個在不加限定的名稱下會變得很難觸及。
- **單純的 skill 安裝。** 本地檔案勝出，這個 skill 會遮蔽內建的那個。

一個解法是完全移除 Claude Code 的內建 skill。這能省下大量的[情境](https://www.aihero.dev/ai-coding-dictionary/context)，而衝突也就不再重要。遮蔽本身可以說是 Claude Code [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 的 bug（skill 作者應該可以自由地為 skill 取任何名字），所以另一個解法是重新命名本地副本。`npx skills update` 會撤銷對 frontmatter 的編輯或重新命名的目錄。使用者回報的持久變通方法，是把 skill fork 成一個新名稱，並把 `code-review` 從受管理的集合中拿掉。記下你 fork 時的 commit，這樣你就能手動重新同步。

**它的子代理一直再次觸發 `/code-review`，並派出更多 agent。**

這是一個已知的開放 bug。好幾個人在不只一個 harness 中重現了它。Standards 與 Spec 的提示並沒有禁止委派，所以一個子代理可能會再次找到這個 skill 並再次展開。一份回報達到了超過 50 個 agent。人們在 fork 上套用的修正，是在兩個子代理的簡報後面都加上一行：「Do not invoke `/code-review` or spawn additional agents: perform this review directly.（不要觸發 `/code-review` 或派出額外的 agent：直接執行這次審查。）」有些人偏好在 harness 層級處理它，讓每個 skill 都繼承這道防護。兩者都還沒有放進發布的 skill 中。如果你在無人看管的情況下執行它，請留意 agent 的數量。

**我應該在寫程式碼的同一個 [session](https://www.aihero.dev/ai-coding-dictionary/session) 中執行它嗎？**

偏好一個新的 session。正如一位讀者所說：「同一個情境審查它自己不是審查，而是附帶 slash command 的確認偏誤。」一個在撰寫 session 中進行審查的 agent，情境中有形塑程式碼的每一個假設。一個獨立的審查者不會有那個情境。這也是為什麼人們會要求一個沒有內建審查步驟的 [implement](https://aihero.dev/skills-implement)，因為那個步驟是在剛寫完 diff 的 session 中執行審查。獨立的版本，就是你自己從一個乾淨的 session 觸發 `/code-review`。

**每張 ticket 之後執行，還是在最後執行一次？**

兩者都可以，而這個 skill 不替你決定。每張 ticket 審查一次，能讓每份 diff 小到讓 Spec 軸有一份清楚的 spec 可以比對，這也是 `implement` 使用的模式。在分支結束時一次批次審查，則能抓到每張 ticket 單獨審查時都會漏掉的 ticket 之間的交互作用。如果你不確定，就每張 ticket 審查一次，最後再針對分岔點跑一次最終審查。

**我能相信這些發現嗎？**

不經檢查的話不能。子代理的輸出是一個假設，而不是證據。一個團隊回報了十幾個基於文字的審查都漏掉的破壞性變更。這個 skill 會把兩份報告原樣或稍加整理後合在一起。它不會針對檔案重新驗證每一項主張，所以一項發現可能引用了錯的位置，或誇大了影響。在據以行動之前，閱讀每一項發現上的引用。skill 要求每一項發現都附帶引用（一條規範規則、一個壞味道加上它的 hunk，或 spec 中的一行），而這正是讓發現可以被檢查的原因。

**為什麼我每次執行它，它都會找到新問題？**

每次修正都會加入新的程式碼要審查，而 Standards 軸中判斷題的那一半每次執行的結果都不同。一位讀者描述了這個迴圈：「/code-review 和 /improve-code-architecture 每次都會找到新東西。我實作修正、重新執行這些 skill，然後一次又一次。」沒有收斂的保證。把一次審查當作一份線索清單。處理那些背後有引用規則的，然後就停止。不要在迴圈中執行它直到它回報乾淨，因為它永遠不會。

**它會審查我還沒 commit 的工作嗎？**

不會。它比對的是 `<fixed-point>...HEAD`。三個點的形式是從 merge-base 開始量，並排除已暫存與工作樹中的變更。如果 `implement` 還沒做一個中間的 commit，審查就看不到即將進入下一個 commit 的工作。先 commit，再審查，然後 amend 或加一個 fixup。

## 它有效的跡象

- 在派出任何子代理之前，它會拒絕在錯誤的 ref 或空的 diff 上開始。
- 報告以 `## Standards` 與 `## Spec` 兩個獨立的區塊送達，而不是一份合併的清單。
- 每一項 Standards 發現都指名了你 repo 某個檔案中的一條規則，或十二種壞味道之一，並引用了 hunk；每一項 Spec 發現都引用了 spec 中的一行。
- 最後的總結列出每個軸最嚴重的問題，並拒絕選出整體的贏家。
- 在沒有可用 spec 的情況下，Spec 區塊會直說，而不是列出它從程式碼推斷出來的需求。

## 它的定位

`code-review` 是建置鏈接近尾端的審查步驟：`grill-with-docs → to-spec → to-tickets → implement → code-review → retro`。它也可以單獨用在任何你指給它的分支或 PR 上。

- [implement](https://aihero.dev/skills-implement) 是最近的鄰居。它驅動建置，並在 commit 之前呼叫這個 skill 作為它自己的收尾審查。[implement-spec](https://aihero.dev/skills-implement-spec) 也這樣做，只是對整個整合分支做一次。
- [retro](https://aihero.dev/skills-retro) 在它之後，並調整它。當一個 session 顯示審查漏掉了某一類錯誤時，`retro` 會提議一個檢查或一條 `CODING_STANDARDS.md` 規則，讓 Standards 軸之後去讀。
- [pr](https://aihero.dev/skills-pr) 在審查過的工作提交時撰寫 pull request 內文。
- [to-spec](https://aihero.dev/skills-to-spec) 與 [to-tickets](https://aihero.dev/skills-to-tickets) 產出 Spec 軸用來比對的文件，所以一份模糊的 spec 會讓那個軸也變得模糊。
- [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是針對整個程式碼庫的對應 skill，因為這個 skill 只看一份 diff。

當你不確定當下情況需要哪個 skill 時，[ask-matt](https://aihero.dev/skills-ask-matt) 會在整套 skill 中為你指路。
