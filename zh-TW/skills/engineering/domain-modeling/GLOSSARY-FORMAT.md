# GLOSSARY.md 格式

## 結構

```md
# {Context 名稱}

{用一到兩句話描述這個 context 是什麼，以及它為什麼存在。}

## Language

**Order（訂單）**:
{用一到兩句話描述這個術語}
_Avoid（避免使用）_: Purchase, transaction

**Invoice（發票）**:
在出貨後寄給客戶的付款請求。
_Avoid（避免使用）_: Bill, payment request

**Customer（客戶）**:
下訂單的個人或組織。
_Avoid（避免使用）_: Client, buyer, account
```

## 規則

- **要有主見。** 當同一個概念有多種說法時，選出最好的一個，其餘列在 `_Avoid_` 下。
- **定義要精簡。** 最多一到兩句話。定義它「是什麼」，而不是它「做什麼」。
- **只收錄這個專案 context 專屬的術語。** 一般的程式設計概念（timeout、錯誤類型、工具函式模式）即使專案大量使用，也不屬於這裡。加入術語前先問：這是這個 context 獨有的概念，還是一般的程式設計概念？只有前者屬於這裡。
- **自然形成群組時，用子標題分組術語。** 如果所有術語都屬於同一個緊密的領域，用平鋪的清單也可以。

## 單一 context 與多 context 的 repo

**單一 context（大多數 repo）：** repo 根目錄放一份 `GLOSSARY.md`。

**多個 context：** repo 根目錄放一份 `GLOSSARY-MAP.md`，列出各個 context、它們在哪裡，以及它們彼此的關係：

```md
# Glossary Map

## Contexts

- [Ordering](./src/ordering/GLOSSARY.md): 接收並追蹤客戶訂單
- [Billing](./src/billing/GLOSSARY.md): 產生發票並處理付款
- [Fulfillment](./src/fulfillment/GLOSSARY.md): 管理倉庫揀貨與出貨

## Relationships

- **Ordering → Fulfillment**: Ordering 發出 `OrderPlaced` 事件；Fulfillment 消費它們以開始揀貨
- **Fulfillment → Billing**: Fulfillment 發出 `ShipmentDispatched` 事件；Billing 消費它們以產生發票
- **Ordering ↔ Billing**: 共用 `CustomerId` 與 `Money` 型別
```

skill 會推斷適用哪一種結構：

- 如果 `GLOSSARY-MAP.md` 存在，讀取它來找出各個 context
- 如果只有根目錄的 `GLOSSARY.md`，就是單一 context
- 如果兩者都不存在，在第一個術語確定時，延遲建立根目錄的 `GLOSSARY.md`

當存在多個 context 時，推斷目前的主題與哪一個相關。如果不清楚，就詢問。
