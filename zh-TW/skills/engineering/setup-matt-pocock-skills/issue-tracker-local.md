# Issue tracker：本地 Markdown

這個 repo 的 issue 與 spec 以 markdown 檔案的形式存放在 `.scratch/` 中。

## 慣例

- 每個功能一個目錄：`.scratch/<feature-slug>/`
- spec 是 `.scratch/<feature-slug>/spec.md`
- 實作的 issue 每張 ticket 一個檔案，放在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，從 `01` 開始編號，絕不合併成單一個 ticket 檔案
- 分類狀態記錄在每個 issue 檔案靠近頂端的一行 `Status:` 中（角色字串請見 `triage-labels.md`）
- 留言與對話紀錄附加在檔案底部的 `## Comments` 標題下

## 當 skill 說「publish to the issue tracker（發布到 issue tracker）」

在 `.scratch/<feature-slug>/` 下建立一個新檔案（需要的話建立目錄）。

## 當 skill 說「fetch the relevant ticket（取得相關的 ticket）」

讀取所引用路徑上的檔案。使用者通常會直接傳入路徑或 issue 編號。

## Wayfinding 操作

由 `/wayfinder` 使用。**地圖（map）**是一個檔案，每張 ticket 各有一個**子**檔案。

- **地圖**：`.scratch/<effort>/map.md`（包含 Notes / Decisions-so-far / Fog 的內文）。
- **子 ticket**：`.scratch/<effort>/issues/NN-<slug>.md`，從 `01` 開始編號，問題寫在內文中。一行 `Type:` 記錄 ticket 類型（`research`/`prototype`/`grilling`/`task`）；一行 `Status:` 記錄 `claimed`/`resolved`。
- **阻擋**：靠近頂端的一行 `Blocked by: NN, NN`。當它列出的每個檔案都是 `resolved` 時，ticket 就解除阻擋。
- **前緣**：掃描 `.scratch/<effort>/issues/`，找出開放、未被阻擋、未被認領的檔案；編號最小的勝出。
- **認領**：在任何工作開始前，設定 `Status: claimed` 並存檔。
- **解決**：在 `## Answer` 標題下附加答案，設定 `Status: resolved`，然後把一個情境指標（要點 + 連結）附加到 `map.md` 中地圖的 Decisions-so-far。
