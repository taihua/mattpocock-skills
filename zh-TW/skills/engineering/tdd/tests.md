# 好的測試與壞的測試

## 好的測試

**整合風格**：透過真正的介面測試，而不是 mock 內部的部件。

```typescript
// 好：測試可觀察的行為
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

特徵：

- 測試使用者或呼叫者在乎的行為
- 只使用公開 API
- 能撐過內部重構
- 描述「做什麼（WHAT）」，而不是「怎麼做（HOW）」
- 每個測試只有一個邏輯上的斷言

## 壞的測試

**實作細節測試**：與內部結構耦合。

```typescript
// 不好：測試實作細節
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

警訊：

- mock 內部協作者
- 測試私有方法
- 對呼叫次數或順序做斷言
- 在行為沒變的情況下重構，測試卻壞了
- 測試名稱描述的是「怎麼做」而不是「做什麼」
- 透過外部手段驗證，而不是透過介面

```typescript
// 不好：繞過介面來驗證
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// 好：透過介面驗證
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**同義反覆的測試**：期望值重述了實作，所以測試在構造上就一定會通過。

```typescript
// 不好：期望值用程式碼計算的同一種方式重新計算
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// 好：期望值是一個獨立、已知的字面值
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```
