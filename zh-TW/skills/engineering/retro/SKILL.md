---
name: retro
description: "對一次 coding session 進行回顧。"
disable-model-invocation: true
---

使用者要求進行一次**回顧（retrospective）**。你要針對 coding agent 的**環境**提出改善建議，讓未來的執行更好。

## 步驟

1. 以 `writing-for-agents` 呼叫 Skill 工具，取得寫作風格指南。

2. 閱讀使用者指定之 session 的第一手資料。這可能代表要搜尋這台機器上的 session 紀錄。如果使用者沒有指定 session，預設為目前這一個。

3. 在以下類別中尋找可改善的候選項目。

- **導覽（Navigation）**：agent 找到正確檔案有多容易？檔案之間有沒有隱藏的依賴？一個**導覽指標（navigation pointer）**會不會讓事情更容易？_適用時機_：這個 session 花了很長時間才找到某項資訊。
- **自動化檢查（Automated checks）**：有沒有自動化檢查可以抓到 agent 犯的錯？Lint、型別、測試、檔案系統 linter？先閱讀 repo 自己的檢查指令（它的 `package.json` 或建置工具中的 `lint`/`check` scripts、它的 CI workflow），這樣如果某個檢查早已存在但沒接上或默默壞掉，那才是發現，而不是重新發明一次。一個沒有任何**護欄（guardrail）**的 repo（沒有 pre-commit hook，也沒有跑 lint/typecheck/test 指令的 CI job）本身就是一個發現：沒有 lint 的 repo 是一個持續錯失的機會，而不是中性的預設。_適用時機_：agent 犯了一個自動化檢查本可抓到的錯，或 repo 完全沒有護欄。
- **程式碼規範（Coding standards）**：是否應該給**審查 agent（reviewer agent）**一條新規則去執行？是否應該移除或釐清某條既有規則？先分類違規的性質：**機械性**的違規（固定的語法模式、被禁止的 API、import 的形式、檔案位置規則）就該有一個確定性的檢查，沒有例外：在 repo 自己的 linter 中加一條自訂規則、一個新的 pre-commit hook，或一個新的 CI job，看 repo 的語言與既有護欄哪一種最便宜。預設是建立檢查，而不是寫規則。把 `CODING_STANDARDS.md` 保留給真正的**判斷題**（跨檔案的一致性、「符合周圍的風格」，以及任何護欄永遠無法取代的東西）。_適用時機_：審查 agent 沒有抓到某個錯誤。
- **全域 AGENTS.md**：有沒有任何引導指示應該移到程式碼規範（或自動化檢查）中？_適用時機_：AGENTS.md 檔案特別龐大，不論是在 repo 中還是使用者的全域範圍。
- **工具經濟性（Tool economy）**：agent 是否做了可以精簡的昂貴工具呼叫？有沒有特別浪費 token 的自訂工具（CLI、MCP）？_適用時機_：agent 做了一次昂貴的工具呼叫。
- **無作用指令（No-ops）**：在引導檔案中尋找不會改變 agent 行為的指示。_適用時機_：引導檔案龐大且難以駕馭。
- **資訊取得（Information access）**：尋找增加 agent 取得資訊能力的機會。把 dev server 的 log 同時輸出一份（tee）、對第三方服務的唯讀存取。_適用時機_：某個關鍵資訊 agent 拿不到。

4. 依嚴重程度排序，向使用者呈現這些候選項目。

## 參考

### 實作 vs 審查

記住，所有工作都會經過兩個階段：實作與審查。實作 agent 承受最大的**情境壓力（context pressure）**。它要負責探索、寫程式碼與除錯。

審查 agent 承受的情境壓力最小：它收到的是一份 diff，所以不需要探索。它通常也不需要寫程式碼或除錯。

這表示應該由審查 agent，而不是實作 agent，來負責執行程式碼規範。

### 檔案

你可以存取 repo 中的幾個檔案：

- `CLAUDE.md`/`AGENTS.md`：這些檔案會被推入任何在這個 repo 中工作的 agent 的 context window。應該極度節制地使用，通常只用來放指向其他檔案的**導覽指標**。
- `CODING_STANDARDS.md`：這個檔案在審查時讀取，而不是在實作時。如果規範檔超過 1,000 行，就加入指向 docs 資料夾的**導覽指標**。
- Docs：把文件當作參考檔案，由其他檔案指向它們。寫新文件之前先找找既有的文件。
- Skills：用 skill 來放文件（因為它們的 description 會進入 agent 的 context window），或用來做使用者觸發的指令。遵循 `writing-for-agents` skill 中的建議。
