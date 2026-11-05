# Skill 的機制

這是 [`writing-for-agents`](SKILL.md) 中專屬於 skill 的分支：當文件是一個 skill 時有什麼不同（frontmatter、觸發方式的選擇，以及路由 skill）。撰寫它的其他一切，都是 `SKILL.md` 中的通用參考。

## 觸發方式（Invocation）

兩種選擇，在兩種負擔之間取捨：

- **模型觸發（model-invoked）**的 skill 保有 `description`，所以 agent 可以自主觸發它，其他 skill 也能使用它。你仍然可以輸入它的名稱：模型觸發一定*包含*使用者觸發；description 只會增加 agent 發現它的管道，從不會移除人類的觸發能力。description 是這個 skill 最上層的情境指標（context pointer），被迫隨時保持載入：以永久的情境負擔換取可被發現性。一個內容全是參考資料的模型觸發 skill，也是共用參考資料的一個存放處：其他 skill 可以觸發它，所以多個 skill 都需要的參考資料可以放在同一個地方。機制：省略 `disable-model-invocation`，並寫一段面向模型的 description，載明觸發的各個分支（`SKILL.md` 中撰寫指標的規則完全適用）。
- **使用者觸發（user-invoked）**的 skill 把 description 從 agent 的觸及範圍中移除：只有人類輸入它的名稱才能觸發，其他 skill 都不行。零情境負擔，但會花費認知負擔：你就是那個必須記得它存在的索引。機制：設定 `disable-model-invocation: true`；`description` 變成面向人類的：一行摘要，移除觸發清單。

只有當 agent 必須自己觸及這個 skill，或另一個 skill 必須觸及它時，才選擇模型觸發。如果它只會被手動觸發，就做成使用者觸發，不付任何情境負擔。

兩個使用者觸發的 skill 都需要的共用參考資料，不能放在其中任何一個裡：沒有 description，誰也觸發不了對方。把它移到 skill 系統之外的一般檔案：任何 skill 都能指向的外部參考資料。

## 依觸發方式拆分

拆分中的「觸發方式」切法（「順序」切法在 `SKILL.md` 中）：當你有一個獨特的主導詞（leading word）應該能單獨觸發它（一個你在提示中真的會用到的觸發詞），或另一個 skill 必須觸及它時，就拆出一個模型觸發的 skill。你要為新的、永遠載入的 description 付出情境負擔，所以這份獨立的觸及能力必須值得。

## 路由 skill（Router skills）

當使用者觸發的 skill 多到你記不住時，這種累積起來的認知負擔可以用一個**路由 skill** 來解決：一個使用者觸發的 skill，列出其他 skill 的名稱以及何時該使用每一個，讓人類只需要記住一個 skill，而不是很多個。它只能提示，永遠無法觸發它們：使用者觸發的 skill 沒有 description，所以除了人類，沒有任何東西能觸及它們。
