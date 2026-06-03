# When to Mock（何时使用 Mock）

仅在**系统边界**处使用 Mock：
- 外部 API（支付、邮件等）
- 数据库（有时优先使用测试数据库）
- 时间 / 随机数
- 文件系统（有时）

**不要 Mock**：
- 你自己的类 / module
- 内部协作对象
- 任何由你控制的代码

## 可 Mock 性设计原则
在系统边界处，设计易于 Mock 的 interface：

### 1. 使用依赖注入
将外部依赖通过参数传入，而非在内部直接创建：
```typescript
// 易于 Mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// 难以 Mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

### 2. 优先使用 SDK 风格接口，而非通用请求器
为每个外部操作创建专用函数，而不是使用一个带条件逻辑的通用函数：
```typescript
// GOOD：每个函数可独立 Mock
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD：Mock 内部需要写条件逻辑
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

SDK 风格的优势：
- 每个 Mock 只返回一种特定结构
- 测试 setup 中无需条件逻辑
- 更容易看出测试调用了哪些接口
- 每个端点都具备类型安全保障