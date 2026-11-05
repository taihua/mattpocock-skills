# 繁體中文翻譯與實作導讀

這個資料夾是 `docs/` 與 `skills/`（`engineering/`、`productivity/` 兩個主要分類）的**繁體中文對照翻譯**，目錄結構與原文一一對應：

```
zh-TW/
├── README.md                 ← 本文（實作導讀）
├── docs/                     ← 對應 /docs：給人看的說明頁
│   ├── agents/
│   ├── engineering/
│   └── productivity/
└── skills/                   ← 對應 /skills：給 agent 執行的 SKILL.md 與輔助檔
    ├── engineering/
    └── productivity/
```

> **注意：這裡的檔案是「閱讀用」的。** 真正被 Claude Code / Codex 載入執行的，仍然是原本 `skills/` 底下的英文檔。翻譯版的 `SKILL.md` 保留了相同的 frontmatter `name`，如果把 `zh-TW/skills/` 也安裝進 harness，會出現同名的重複 skill。想實際使用中文版，請只安裝其中一份。

翻譯原則：

- skill 名稱、指令（如 `/tdd`）、檔名、程式碼、frontmatter 的 key 一律保留英文。
- frontmatter 中的 `description`、`argument-hint` 有翻譯（它們是文字內容）。
- 關鍵術語第一次出現時會附上英文，例如「接縫（seam）」、「前緣（frontier）」，方便對照原文。
- `docs/` 頁面中指向 `aihero.dev` 的外部連結維持原樣（連到的是英文網站）。
- `skills/in-progress/`、`skills/misc/` 沒有翻譯：它們不在 plugin 中發布，也沒有 docs 頁。

---

## 一、Skill 是什麼？

一個 **skill** 就是一個資料夾，裡面最重要的是一份 `SKILL.md`。agent（Claude Code、Codex 等）在需要時把這份 Markdown 載入情境（context），然後照著裡面寫的做。**它不是程式，而是寫給 agent 看的指示文件。** 所以這個 repo 的「實作」，本質上就是「怎麼寫好一份給 agent 讀的文件」。

### 一個 skill 資料夾的樣子

以 `tdd` 為例：

```
skills/engineering/tdd/
├── SKILL.md              ← 主檔：frontmatter + 指示本文
├── tests.md              ← 輔助參考檔（好 / 壞測試範例）
├── mocking.md            ← 輔助參考檔（何時 mock）
└── agents/
    └── openai.yaml       ← 給 Codex 的 UI metadata
```

有些 skill 還會附帶腳本或範本，例如：

- `wizard/template.sh`：產生互動式 bash 精靈的固定函式庫
- `diagnosing-bugs/scripts/hitl-loop.template.sh`：「人在迴圈中」的除錯腳本範本

### `SKILL.md` 的結構

```markdown
---
name: tdd
description: Test-driven development. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests.
---

# Test-Driven Development

（以下是給 agent 的指示本文……）
```

frontmatter 中會出現的欄位：

| 欄位 | 作用 |
| --- | --- |
| `name` | skill 的名稱，也就是 `/name` 指令 |
| `description` | 一段描述。**對模型觸發的 skill 來說，它會一直放在 agent 的情境中**，agent 靠它判斷何時該自動使用這個 skill |
| `disable-model-invocation: true` | 設定後，只有人類輸入 `/name` 才能觸發（使用者觸發） |
| `argument-hint` | 輸入指令時給使用者的參數提示，例如 `handoff` 的「下一個 session 會用來做什麼？」 |
| `metadata` | 其他資訊，例如 `pr` 用它記錄致謝來源 |

### `agents/openai.yaml`

每個 skill 旁邊都有一份，給 Codex 用：

```yaml
interface:
  display_name: "TDD"
  short_description: "Test-driven red-green-refactor"
```

使用者觸發的 skill 還會多一段：

```yaml
policy:
  allow_implicit_invocation: false
```

這必須和 `SKILL.md` 的 `disable-model-invocation: true` 保持一致：一個 skill 要嘛在兩個 harness 中都是使用者觸發，要嘛都不是。

---

## 二、兩種觸發方式（最核心的設計）

整個 repo 把每個 skill 分成兩類，依據原文 `.agents/invocation.md`：

| | 使用者觸發（User-invoked） | 模型觸發（Model-invoked） |
| --- | --- | --- |
| 誰能觸發 | **只有人類**輸入 `/name` | 模型自己判斷，或人類輸入 |
| 設定 | `disable-model-invocation: true` + `policy.allow_implicit_invocation: false` | 兩者都不設（預設） |
| `description` 寫法 | 給人看的一行摘要，不寫觸發條件 | 給模型看，寫滿觸發語句（「Use when the user wants…」） |
| 情境成本 | 零：description 不會被放進 agent 情境 | 每一輪都佔用情境 |
| 例子 | `grill-me`、`to-spec`、`implement`、`triage`、`ask-matt` | `grilling`、`tdd`、`code-review`、`domain-modeling`、`prototype` |

判斷標準只有一句：**「模型有沒有可能需要自己主動去用它？」** 有的話就做成模型觸發，否則做成使用者觸發，不付任何情境成本。

一個重要的限制：**使用者觸發的 skill 無法被其他 skill 呼叫。** 所以當一個 skill 需要另一個使用者觸發的 skill 作為前置條件時，它只能寫「請告訴使用者執行 `/setup-matt-pocock-skills`」，而不能自己去呼叫。

### Skill 之間怎麼互相呼叫

規則是：**明確寫出「呼叫 Skill 工具」**，而不是用 `../other-skill/FILE.md` 的路徑去引用別的資料夾。例如：

- `grill-me/SKILL.md` 的全部內容就是：`Call the Skill tool with "grilling".`
- `grill-with-docs/SKILL.md` 是：`Call the Skill tool twice, for "grilling" and "domain-modeling".`

所以 `grill-me` 與 `grill-with-docs` 其實只是「入口」，真正的訪談邏輯放在模型觸發的 `grilling` 中，詞彙表與 ADR 的邏輯放在 `domain-modeling` 中。這就是「單一事實來源」：同一件事只寫在一個地方，其他 skill 透過呼叫來共用。

### 共用參考資料放哪裡

- 只有某個 skill 自己用的參考 → 放在那個 skill 的資料夾中（例如 `tdd/tests.md`）。
- 多個 skill 都要用的參考 → 做成一個模型觸發的 skill（例如 `codebase-design` 提供深模組詞彙，`tdd` 與 `improve-codebase-architecture` 都去呼叫它）。

---

## 三、寫作原則（為什麼 SKILL.md 長那樣）

這些原則整理在 `writing-for-agents` skill 中（中文版見 [skills/productivity/writing-for-agents/SKILL.md](./skills/productivity/writing-for-agents/SKILL.md)），重點如下：

1. **兩種負擔**：情境負擔（佔 agent 的 context window）與認知負擔（人要記得有哪些 skill）。每一個設計決策都是在兩者之間取捨。
2. **情境指標（context pointer）**：description、`AGENTS.md` 中的一行，都是「指向某份資料、並說明何時該去讀它」的指標。措辭決定 agent 多可靠地去讀。
3. **漸進式揭露（progressive disclosure）**：每個分支都需要的內容放在 `SKILL.md` 本文；只有部分情況需要的內容移到輔助檔（例如 `prototype/` 依問題分成 `LOGIC.md` 與 `UI.md` 兩個分支）。
4. **完成條件**：每一步都要有清楚、可檢查的「做完的標準」，防止 agent 草草結束。`diagnosing-bugs` 的「必須先有一個會亮紅燈的指令才能進入階段 2」就是典型例子。
5. **主導詞（leading words）**：用一個模型已經懂的詞（*tight*、*red*、*tracer bullet*、*fog of war*）取代一整句說明，又省 token 又更精準。
6. **刪減**：刪掉模型本來就會做的指示（no-op）、刪掉重複、刪掉過時內容。
7. **用正向描述而非禁止**：寫「要做什麼」，而不是「不要做什麼」，因為禁止反而會把那個行為帶進情境。

另外，凡是要派出子代理（subagent）的 skill，都會寫明「以全新的 context window 啟動」，因為有些 harness（例如 Codex）預設會 fork 父對話，那樣子代理就繼承了它原本要逃離的情境。

---

## 四、Repo 的組織方式

### 分類資料夾（bucket）

| 資料夾 | 用途 | 是否隨 plugin 發布 | 是否有 docs 頁 |
| --- | --- | --- | --- |
| `skills/engineering/` | 日常程式工作 | 是 | 是 |
| `skills/productivity/` | 日常非程式工作流程 | 是 | 是 |
| `skills/in-progress/` | beta，公開徵求回饋 | 否 | 否 |
| `skills/misc/` | 保留但很少用 | 否 | 否 |
| `skills/deprecated/` | 已不再使用（目前是空的） | 否 | 否 |

每個分類資料夾都有自己的 `README.md`，把 skill 分成「使用者觸發」與「模型觸發」兩組列出。

### 發布的設定檔

- `.claude-plugin/plugin.json`：`skills` 陣列列出**所有** `engineering/` 與 `productivity/` 的 skill，這就是 Claude Code plugin 實際發布的集合。
- `.claude-plugin/marketplace.json`：repo 自己的 marketplace，其他 agent 從這裡安裝。
- 修改這兩個檔案後要執行 `claude plugin validate . --strict`。

### `docs/` 與 `skills/` 的關係

- `skills/.../SKILL.md` 是**給 agent 讀的**：精簡、指令式、每個字都有成本。
- `docs/.../<skill>.md` 是**給人讀的**：解釋這個 skill 做什麼、何時使用、常見問題、怎麼判斷它有正常運作。發布在 `https://aihero.dev/skills-<skill-name>`。

每一份 docs 頁都有固定四個段落（依原文 `.agents/writing-docs.md`）：

1. **What it does（它做什麼）**
2. **When to reach for it（何時使用）**
3. **Common questions（常見問題）**
4. **It's working if（它有效的跡象）**

最後通常還有一段 **Where it fits（它的定位）**，說明它在整個流程中與哪些 skill 相鄰。

### 腳本

- `scripts/link-skills.sh`：把 `deprecated/` 與 `misc/` 以外的所有 skill，以 symlink 的方式連到 `~/.claude/skills` 與 `~/.agents/skills`，供維護者本地使用。
- `scripts/list-skills.sh`：列出 repo 中所有 `SKILL.md`。（注意：它也會列出 `zh-TW/` 下的翻譯版。）
- `scripts/sync-plugin-version.mjs`：讓 `plugin.json` 的版本與 `package.json` 同步。

---

## 五、整體流程地圖

`ask-matt` 是所有 skill 的路由器（中文版見 [skills/engineering/ask-matt/SKILL.md](./skills/engineering/ask-matt/SKILL.md)）。主流程是：

```txt
/setup-matt-pocock-skills（每個 repo 執行一次）
        ↓
/grill-with-docs   ← 拷問式訪談，磨利想法，寫入 GLOSSARY.md 與 ADR
        ↓
（需要的話繞道 /prototype）
        ↓
/to-spec           ← 把對話整理成 spec 發布到 issue tracker
        ↓
/to-tickets        ← 拆成曳光彈式的垂直切片 ticket，帶阻擋關係
        ↓
/implement（每張 ticket 一次）或 /implement-spec（一次平行跑完）
   └─ 內部驅動 /tdd，最後執行 /code-review
        ↓
/pr                ← 撰寫 PR 內文
        ↓
/retro             ← 回顧 session，改善 agent 的環境
```

匝道（會匯入主流程的起點）：

- **`/triage`**：處理別人送來的 bug 與請求。
- **`/diagnosing-bugs`**：某個東西壞了。
- **`/wayfinder`**：大到一個 session 裝不下的工作，先畫成決策地圖。

建議閱讀順序：先讀 [docs/engineering/ask-matt.md](./docs/engineering/ask-matt.md) 了解全貌，再依你關心的流程讀各個 skill 的 docs 頁，最後對照 `skills/` 下的 `SKILL.md` 看它實際給 agent 的指示是什麼。

---

## 檔案對照清單

### Engineering skills

| Skill | 指示（給 agent） | 說明（給人） |
| --- | --- | --- |
| ask-matt | [SKILL.md](./skills/engineering/ask-matt/SKILL.md)、[PHASE-BOUNDARIES.md](./skills/engineering/ask-matt/PHASE-BOUNDARIES.md) | [ask-matt.md](./docs/engineering/ask-matt.md) |
| code-review | [SKILL.md](./skills/engineering/code-review/SKILL.md) | [code-review.md](./docs/engineering/code-review.md) |
| codebase-design | [SKILL.md](./skills/engineering/codebase-design/SKILL.md)、[DEEPENING.md](./skills/engineering/codebase-design/DEEPENING.md)、[DESIGN-IT-TWICE.md](./skills/engineering/codebase-design/DESIGN-IT-TWICE.md) | [codebase-design.md](./docs/engineering/codebase-design.md) |
| diagnosing-bugs | [SKILL.md](./skills/engineering/diagnosing-bugs/SKILL.md) | [diagnosing-bugs.md](./docs/engineering/diagnosing-bugs.md) |
| domain-modeling | [SKILL.md](./skills/engineering/domain-modeling/SKILL.md)、[ADR-FORMAT.md](./skills/engineering/domain-modeling/ADR-FORMAT.md)、[GLOSSARY-FORMAT.md](./skills/engineering/domain-modeling/GLOSSARY-FORMAT.md) | [domain-modeling.md](./docs/engineering/domain-modeling.md) |
| grill-with-docs | [SKILL.md](./skills/engineering/grill-with-docs/SKILL.md) | [grill-with-docs.md](./docs/engineering/grill-with-docs.md) |
| implement | [SKILL.md](./skills/engineering/implement/SKILL.md) | [implement.md](./docs/engineering/implement.md) |
| implement-spec | [SKILL.md](./skills/engineering/implement-spec/SKILL.md) | [implement-spec.md](./docs/engineering/implement-spec.md) |
| improve-codebase-architecture | [SKILL.md](./skills/engineering/improve-codebase-architecture/SKILL.md)、[HTML-REPORT.md](./skills/engineering/improve-codebase-architecture/HTML-REPORT.md) | [improve-codebase-architecture.md](./docs/engineering/improve-codebase-architecture.md) |
| pr | [SKILL.md](./skills/engineering/pr/SKILL.md)、[CREDITS.md](./skills/engineering/pr/CREDITS.md) | [pr.md](./docs/engineering/pr.md) |
| prototype | [SKILL.md](./skills/engineering/prototype/SKILL.md)、[LOGIC.md](./skills/engineering/prototype/LOGIC.md)、[UI.md](./skills/engineering/prototype/UI.md) | [prototype.md](./docs/engineering/prototype.md) |
| research | [SKILL.md](./skills/engineering/research/SKILL.md) | [research.md](./docs/engineering/research.md) |
| retro | [SKILL.md](./skills/engineering/retro/SKILL.md) | [retro.md](./docs/engineering/retro.md) |
| setup-matt-pocock-skills | [SKILL.md](./skills/engineering/setup-matt-pocock-skills/SKILL.md) 與 5 份範本 | [setup-matt-pocock-skills.md](./docs/engineering/setup-matt-pocock-skills.md) |
| tdd | [SKILL.md](./skills/engineering/tdd/SKILL.md)、[tests.md](./skills/engineering/tdd/tests.md)、[mocking.md](./skills/engineering/tdd/mocking.md) | [tdd.md](./docs/engineering/tdd.md) |
| to-spec | [SKILL.md](./skills/engineering/to-spec/SKILL.md) | [to-spec.md](./docs/engineering/to-spec.md) |
| to-tickets | [SKILL.md](./skills/engineering/to-tickets/SKILL.md) | [to-tickets.md](./docs/engineering/to-tickets.md) |
| triage | [SKILL.md](./skills/engineering/triage/SKILL.md)、[AGENT-BRIEF.md](./skills/engineering/triage/AGENT-BRIEF.md)、[OUT-OF-SCOPE.md](./skills/engineering/triage/OUT-OF-SCOPE.md) | [triage.md](./docs/engineering/triage.md) |
| wayfinder | [SKILL.md](./skills/engineering/wayfinder/SKILL.md) | [wayfinder.md](./docs/engineering/wayfinder.md) |
| wizard | [SKILL.md](./skills/engineering/wizard/SKILL.md) | [wizard.md](./docs/engineering/wizard.md) |
| （已封存）resolving-merge-conflicts | 已從 plugin 移除 | [resolving-merge-conflicts.md](./docs/engineering/resolving-merge-conflicts.md) |

### Productivity skills

| Skill | 指示（給 agent） | 說明（給人） |
| --- | --- | --- |
| grill-me | [SKILL.md](./skills/productivity/grill-me/SKILL.md) | [grill-me.md](./docs/productivity/grill-me.md) |
| grilling | [SKILL.md](./skills/productivity/grilling/SKILL.md) | [grilling.md](./docs/productivity/grilling.md) |
| handoff | [SKILL.md](./skills/productivity/handoff/SKILL.md) | [handoff.md](./docs/productivity/handoff.md) |
| teach | [SKILL.md](./skills/productivity/teach/SKILL.md) 與 4 份格式檔 | [teach.md](./docs/productivity/teach.md) |
| to-questionnaire | [SKILL.md](./skills/productivity/to-questionnaire/SKILL.md) | [to-questionnaire.md](./docs/productivity/to-questionnaire.md) |
| wait-what | [SKILL.md](./skills/productivity/wait-what/SKILL.md) | [wait-what.md](./docs/productivity/wait-what.md) |
| writing-for-agents | [SKILL.md](./skills/productivity/writing-for-agents/SKILL.md)、[SKILL-MECHANICS.md](./skills/productivity/writing-for-agents/SKILL-MECHANICS.md) | [writing-for-agents.md](./docs/productivity/writing-for-agents.md) |
