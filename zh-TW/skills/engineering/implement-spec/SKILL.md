---
name: implement-spec
description: "把 /to-spec 與 /to-tickets 的成果實作成程式碼。"
disable-model-invocation: true
---

你已經拿到一份 spec。這份 spec 應該有相關聯的 ticket，描述如何實作它。

issue tracker 應該已經提供給你了。如果沒有，請告訴使用者執行 `/setup-matt-pocock-skills`。

目標是在單一個**整合分支（integration branch）**上實作完整份 spec，並且每張 ticket 都以 issue tracker 結案工作的方式被解決。

這些 ticket 不是一串步驟。它們是一張**任務圖（task graph）**，彼此之間有阻擋關係。這表示永遠會有一個**前緣（frontier）**，由已經可以接手的 ticket 組成。

每一個子代理都要以全新的 context window 啟動。

與子代理之間的溝通應該精簡。主要透過**情境指標（context pointer）**溝通：指向 spec、ticket、研究筆記和先前的 commit。不要重複透過指標就能取得的資訊。

**實作子代理（implementer subagent）**應盡可能在背景執行，以達到最大的並行度。

## 步驟

1. 閱讀 spec 與 ticket，理解這張任務圖。

2. （選用）使用一個**探索子代理（exploration subagent）**來進行 ticket 所需的任何探索：相關的程式碼檔案或外部文件。確保探索子代理可以存檔：它應該把 markdown 筆記存在 repo 之外、所有後續子代理都能存取的目錄中。這讓**實作子代理**可以專注在實作，而不是探索。

3. 建立整合分支。如果 issue tracker 是透過 PR 來結案工作，或使用者要求開 PR，就在步驟 5 第一次合併之後開一個 draft PR（一個沒有領先 main 任何 commit 的分支無法開 PR），並標記為會關閉這份 spec 與這些 ticket。

4. 每張 ticket 使用一個全新的**實作子代理**，每個子代理都在自己的 worktree、自己的分支上工作。每個實作子代理：
   - 開始前確認自己的 worktree 是以整合分支為基礎，若不是就重設到整合分支上；
   - 以 `tdd` 呼叫 Skill 工具來實作這張 ticket；
   - 在回報完成之前，把整合分支的最新狀態合併進自己的分支

5. 當一個**實作子代理**完成後，使用一個**合併子代理（merger subagent）**把它的工作合併到整合分支。

6. 如果這改變了可接手 ticket 的**前緣**，就啟動更多**實作子代理**去處理新的 ticket。這能達到最大的並行度。

7. 所有 ticket 都完成後，在整合分支上以 `code-review` 呼叫 Skill 工具。用一個全新的**實作子代理**修正 code review 提出的所有問題。

8. 如果有 draft PR，把它標記為可審查（ready for review）。否則，以 issue tracker 結案工作的方式解決每張 ticket，並回報整合分支。

9. 清理所有**實作子代理**的 worktree。
