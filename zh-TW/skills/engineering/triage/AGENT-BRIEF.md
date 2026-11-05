# 撰寫 Agent 簡報（Agent Brief）

agent 簡報是一則結構化的留言，在 GitHub issue 或 PR 進入 `ready-for-agent` 狀態時發布。它是離線（AFK）agent 工作時所依據的權威規格。原始內文與討論是背景：agent 簡報才是合約。

簡報說明 **agent 應該做什麼**，它同時涵蓋兩種對象：對 issue 而言，是從零建立變更；對 PR 而言，則是*對既有 diff* 還剩下哪些要做的事：把它完成、補上缺口、處理審查意見。兩者原則相同；下方的 PR 範例展示了其中的差異。

## 原則

### 耐久重於精確

issue 可能會在 `ready-for-agent` 狀態停留好幾天甚至好幾週。程式碼庫在這段期間會改變。撰寫簡報時要讓它即使在檔案被重新命名、移動或重構後仍然有用。

- **要**描述介面、型別與行為合約
- **要**指名 agent 應該尋找或修改的具體型別、函式簽章或設定結構
- **不要**引用檔案路徑：它們會過時
- **不要**引用行號
- **不要**假設目前的實作結構會維持不變

### 描述行為，而不是程序

描述系統**應該做什麼**，而不是**如何**實作它。agent 會重新探索程式碼庫，並自己做出實作決策。

- **好：**「`SkillConfig` 型別應接受一個型別為 `CronExpression` 的選填 `schedule` 欄位」
- **不好：**「打開 src/types/skill.ts，在第 42 行加入一個 schedule 欄位」
- **好：**「當使用者不帶參數執行 `/triage` 時，應該看到需要關注之 issue 的摘要」
- **不好：**「在主處理函式中加入一個 switch 敘述」

### 完整的驗收條件

agent 需要知道什麼時候算完成。每份 agent 簡報都必須有具體、可測試的驗收條件。每一條都應該能獨立驗證。

- **好：**「執行 `gh issue list --label needs-triage` 會回傳已經過初步分類的 issue」
- **不好：**「分類應該正確運作」

### 明確的範圍邊界

說明什麼不在範圍內。這能防止 agent 過度鍍金，或對相鄰的功能做出假設。

## 範本

```markdown
## Agent Brief

**Category（類別）:** bug / enhancement
**Summary（摘要）:** 一行描述需要發生什麼事

**Current behavior（目前行為）:**
描述現在發生的事。對 bug 而言，這是壞掉的行為。
對功能增強而言，這是新功能所要建立在其上的現狀。

**Desired behavior（期望行為）:**
描述 agent 的工作完成後應該發生的事。
要具體說明邊界情況與錯誤狀況。

**Key interfaces（關鍵介面）:**
- `TypeName`：需要改什麼，以及為什麼
- `functionName()` 的回傳型別：目前回傳什麼 vs 應該回傳什麼
- 設定結構：需要的任何新設定選項

**Acceptance criteria（驗收條件）:**
- [ ] 具體、可測試的條件 1
- [ ] 具體、可測試的條件 2
- [ ] 具體、可測試的條件 3

**Out of scope（範圍外）:**
- 在這個 issue 中「不」應該修改或處理的事
- 看似相關、但其實是分開的相鄰功能
```

## 範例

### 好的 agent 簡報（bug）

```markdown
## Agent Brief

**Category:** bug
**Summary:** Skill 描述截斷時從單字中間切斷，產生殘缺的輸出

**Current behavior:**
當 skill 描述超過 1024 個字元時，它會不管單字邊界、剛好在第
1024 個字元處被截斷。這會產生在單字中間結束的描述
（例如 "Use when the user wants to confi"）。

**Desired behavior:**
截斷應該在 1024 個字元之前的最後一個單字邊界處斷開，
並加上 "..." 表示已截斷。

**Key interfaces:**
- `SkillMetadata` 型別的 `description` 欄位：不需要改型別，
  但填入它的驗證或處理邏輯需要尊重單字邊界
- 任何讀取 SKILL.md frontmatter 並擷取描述的函式

**Acceptance criteria:**
- [ ] 1024 字元以下的描述維持不變
- [ ] 超過 1024 字元的描述在 1024 字元之前的
      最後一個單字邊界處截斷
- [ ] 截斷後的描述以 "..." 結尾
- [ ] 包含 "..." 在內的總長度不超過 1024 字元

**Out of scope:**
- 改變 1024 字元的上限本身
- 多行描述的支援
```

### 好的 agent 簡報（功能增強）

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** 加入 `.out-of-scope/` 目錄支援，用來追蹤被拒絕的功能請求

**Current behavior:**
當一個功能請求被拒絕時，issue 會被加上 `wontfix` 標籤並附一則留言後關閉。
這個決策或理由沒有持久的紀錄。未來類似的請求需要維護者回想
或搜尋先前的討論。

**Desired behavior:**
被拒絕的功能請求應該記錄在 `.out-of-scope/<concept>.md` 檔案中，
記錄決策、理由，以及所有請求過該功能之 issue 的連結。
在分類新 issue 時，應該檢查這些檔案是否有相符的。

**Key interfaces:**
- `.out-of-scope/` 中的 Markdown 檔案格式：每個檔案應該有一個
  `# Concept Name` 標題、一行 `**Decision:**`、一行 `**Reason:**`，
  以及一份附 issue 連結的 `**Prior requests:**` 清單
- 分類流程應該在早期讀取所有 `.out-of-scope/*.md` 檔案，
  並依概念相似度把進來的 issue 與它們比對

**Acceptance criteria:**
- [ ] 把一個功能以 wontfix 關閉時，會在 `.out-of-scope/` 中建立或更新一個檔案
- [ ] 檔案包含決策、理由，以及指向已關閉 issue 的連結
- [ ] 如果已經存在相符的 `.out-of-scope/` 檔案，新 issue 會被附加到
      它的「Prior requests」清單，而不是建立重複的檔案
- [ ] 分類期間，會檢查既有的 `.out-of-scope/` 檔案，並在新 issue
      與先前的拒絕相符時把它提出來

**Out of scope:**
- 自動比對（由人確認是否相符）
- 重新開啟先前被拒絕的功能
- Bug 回報（只有功能增強的拒絕會進入 `.out-of-scope/`）
```

### 好的 agent 簡報（PR）

對 PR 而言，「Current behavior」描述的是 diff 的狀態，而簡報要求 agent 把它完成或修正，而不是從零開始建立。

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** 完成貢獻者為 `triage list` 加的 `--json` 輸出旗標

**Current behavior:**
這個 PR 加入了一個 `--json` 旗標，把 issue 清單序列化成 JSON。正常路徑
可以運作，而且 diff 符合專案的指令結構。還剩兩個缺口：錯誤仍以
給人看的文字輸出（不是 JSON），而且新的旗標沒有測試涵蓋。

**Desired behavior:**
使用 `--json` 時，所有輸出（包含錯誤）都是 stdout 上格式正確的 JSON，
而且指令的結束碼不變。沒有這個旗標時，既有的人類可讀輸出完全不受影響。

**Key interfaces:**
- 在 `--json` 下，指令的錯誤路徑應該輸出 `{ "error": string }`，
  而不是純文字的錯誤
- 重複使用 PR 已經加入的序列化器；不要再引入第二個

**Acceptance criteria:**
- [ ] `triage list --json` 在成功與錯誤的情況下都輸出有效的 JSON
- [ ] 結束碼與非 JSON 的指令相同
- [ ] 有一個測試涵蓋 `--json` 的成功輸出，以及一個錯誤情況
- [ ] 預設（非 JSON）的輸出逐位元組完全不變

**Out of scope:**
- 為其他任何指令加入 `--json`
- 改變 PR 已經定義好的成功 payload 的 JSON 結構
```

### 不好的 agent 簡報

```markdown
## Agent Brief

**Summary:** 修好 triage 的 bug

**What to do:**
triage 那個東西壞了。看一下主檔案然後把它修好。
大約在第 150 行的那個函式有問題。

**Files to change:**
- src/triage/handler.ts (line 150)
- src/types.ts (line 42)
```

這很糟糕，因為：
- 沒有類別
- 描述模糊（「triage 那個東西壞了」）
- 引用了會過時的檔案路徑與行號
- 沒有驗收條件
- 沒有範圍邊界
- 沒有描述目前行為與期望行為
