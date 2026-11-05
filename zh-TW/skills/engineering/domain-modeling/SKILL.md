---
name: domain-modeling
description: 建立並磨利專案的領域模型。在討論程式碼庫的術語、撰寫或編輯 GLOSSARY.md，或記錄、編輯 ADR 時使用。
---

# 領域建模（Domain Modeling）

在設計的同時，主動建立並磨利專案的領域模型。這是*主動*的紀律：質疑術語、發想邊界情境，並在詞彙表與決策一成形時就立刻寫下來。（單純*讀取* `GLOSSARY.md` 來取得詞彙並不是這個 skill：那是任何 skill 都能做的一行習慣。這個 skill 用在你要*改變*模型的時候，而不只是使用它。）

## 檔案結構

大多數 repo 只有單一 context：

```
/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目錄有 `GLOSSARY-MAP.md`，代表這個 repo 有多個 context。這份地圖會指出每一個 context 放在哪裡：

```
/
├── GLOSSARY-MAP.md
├── docs/
│   └── adr/                          ← 系統層級的決策
├── src/
│   ├── ordering/
│   │   ├── GLOSSARY.md
│   │   └── docs/adr/                 ← 特定 context 的決策
│   └── billing/
│       ├── GLOSSARY.md
│       └── docs/adr/
```

延遲建立檔案：只有在你有東西要寫時才建立。如果沒有 `GLOSSARY.md`，在第一個術語確定時建立。如果沒有 `docs/adr/`，在需要第一份 ADR 時建立。

## 在 session 進行中

### 以 glossary 為準提出質疑

當使用者用了一個與 `GLOSSARY.md` 既有用語衝突的術語時，立刻指出來。「你的 glossary 把『cancellation（取消）』定義為 X，但你似乎指的是 Y。到底是哪一個？」

### 磨利模糊的用語

當使用者用了模糊或多義的術語時，提出一個精確的標準術語。「你說的『account』：是指 Customer 還是 User？這兩個是不同的東西。」

### 討論具體情境

在討論領域關係時，用具體的情境來做壓力測試。發想能探測邊界情況的情境，逼使用者把概念之間的邊界講精確。

### 與程式碼交叉比對

當使用者說明某件事如何運作時，檢查程式碼是否一致。如果發現矛盾，就把它提出來：「你的程式碼是取消整張 Order，但你剛才說可以部分取消。哪一個才對？」

### 即時更新 GLOSSARY.md

當一個術語確定時，當場就更新 `GLOSSARY.md`。不要累積起來一次處理：發生時就記錄。使用 [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md) 中的格式。

`GLOSSARY.md` 應該完全不含任何實作細節。不要把 `GLOSSARY.md` 當成 spec、草稿區，或存放實作決策的地方。它就是一份詞彙表，僅此而已。

### 節制地提議 ADR

只有在以下三個條件全部成立時，才提議建立 ADR：

1. **難以逆轉**：之後改變主意的成本是有意義的
2. **沒有背景就會令人意外**：未來的讀者會納悶「他們為什麼要這樣做？」
3. **是真正取捨的結果**：當時確實有其他選擇，而你基於特定理由選了其中一個

只要三者缺一，就跳過 ADR。使用 [ADR-FORMAT.md](./ADR-FORMAT.md) 中的格式。
