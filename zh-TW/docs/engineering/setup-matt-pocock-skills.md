## 它做什麼

`setup-matt-pocock-skills` 回答關於一個 repo 的三個問題：issue 放在哪裡、分類標籤叫什麼名字，以及領域文件放在哪裡。它把答案記錄成 `docs/agents/` 下的 markdown 檔案。

這些檔案是不同 repo 之間唯一會變動的東西。skill 本身在每個地方都一模一樣。它們在執行時讀取 `docs/agents/issue-tracker.md`，並照著它說的做。這就是為什麼這套 skill 沒有綁定 GitHub，也是為什麼你永遠不需要編輯 skill 檔案來讓它指向另一個 tracker。以「把這些 skill 連到一個自訂的 issue tracker」來觸發它，任何你能以程式方式連接的東西都能用，而且完全不需要修改 skill。

它是一個由提示驅動的 skill，不是確定性的腳本。它讀取你的 `git remote`、`CLAUDE.md` 與 `GLOSSARY.md`，提出它的發現，並在寫入任何東西之前等你確認。

## 何時使用

你透過輸入 `/setup-matt-pocock-skills` 來觸發它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不會自己去用它。它的 metadata 刻意把它標記為不可被觸發，所以沒有任何其他 skill 能替你觸發它。

每個 repo 使用一次，在第一次使用任何其他 engineering skill 之前。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 開始猜測你的 issue 要放在哪裡，或套用你的 tracker 沒有的標籤，代表這個 repo 還沒設定好。你可以在專案進行到一半的 repo 中執行它。這個 skill 會讀取已經存在的東西，所以先前的工作都不會遺失。

## 前置條件

它會寫入你執行它的那個 repo：

| 它寫入 | 位置 |
| --- | --- |
| `issue-tracker.md` | `docs/agents/` |
| `domain.md` | `docs/agents/` |
| `triage-labels.md` | `docs/agents/`，只有在 `triage` skill 已安裝時 |
| 一個 `## Agent skills` 區塊 | `CLAUDE.md` 或 `AGENTS.md` 中已經存在的那一個 |

你把這一切都以 markdown 的形式 commit。沒有使用者層級或全域的模式。設定存在 repo 中，所以每個 repo 都有自己的一份。

## 三個決策

它在每一節都先給出建議的答案，並跳過任何它在探索時已經回答過的問題。大多數執行只需要兩次確認。

| 決策 | 它提議什麼 | 何時會問 |
| --- | --- | --- |
| **Issue tracker** | 與你的 `git remote` 相符的那一個 | 一定會問，因為這是唯一真正的選擇 |
| **分類標籤** | 保留五個標準名稱（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`） | 只有在 `triage` skill 已安裝時 |
| **領域文件** | 單一 context：根目錄一份 `GLOSSARY.md` 加上 `docs/adr/` | 只有在發現 monorepo 跡象時，此時它會提供多 context 的 `GLOSSARY-MAP.md` 選項 |

Tracker 的選項：

| 選項 | issue 放在哪裡 | 需要 |
| --- | --- | --- |
| **GitHub** | repo 的 GitHub Issues | `gh` CLI |
| **GitLab** | repo 的 GitLab Issues | `glab` CLI |
| **本地 markdown** | 這個 repo 中 `.scratch/<feature>/` 下的檔案 | 什麼都不需要，連 remote 都不用 |
| **其他** | 你說在哪就在哪 | 你用一段話描述流程 |

前三種以範本的形式附在 skill 中，開箱即用。本地 markdown 是一個完整的選項，而不是退而求其次的方案。這個 skill 支援沒有 remote 的個人專案。一個注意事項：如果你使用 GitHub，就不要用本地 markdown。它們是二選一的，挑一個。

「其他」也是一個完整的選項。Jira、Linear、Azure DevOps 與 Beads 都是這樣運作的。你描述流程，skill 把你的文字記錄在 `docs/agents/issue-tracker.md` 中，下游的 skill 就照著這段文字做。使用者已經做出了這樣的東西：一個透過 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 使用 Jira 的變體、一個形式像 `gh` 的 Gitea CLI、一個手工打造的本地儀表板。

## 常見問題

**我一定要用 GitHub 嗎？**

不用。GitHub、GitLab 以及 `.scratch/` 下的本地 markdown 都以現成的範本附上，其他任何東西都可以透過「其他」路徑運作。這是被問最多次的問題，大致是這些說法：*「被 GitHub 鎖死了」*、*「我能用 GitLab / Jira 嗎」*、*「那 Azure DevOps 呢」*。答案永遠一樣：選擇 tracker 的是設定，而不是 skill。

**更新 skill 之後，我需要重新執行它嗎？**

在 v1.1 之後的直接答案是要。這個 skill 自己的結尾訊息則比較溫和。它告訴你只有在切換 tracker 或從頭來過時才需要重新執行。兩種說法都說得通。種子範本在不同版本之間會改變，所以一份來自舊版的 `docs/agents/issue-tracker.md`，可能會與現在讀取它的 skill 不一致。如果某個下游 skill 做的事與文件描述的不同，就重新執行設定。這很便宜。

**它寫到了 `CLAUDE.md`，但我用的是 Codex。**

這是一個已知的缺口，而且仍未解決。檔案選擇的規則是「如果 `CLAUDE.md` 存在就編輯它，否則編輯 `AGENTS.md`」。它檢查的是哪個檔案存在，而不是正在執行的是哪個 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)。在一個留有 Claude Code 時期 `CLAUDE.md` 的 repo 中，這個 skill 會把 `## Agent skills` 區塊寫進一個 Codex 永遠不會讀的檔案。使用者有兩種變通方法：手動把區塊移到 `AGENTS.md`，或讓 `AGENTS.md` 作為標準，並把 `CLAUDE.md` 改成一行指向它的指標。如果兩個檔案都不存在，skill 會問你要建立哪一個，而不是替你選。這讓預期它會自己決定的人感到困惑。

**它沒有建立我的分類標籤。**

它本來就不會。`docs/agents/triage-labels.md` 是一份*對應表*：它告訴 `/triage` 你的 tracker 中哪些字串對應到五個標準角色。它不會執行 `gh label create`。在一個全新的 GitHub repo 上，這些標籤還不存在，而使用者已經不只一次把這件事回報成 bug。兩個後果：

- 如果你的 tracker 已經使用標準名稱，這份對應表就是一張恆等表，沒有任何東西需要設定。這是預期的常見情況，而不是漏掉的步驟。
- 這個 skill 也不會建立 [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 與 `wayfinder:<type>` 標籤，而 `gh issue create --label <不存在的標籤>` 會直接失敗，而不是建立那個標籤。在 GitHub repo 上第一次執行 wayfinder 之前，請手動建立它們。

**我能在這裡設定其他 skill 的行為嗎（[拷問](https://www.aihero.dev/ai-coding-dictionary/grilling)的節奏、問題格式、語氣）？**

不行。它設定三件事：tracker、標籤、文件配置。使用者曾要求讓它成為存放個人偏好的地方。答案是 skill 保持自己的主見，不接受個人設定。偏好應該以一般指示的形式放在你的 `CLAUDE.md` 中，每一個 skill 本來就會讀它。

**我能把設定放在 `~/.claude`，而不是 commit 到每一個 repo 嗎？**

目前不行。一位在許多 repo 中使用這些 skill 的使用者對此提出了一個開放中的請求，但不存在使用者層級的模式。每個 repo 都帶著自己的 `docs/agents/`。

**用一個 skill 來設定其他 skill，不是很奇怪嗎？**

一個長期存在的抱怨認為是的，原話是：*「用一個 skill 來設定其他 skill 讓我覺得不太對：這代表 LLM 在設定它自己的 skill。」* 這個取捨是真實的。如果沒有設定步驟，每個會碰到 issue 的 skill 都需要自己的一份 tracker 指示。產出是你可以閱讀與編輯的 markdown，這限制了風險。你可以閱讀它寫的每一個檔案，並手動修改。日常的變更就用這種方式做，而不是再執行一次。

## 它有效的跡象

- `docs/agents/issue-tracker.md` 與 `docs/agents/domain.md` 存在，如果 `triage` 已安裝，還有 `triage-labels.md`。
- 在你的 harness 會讀的指示檔中出現一個 `## Agent skills` 段落，每個檔案都有一行指向它的摘要。
- 它提議的 tracker 與你使用的 remote 相符，而標籤字串與你 tracker 中存在的標籤相符。
- 之後，`/to-tickets` 發布時不會再問你 issue 放在哪裡，而 `/triage` 會套用標籤，而不是自己發明標籤。
- skill 檔案本身沒有任何改變。如果設定編輯了某個 `SKILL.md`，就是出錯了。

## 它的定位

`setup-matt-pocock-skills` 是工程流程的**一次性設定**，是其他一切都預設已完成的前置條件，而不是鏈中的一步。它的鄰居就是它的讀者：[triage](https://aihero.dev/skills-triage) 套用這裡寫下的標籤詞彙；[to-spec](https://aihero.dev/skills-to-spec) 與 [to-tickets](https://aihero.dev/skills-to-tickets) 發布到這裡指名的 tracker；而 [wayfinder](https://aihero.dev/skills-wayfinder) 會讀取同一份 tracker 檔案中的「Wayfinding operations」一節，了解如何存放地圖與子 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)。[domain-modeling](https://aihero.dev/skills-domain-modeling) 之後會填入設定所記錄的領域文件配置。它只有在你確定一個術語或決策時才會建立 `GLOSSARY.md` 與 ADR，所以設定之後 repo 沒有任何領域文件是正常的。至於下一步該用哪個 skill，[ask-matt](https://aihero.dev/skills-ask-matt) 會在整套 skill 中為你指路。
