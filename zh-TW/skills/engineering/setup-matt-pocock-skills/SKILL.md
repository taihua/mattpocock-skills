---
name: setup-matt-pocock-skills
description: "為 engineering skill 設定這個 repo：設定它的 issue tracker、分類標籤詞彙與領域文件的配置。在第一次使用其他 engineering skill 之前執行一次。"
disable-model-invocation: true
---

# 設定 Matt Pocock 的 Skill

建立 engineering skill 所預設的、每個 repo 各自的設定：

- **Issue tracker**：issue 放在哪裡（預設是 GitHub；本地 markdown 也是開箱即用）
- **分類標籤**：五個標準分類角色所使用的字串
- **領域文件**：`GLOSSARY.md` 與 ADR 放在哪裡，以及讀取它們時要遵守的規則

這是一個由提示驅動的 skill，不是確定性的腳本。探索、呈現你發現的東西、與使用者確認，然後才寫入。

## 流程

### 1. 探索

查看目前的 repo，了解它的起始狀態。讀取所有存在的東西；不要假設：

- `git remote -v` 與 `.git/config`：這是一個 GitHub repo 嗎？是哪一個？
- repo 根目錄的 `AGENTS.md` 與 `CLAUDE.md`：有沒有其中一個存在？其中是否已經有 `## Agent skills` 段落？
- repo 根目錄的 `GLOSSARY.md` 與 `GLOSSARY-MAP.md`，以及舊版的 `CONTEXT.md` / `CONTEXT-MAP.md`。如果存在舊版檔案，提議用 `git mv` 把它改成 `GLOSSARY` 的名稱，並在 `docs/agents/domain.md` 中改寫這些名稱。
- `docs/adr/` 以及任何 `src/*/docs/adr/` 目錄
- `docs/agents/`：這個 skill 先前的產出是否已經存在？
- `.scratch/`：這是已經在使用本地 markdown issue tracker 慣例的跡象
- `triage` skill 是否已安裝？（在這個 skill 旁邊有一個 `triage` skill 資料夾，或你可用的 skill 中有 `triage`。）這決定了 B 節是否需要執行。
- Monorepo 的跡象：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 欄位，或有內容的 `packages/*` 且各自有自己的 `src/`。這些只會出現在真正大型的多套件 repo 中；沒有這些就代表單一 context，幾乎所有 repo 都是如此。

### 2. 呈現發現並詢問

摘要說明有哪些東西存在、缺少哪些。然後依序處理各節。一節、一個答案，然後下一節。

每一節都先給出建議的答案，讓使用者一個字就能接受。只有在選擇確實會分岔時才附上一行說明；當探索已經決定了某一節時就整節跳過（`triage` 沒安裝時跳過 B 節，沒有 monorepo 時跳過 C 節）。

**A 節：Issue tracker。**

> 說明：「issue tracker」是這個 repo 存放 issue 的地方。像 `to-tickets`、`triage` 和 `to-spec` 這些 skill 會從那裡讀取、也會寫入那裡。它們需要知道該呼叫 `gh issue create`、在 `.scratch/` 下寫一個 markdown 檔，還是遵循你描述的其他流程。選擇你實際追蹤這個 repo 工作的地方。

預設立場：這些 skill 是為 GitHub 設計的。如果 `git remote` 指向 GitHub，就提議用它。如果 `git remote` 指向 GitLab（`gitlab.com` 或自架的主機），就提議 GitLab。否則（或如果使用者偏好），提供以下選項：

- **GitHub**：issue 放在 repo 的 GitHub Issues（使用 `gh` CLI）
- **GitLab**：issue 放在 repo 的 GitLab Issues（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 markdown**：issue 以檔案形式放在這個 repo 的 `.scratch/<feature>/` 下（適合個人專案或沒有 remote 的 repo）
- **其他**（Jira、Linear 等）：請使用者用一段話描述流程；skill 會把它以自由文字記錄下來

把選擇記錄在 `docs/agents/issue-tracker.md`。GitHub 與 GitLab 的範本帶有一個「PR 作為請求管道」的旗標，預設為**關閉**。保持關閉，也不要提起它：想把外部 PR 放進分類佇列的使用者，之後可以自己在檔案中切換這個旗標。

GitHub 或 GitLab tracker 需要一個可連線的 repo。當使用者選擇其中之一，但沒有任何 `git remote` 指向該主機時，向他取得 repo 的 URL 並接上（需要的話先 `git init`，然後 `git remote add origin <url>`），再繼續往下。當 `gh repo view` / `glab repo view` 在這個目錄中能印出專案時就完成了；如果不行，就停下來並告訴使用者缺少什麼。

**B 節：分類標籤詞彙。** 如果 `triage` skill 沒有安裝（探索時已經得知），就整節跳過，因為沒安裝的 skill 不需要標籤。

如果已經安裝，就只問一個問題：

> 你要保留預設的分類標籤嗎？（建議：**是**）

預設值是五個標準角色，每個標籤字串都等於它的名稱：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。如果回答**是**，就照原樣寫入。只有在使用者說不時（通常是因為他的 tracker 已經使用其他名稱，例如用 `bug:triage` 代表 `needs-triage`），才蒐集覆寫值，讓 `triage` 套用既有的標籤，而不是建立重複的標籤。

**C 節：領域文件。** 預設為**單一 context**（repo 根目錄一份 `GLOSSARY.md` + `docs/adr/`）。這適合幾乎所有 repo；不用詢問就直接寫。

只有在探索時發現 monorepo 跡象時，才提供**多 context**（根目錄一份 `GLOSSARY-MAP.md`，指向每個 context 各自的 `GLOSSARY.md`）選項。然後確認他們要哪一種配置。

### 3. 確認並編輯

給使用者看以下內容的草稿：

- 要加到 `CLAUDE.md` / `AGENTS.md` 其中一個（選擇規則見步驟 4）的 `## Agent skills` 區塊
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 與 `docs/agents/triage-labels.md` 的內容（最後一個只有在 `triage` 已安裝時才有）

在寫入之前讓他們編輯。

### 4. 寫入

**選擇要編輯的檔案：**

- 如果 `CLAUDE.md` 存在，就編輯它。
- 否則如果 `AGENTS.md` 存在，就編輯它。
- 如果兩者都不存在，詢問使用者要建立哪一個；不要替他選。

當 `CLAUDE.md` 已經存在時，絕不建立 `AGENTS.md`（反之亦然）；永遠編輯已經存在的那一個。

如果選定的檔案中已經有 `## Agent skills` 區塊，就原地更新它的內容，而不是附加一個重複的區塊。不要覆寫使用者對周圍段落的編輯。

區塊內容：

```markdown
## Agent skills

### Issue tracker

[一行摘要說明 issue 在哪裡追蹤]. See `docs/agents/issue-tracker.md`.

### Triage labels

[一行摘要說明標籤詞彙]. See `docs/agents/triage-labels.md`.

### Domain docs

[一行摘要說明配置："single-context" 或 "multi-context"]. See `docs/agents/domain.md`.
```

只有在 `triage` 已安裝且 B 節有執行時，才包含 `### Triage labels` 子區塊，並寫入 `docs/agents/triage-labels.md`。如果沒有，兩者都省略。

當 B 節在 GitHub 或 GitLab 上執行時，為 tracker 缺少的每一個設定好的標籤建立標籤（`gh label create` / `glab label create`）。

接著以這個 skill 資料夾中的種子範本為起點，寫入文件檔案：

- [issue-tracker-github.md](./issue-tracker-github.md)：GitHub issue tracker
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md)：GitLab issue tracker
- [issue-tracker-local.md](./issue-tracker-local.md)：本地 markdown issue tracker
- [triage-labels.md](./triage-labels.md)：標籤對應（只有在 `triage` 已安裝時）
- [domain.md](./domain.md)：領域文件的讀取規則 + 配置

對於「其他」issue tracker，依使用者的描述從頭撰寫 `docs/agents/issue-tracker.md`。

### 5. 完成

告訴使用者設定已完成，以及哪些 engineering skill 現在會讀取這些檔案。提到他們之後可以直接編輯 `docs/agents/*.md`；只有在想切換 issue tracker 或從頭來過時，才需要重新執行這個 skill。
