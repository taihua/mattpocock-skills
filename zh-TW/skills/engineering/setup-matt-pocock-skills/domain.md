# 領域文件（Domain Docs）

engineering skill 在探索程式碼庫時，應該如何使用這個 repo 的領域文件。

## 探索之前，先讀這些

- repo 根目錄的 **`GLOSSARY.md`**，或
- repo 根目錄的 **`GLOSSARY-MAP.md`**（如果存在）：它會指向每個 context 各一份的 `GLOSSARY.md`。閱讀與主題相關的每一份。
- **`docs/adr/`**：閱讀與你即將工作的領域有關的 ADR。在多 context 的 repo 中，也要檢查 `src/<context>/docs/adr/` 中屬於特定 context 的決策。

如果這些檔案有任何一個不存在，**安靜地繼續進行**。不要特別指出它們不存在；不要一開始就建議建立它們。`/domain-modeling` skill（透過 `/grill-with-docs` 與 `/improve-codebase-architecture` 觸及）會在術語或決策真的被確定時延遲建立它們。

## 檔案結構

單一 context 的 repo（大多數 repo）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多 context 的 repo（根目錄存在 `GLOSSARY-MAP.md`）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← 系統層級的決策
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← 特定 context 的決策
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 使用詞彙表的用語

當你的產出中提到某個領域概念時（在 issue 標題、重構提案、假設、測試名稱中），使用 `GLOSSARY.md` 中定義的術語。不要偏離成詞彙表明確要避免的同義詞。

如果你需要的概念還不在詞彙表中，這就是一個訊號：要嘛你在發明專案不使用的語言（重新考慮），要嘛真的存在一個缺口（記下來，留給 `/domain-modeling`）。

## 標示 ADR 衝突

如果你的產出與既有的 ADR 矛盾，要明確提出來，而不是默默地覆蓋：

> _與 ADR-0007（事件溯源的訂單）矛盾，但值得重新討論，因為……_
