# 何時該 Mock

只在**系統邊界**上 mock：

- 外部 API（金流、email 等）
- 資料庫（有時候；優先使用測試用 DB）
- 時間與隨機性
- 檔案系統（有時候）

不要 mock：

- 你自己的類別或模組
- 內部協作者
- 任何你能控制的東西

## 為可 mock 性而設計

在系統邊界上，設計容易 mock 的介面：

**1. 使用依賴注入**

把外部依賴傳進來，而不是在內部建立它們：

```typescript
// 容易 mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// 難以 mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. 偏好 SDK 風格的介面，而不是通用的 fetcher**

為每個外部操作建立特定的函式，而不是一個帶有條件邏輯的通用函式：

```typescript
// 好：每個函式都能獨立 mock
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// 不好：mock 時需要在 mock 內部寫條件邏輯
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

SDK 做法的意義是：
- 每個 mock 只回傳一種特定的形狀
- 測試的準備程式碼中沒有條件邏輯
- 更容易看出一個測試用到了哪些 endpoint
- 每個 endpoint 都有型別安全
