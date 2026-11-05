---
name: handoff
description: 把目前的對話壓縮成一份交接文件，讓另一個 agent 接手。
argument-hint: "下一個 session 會用來做什麼？"
disable-model-invocation: true
---

撰寫一份交接文件，摘要目前的對話，讓一個全新的 agent 可以繼續這項工作。存到使用者作業系統的暫存目錄（`$TMPDIR`，沒有的話用 `/tmp`；Windows 上是 `%TEMP%`），而不是目前的工作區。

在文件中加入一個「建議使用的 skill」段落，列出下一個 agent 應該用 Skill 工具呼叫哪些 skill。

不要重複已經記錄在其他產出物（spec、計畫、ADR、issue、commit、diff）中的內容。改用路徑或 URL 引用它們。

遮蔽任何敏感資訊，例如 API 金鑰、密碼或個人可識別資訊。

如果使用者有傳入參數，就把它當作下一個 session 要專注處理的事項說明，並據此調整文件內容。
