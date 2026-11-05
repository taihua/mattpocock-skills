# Issue tracker：GitHub

這個 repo 的 issue 與 spec 以 GitHub issue 的形式存在。所有操作都使用 `gh` CLI。

## 慣例

- **建立 issue**：`gh issue create --title "..." --body "..."`。多行內文使用 heredoc。
- **讀取 issue**：`gh issue view <number> --json number,title,body,labels,comments`。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，搭配適當的 `--label` 與 `--state` 過濾條件。
- **把 issue 設為某個父 issue 的子 issue**：`gh issue create --parent <parent> ...`，或之後再用 `gh issue edit <parent> --add-sub-issue <child>`（`gh` 2.94 以上）。較舊的 `gh`：`gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（資料庫 id，如下方**阻擋**所述）。沒有子 issue 功能時，在子 issue 內文最上方放 `Part of #<parent>`。
- **阻擋**：GitHub 的**原生 issue 依賴關係**，這是標準且在 UI 上可見的表示方式。用 `gh issue create --blocked-by <n>,<n> ...` 或 `gh issue edit <child> --add-blocked-by <blocker>`（`gh` 2.94 以上）新增一條關係。較舊的 `gh`：`gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`，其中 `<blocker-db-id>` 是阻擋者的數字**資料庫 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，*不是* `#number` 或 `node_id`）。在沒有依賴關係功能的地方，退而在子 issue 內文最上方放一行 `Blocked by: #<n>, #<n>`。當每個阻擋者都關閉時，ticket 就解除阻擋。
- **計算開放中的阻擋者**：`gh api repos/<owner>/<repo>/issues/<n> --jq .issue_dependencies_summary.blocked_by`（只算開放中的阻擋者；0 代表可以開始）。`gh issue view <n> --json blockedBy` 也會列出已關閉的阻擋者，所以要過濾：`--jq '[.blockedBy.nodes[] | select(.state == "OPEN")] | length'`。
- **在 issue 上留言**：`gh issue comment <number> --body "..."`
- **套用 / 移除標籤**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **關閉**：先用 `gh issue comment <number> --body "..."` 發布說明，然後 `gh issue close <number>`。對已經關閉的 issue，`gh issue close --comment` 會丟掉留言，但仍然以 0 結束。

從 `git remote -v` 推斷 repo；在 clone 下來的目錄中執行時，`gh` 會自動這麼做。

只在使用者自己的 repo 中建立 issue。除非使用者擁有 `mattpocock/skills`（這些 skill 的來源 repo），否則絕不在那裡建立 issue。在 fork 中，`gh` 可能預設指向上游的父 repo，所以在一個 session 中第一次執行 `gh issue create` 之前，用 `gh repo view --json nameWithOwner` 檢查目標，如果不是使用者自己的 repo，就與使用者確認。

## 把 Pull request 作為分類管道

**PRs as a request surface: no.**（PR 作為請求管道：否）_（如果這個 repo 把外部 PR 視為功能請求，就設為 `yes`；`/triage` 會讀取這個旗標。）_

設為 `yes` 時，PR 會使用與 issue 相同的標籤與狀態，並使用對應的 `gh pr` 指令：

- **讀取 PR**：`gh pr view <number>`，接著 `gh pr view <number> --comments`（在非 TTY 環境下，`--comments` 只會印出留言，不包含內文），以及用 `gh pr diff <number>` 取得 diff。
- **列出要分類的外部 PR**：`gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **留言 / 標籤 / 關閉**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 issue 與 PR 共用同一組編號，所以單純的 `#42` 可能是其中任何一個：先用 `gh pr view 42` 解析，不行再退回 `gh issue view 42`。

## 當 skill 說「publish to the issue tracker（發布到 issue tracker）」

建立一個 GitHub issue。

## 當 skill 說「fetch the relevant ticket（取得相關的 ticket）」

依上方**讀取 issue** 的方式讀取。

## Wayfinding 操作

由 `/wayfinder` 使用。**地圖（map）**是單一個 issue，以**子** issue 作為 ticket。

- **地圖**：單一個標上 `wayfinder:map` 的 issue，內文包含 Notes / Decisions-so-far / Fog。`gh issue create --label wayfinder:map`。
- **子 ticket**：以 GitHub 子 issue 連結到地圖的 issue（參見**把 issue 設為某個父 issue 的子 issue**）。在沒有啟用子 issue 的地方，把子 issue 加到地圖內文的任務清單中，並在子 issue 內文最上方放 `Part of #<map>`。標籤：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被認領，ticket 就指派給主導的開發者。
- **阻擋**：參見「慣例」中的**阻擋**。
- **前緣查詢**：列出地圖中開放的子項（`gh issue list --state open`，限定在地圖的子 issue 或任務清單內），去掉任何有開放阻擋者（參見**計算開放中的阻擋者**，或 `Blocked by` 那一行中有開放的 issue）或已有指派者的項目；依地圖順序排第一個的勝出。
- **認領**：`gh issue edit <n> --add-assignee @me`，這是 session 的第一個寫入動作。
- **解決**：`gh issue comment <n> --body "<answer>"`，然後 `gh issue close <n>`，接著把一個情境指標（要點 + 連結）附加到地圖的 Decisions-so-far。
