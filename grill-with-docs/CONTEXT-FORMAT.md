# CONTEXT.md 格式规范

## 文档结构

```md
# {上下文名称}

{用一到两句话描述该上下文的用途与存在意义。}

## Language（术语规范）

**Order**:
{用一到两句话描述该术语释义}
_Avoid_: Purchase, transaction

**Invoice**:
交付完成后发给客户的付款索要单据。
_Avoid_: Bill, payment request

**Customer**:
下达订单的个人或组织机构。
_Avoid_: Client, buyer, account
```

## 编写规则
- **用词统一化**：同一概念存在多个表述时，选定最优用词，其余备选词汇统一列入`_Avoid_`。
- **释义精简**：释义最多一两句话，聚焦定义「事物本身」，不罗列行为作用。
- **仅收录业务专属术语**：通用编程概念（超时、异常类型、工具类设计模式等）即便项目高频使用也不纳入。新增术语前自查：该概念是否为本业务域独有？仅专属概念才可写入。
- **按需归类分组**：术语可按业务聚类拆分二级标题；全部术语归属同一业务域时，平铺罗列即可。

## 单/多上下文项目仓库区分
**单上下文（绝大多数项目）**：项目根目录仅存放一份 `CONTEXT.md`。

**多上下文项目**：项目根目录新建`CONTEXT-MAP.md`，罗列全部上下文、文件路径与依赖关联：
```md
# Context Map（上下文映射）

## Contexts（上下文清单）

- [Ordering](./src/ordering/CONTEXT.md) — 接收、跟踪客户订单
- [Billing](./src/billing/CONTEXT.md) — 开具发票、处理款项结算
- [Fulfillment](./src/fulfillment/CONTEXT.md) — 管理仓库拣货与发货

## Relationships（上下文关联关系）

- **Ordering → Fulfillment**: Ordering 发布 `OrderPlaced` 事件；Fulfillment 消费事件启动拣货
- **Fulfillment → Billing**: Fulfillment 发布 `ShipmentDispatched` 事件；Billing 消费事件生成发票
- **Ordering ↔ Billing**: 共用 `CustomerId`、`Money` 数据类型
```

工具自动判别项目结构：
- 存在`CONTEXT-MAP.md`：读取文件获取所有上下文位置
- 仅根目录存在`CONTEXT.md`：按单上下文处理
- 两者均无：首次解析术语时自动在根目录创建`CONTEXT.md`

多上下文场景下，自动匹配当前内容所属上下文；无法判定归属时主动询问确认。