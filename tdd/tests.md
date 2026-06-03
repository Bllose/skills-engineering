# Good and Bad Tests（优质用例与劣质用例区分规范）
## 优质用例
**集成测试风格**：依托真实 interface 开展测试，不对内部组件做 Mock。
```typescript
// 优质：校验可观测的业务行为
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

特点：
- 校验调用方/使用者关心的业务行为
- 仅使用对外 public API
- 内部代码重构不会导致用例失效
- 描述「做什么(WHAT)」，而非「如何实现(HOW)」
- 单个用例只包含一条业务逻辑断言

## 劣质用例
**绑定实现细节的用例**：与代码内部结构强耦合。
```typescript
// 劣质：绑定内部实现细节
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

劣质地标：
- 对内部依赖组件进行 Mock
- 测试私有方法
- 断言方法调用次数、调用顺序
- 业务逻辑无改动，仅重构代码就引发用例失败
- 用例命名描述实现方式(HOW)而非业务能力(WHAT)
- 绕过接口，通过外部途径校验结果

```typescript
// 劣质：绕过 interface 直接校验底层存储
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// 优质：全程通过 interface 做结果校验
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```