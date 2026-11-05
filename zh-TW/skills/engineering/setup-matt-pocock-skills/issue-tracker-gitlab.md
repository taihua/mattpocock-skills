# Issue tracker：GitLab

這個 repo 的 issue 與 spec 以 GitLab issue 的形式存在。所有操作都使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

## 慣例

- **建立 issue**：`glab issue create --title "..." --description "..."`。多行描述使用 heredoc。傳入 `--description -` 會開啟編輯器。
- **讀取 issue**：`glab issue view <number> --comments`。使用 `-F json` 取得機器可讀的輸出。
- **列出 issue**：`glab issue list -O json`，搭配適當的 `--label` 過濾條件。
- **在 issue 上留言**：`glab issue note <number> --message "..."`。GitLab 把留言稱為「notes」。
- **套用 / 移除標籤**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多個標籤可以用逗號分隔，或重複使用旗標。
- **關閉**：`glab issue close <number>`。`glab issue close` 不接受關閉留言，所以要先用 `glab issue note <number> --message "..."` 發布說明，再關閉。
- **Merge request**：GitLab 把 PR 稱為「merge request」。使用 `glab mr create`、`glab mr view`、`glab mr note` 等等，形式與 `gh pr ...` 相同，只是用 `mr` 取代 `pr`，用 `note`/`--message` 取代 `comment`/`--body`。

從 `git remote -v` 推斷 repo；在 clone 下來的目錄中執行時，`glab` 會自動這麼做。

## 把 Merge request 作為分類管道

**MRs as a request surface: no.**（MR 作為請求管道：否）_（如果這個 repo 把外部 merge request 視為功能請求，就設為 `yes`；`/triage` 會讀取這個旗標。）_

設為 `yes` 時，MR 會使用與 issue 相同的標籤與狀態，並使用對應的 `glab mr` 指令：

- **讀取 MR**：`glab mr view <number> --comments`，以及用 `glab mr diff <number>` 取得 diff。
- **列出要分類的外部 MR**：`glab mr list -O json`，然後只保留作者不是專案成員或擁有者的 MR（貢獻者的 MR，而不是維護者進行中的工作）。
- **留言 / 標籤 / 關閉**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

與 GitHub 不同，GitLab 的 issue 與 MR 分開編號，所以只要知道維護者指的是哪一種，`#42` 就沒有歧義。

## 當 skill 說「publish to the issue tracker（發布到 issue tracker）」

建立一個 GitLab issue。

## 當 skill 說「fetch the relevant ticket（取得相關的 ticket）」

執行 `glab issue view <number> --comments`。

## Wayfinding 操作

由 `/wayfinder` 使用。**地圖（map）**是單一個 issue，以**子** issue 作為 ticket。

- **地圖**：單一個標上 `wayfinder:map` 的 issue，內文包含 Notes / Decisions-so-far / Fog。`glab issue create --label wayfinder:map`。（在有原生 epic 的 GitLab 方案中，可以改用 epic 來存放地圖；有標籤的 issue 則在任何地方都可用。）
- **子 ticket**：一個在描述最上方帶有 `Part of #<map>` 的 issue，連結到地圖（`glab issue create --linked-issues <map> --link-type relates_to`），並帶有 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）標籤。一旦被認領，ticket 就指派給主導的開發者。
- **阻擋**：GitLab 的**原生阻擋連結**，這是標準且在 UI 上可見的表示方式。以留言的形式發布 `/blocked_by #<n>` quick action 來新增（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生阻擋連結是 Premium/Ultimate 方案的功能；在免費方案（或無法使用的地方），退而在描述最上方放一行 `Blocked by: #<n>, #<n>`。當每個阻擋者都關閉時，ticket 就解除阻擋。
- **前緣查詢**：用 `glab api projects/:id/issues/<map>/links` 列出地圖的子項，保留帶有 `wayfinder:<type>` 標籤的開放 issue；去掉任何有開放阻擋者的項目：一條指向開放 issue 的原生 `blocked_by` 連結（`glab api projects/:id/issues/<child-iid>/links`）、`Blocked by` 那一行中有開放的 issue，或已有指派者；依地圖順序排第一個的勝出。
- **認領**：`glab issue update <n> --assignee @me`，這是 session 的第一個寫入動作。
- **解決**：`glab issue note <n> --message "<answer>"`，然後 `glab issue close <n>`，接著把一個情境指標（要點 + 連結）附加到地圖的 Decisions-so-far。
