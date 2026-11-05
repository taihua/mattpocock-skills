---
name: implement
description: "根據 spec 或一組 ticket 實作一項工作。"
disable-model-invocation: true
---

實作使用者在 spec 或 ticket 中描述的工作。

如果使用者傳入一個 ticket 的引用，就從 issue tracker 取得它，並在開始前說出它的標題。如果引用有歧義，就詢問。

在可能的地方，於事先約定的接縫（seam）上，以 "tdd" 呼叫 Skill 工具。

定期執行型別檢查、定期執行單一測試檔，並在最後執行一次完整的測試套件。

完成後，以 "code-review" 呼叫 Skill 工具來審查這項工作。

把你的工作 commit 到目前的分支。
