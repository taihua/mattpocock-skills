# Productivity（生產力）

通用的工作流程工具，不限於程式碼。

## 使用者觸發（User-invoked）

只有在你親自輸入名稱時才能觸發（Claude Code：`disable-model-invocation: true`；Codex：在 `agents/openai.yaml` 中設定 `policy.allow_implicit_invocation: false`）。

- **[grill-me](./grill-me/SKILL.md)**：針對一個計畫或設計接受毫不留情的連續提問，直到設計樹（design tree）的每一個分支都被解決。
- **[handoff](./handoff/SKILL.md)**：把目前的對話壓縮成一份交接文件，讓另一個 agent 可以接手繼續工作。
- **[teach](./teach/SKILL.md)**：以多次課程的方式教使用者一項新技能或新概念，並把目前的目錄當作有狀態（stateful）的教學工作區。
- **[to-questionnaire](./to-questionnaire/SKILL.md)**：把一個你無法獨自回答的決策，轉成一份 Markdown 問卷，交給唯一能回答的那個人（可以非同步填寫，也可以在會議中一起填）。
- **[wait-what](./wait-what/SKILL.md)**：當某則訊息讓你看不懂時立刻使用。agent 會補上你缺少的背景，用淺白的英文、並使用你 `GLOSSARY.md` 裡的詞彙重新說明一次。

## 模型觸發（Model-invoked）

模型或使用者都可以觸發（description 中寫了豐富的觸發語句，讓模型能主動使用它們）。

- **[grilling](./grilling/SKILL.md)**：針對一個計畫、決策或想法，毫不留情地連續訪問使用者，直到設計樹的每一個分支都被解決。
- **[writing-for-agents](./writing-for-agents/SKILL.md)**：撰寫給 agent 看的文件：skill、AGENTS.md/CLAUDE.md，以及任何 agent 會透過指標（pointer）讀到的文件。
