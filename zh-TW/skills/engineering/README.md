# Engineering（工程）

我每天做程式工作時使用的 skill。

## 使用者觸發（User-invoked）

只有在你親自輸入名稱時才能觸發（Claude Code：`disable-model-invocation: true`；Codex：在 `agents/openai.yaml` 中設定 `policy.allow_implicit_invocation: false`）。

- **[ask-matt](./ask-matt/SKILL.md)**：詢問哪個 skill 或流程適合你的情況。是本 repo 中所有使用者觸發 skill 的路由器。
- **[grill-with-docs](./grill-with-docs/SKILL.md)**：一場拷問式訪談，同時建立你專案的領域模型（domain model），磨利術語，並即時更新 `GLOSSARY.md` 與 ADR。
- **[triage](./triage/SKILL.md)**：讓 issue 在由各種分類角色組成的狀態機中流轉。
- **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)**：掃描程式碼庫，找出可以「加深」的機會，以視覺化 HTML 報告呈現，然後針對你挑選的那一個進行拷問式討論。
- **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)**：為 engineering skill 設定這個 repo（issue tracker、分類標籤、領域文件的配置）。每個 repo 執行一次。
- **[to-spec](./to-spec/SKILL.md)**：把目前的對話轉成一份 spec（規格），並發布到 issue tracker。
- **[to-tickets](./to-tickets/SKILL.md)**：把任何計畫、spec 或對話拆成一組曳光彈（tracer-bullet）式的 ticket，每張都宣告它的阻擋關係，可以是本地檔案中的文字，也可以是真正 tracker 上的原生阻擋連結。
- **[implement](./implement/SKILL.md)**：實作 spec 或一組 ticket 所描述的工作，在事先約定的接縫（seam）上驅動 `/tdd`，並在 commit 前以 `/code-review` 收尾。
- **[implement-spec](./implement-spec/SKILL.md)**：在單一整合分支上實作整份 spec。把 ticket 當作任務圖來處理，在已就緒的前緣上執行多個實作子代理以達到最大並行度，最後以 `/code-review` 收尾。
- **[wayfinder](./wayfinder/SKILL.md)**：規劃一大塊工作（超過一個 agent session 能容納的量），在 issue tracker 上建立一張由決策 ticket 組成的共用地圖，一次解決一張，直到通往目的地的路變得清楚。
- **[retro](./retro/SKILL.md)**：在一個 session 結束後，建議如何改善 coding agent 的環境（導覽、自動化檢查、程式碼規範、引導檔案、工具），依嚴重程度排序。

## 模型觸發（Model-invoked）

模型或使用者都可以觸發（description 中寫了豐富的觸發語句，讓模型能主動使用它們）。

- **[prototype](./prototype/SKILL.md)**：建立一個用完即丟的原型來回答一個設計問題：針對狀態或邏輯，做成單一可分享的 HTML 檔；或做出幾個可切換的 UI 變體。

- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)**：針對棘手 bug 與效能退化的紀律化診斷迴圈：建立一個會在這個 bug 上亮紅燈的回饋迴圈 → 最小化 → 提出假設 → 加入觀測 → 修復 → 回歸測試。
- **[research](./research/SKILL.md)**：針對一個問題，以高可信度的第一手資料進行調查，並把發現整理成一份附引用的 Markdown 檔存進 repo，以背景 agent 的方式執行。
- **[tdd](./tdd/SKILL.md)**：以紅燈、綠燈、重構（red-green-refactor）迴圈進行測試驅動開發。一次一個垂直切片地建立功能或修復 bug。
- **[domain-modeling](./domain-modeling/SKILL.md)**：主動建立並磨利專案的領域模型，方法是質疑術語、用情境做壓力測試，並即時更新 `GLOSSARY.md` 與 ADR。
- **[codebase-design](./codebase-design/SKILL.md)**：設計深模組（deep module）的共用紀律與詞彙：小介面、乾淨的接縫、可透過介面測試。
- **[code-review](./code-review/SKILL.md)**：從某個固定點以來的 diff 的雙軸審查：**Standards（規範）**（是否遵循 repo 的程式碼規範，加上 Fowler 的程式碼壞味道基準？）與 **Spec（規格）**（是否忠實實作了原始的 issue 或 spec？），以平行的子代理執行。
- **[pr](./pr/SKILL.md)**：pull request 內文應有的樣子：用能讓變更一目了然的最小視覺化作為摘要、證明它可運作的前後對比證據，以及合併風險判斷（單向門或雙向門，加上影響範圍）。
- **[wizard](./wizard/SKILL.md)**：產生一個互動式 bash 精靈，引導人類完成只有他能做的步驟：佈建基礎設施、設定憑證或 CI secret、操作陌生的第三方後台，或執行一次性的遷移或切換。
