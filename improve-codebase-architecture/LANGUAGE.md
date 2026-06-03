# Language（统一术语规范）
本规范是架构优化产出内容的共用术语库。所有建议必须严格使用原文词汇，禁止替换为 component、service、API、boundary，统一用词是本规范的核心目的。

## 术语释义
**Module**
同时具备 Interface 与实现代码的代码单元。定义不限制代码粒度，函数、类、包、跨分层代码片段都属于 Module。
_Avoid_: unit, component, service

**Interface**
调用方正确使用 Module 所需掌握的全部信息。除类型签名外，还包含业务不变约束、调用顺序限制、异常类型、必填配置、性能特征。
_Avoid_: API, signature（释义片面，仅指代类型层面外露内容）

**Implementation**
Module 内部的代码本体。与 **Adapter** 概念区分：一个 Postgres 仓储可以是小巧 Adapter + 庞大 Implementation；内存模拟实现可以是庞大 Adapter + 精简 Implementation。讨论 Seam 相关内容时用 Adapter，其余场景使用 Implementation。

**Depth**
体现在 Interface 层面的复用收益：调用方只需学习少量接口，即可驱动大量业务逻辑。**Deep（深层）**：少量接口承载海量逻辑；**Shallow（浅层）**：接口复杂度和内部实现几乎持平。

**Seam** _(源自 Michael Feathers)_
无需修改当前位置代码即可替换业务逻辑的点位，是 Module 的 Interface 所在位置。Seam 的选址是独立设计决策，和接口背后的实现内容互不绑定。
_Avoid_: boundary（易和 DDD 的限界上下文概念混淆重载）

**Adapter**
挂载在 Seam 上、实现对应 Interface 的具体实现体。描述的是**角色（占位插槽）**，而非内部实现细节。

**Leverage**
调用方从 Depth 中获得的收益：用少量学习成本的接口获得丰富能力，一套实现可支撑 N 处调用、M 组测试用例。

**Locality**
维护人员从 Depth 中获得的收益：代码改动、缺陷、业务知识、校验逻辑收敛在一处，不会分散在各个调用方；改一处即全量修复。

## 设计原则
- **Depth is a property of the interface, not the implementation.**
Depth 隶属于 Interface 而非内部实现。深层 Module 内部可由多个小型、可 Mock、可替换的组件拼装，但这些内部组件不会暴露在接口中。Module 除了接口处的**external seam（外部接缝）**，还可拥有仅在内部实现、供自身单元测试使用的 **internal seams（内部接缝）**。
- **The deletion test（删除校验法）**
设想移除该 Module：如果整体复杂度直接消失，说明该 Module 仅做透传转发；如果复杂度四散到 N 个调用方，说明该 Module 具备封装价值。
- **The interface is the test surface.**
业务调用代码与测试代码共用同一条 Seam。若必须穿透接口才能完成测试，代表 Module 设计形态不合理。
- **One adapter means a hypothetical seam. Two adapters means a real one.**
仅存在单个 Adapter 代表预留的潜在 Seam；出现两个及以上 Adapter 才代表成型的真实 Seam。无实际多变场景时，不新增 Seam。

## 概念关联关系
- 一个 **Module** 有且仅有一套对外 **Interface**（面向调用方与测试的外露面）。
- **Depth** 是依附于 **Module** 的属性，以自身 **Interface** 作为衡量基准。
- **Seam** 是 **Module** 的 **Interface** 的挂载点位。
- **Adapter** 挂载于 **Seam** 并落地实现对应 **Interface**。
- **Depth** 为调用方带来 **Leverage**，为维护人员带来 **Locality**。

## 不采用的错误定义方式
- **Depth 按实现代码行数 / 接口代码行数比值判定（Ousterhout 理论）**：该方案会诱导刻意堆砌无效实现代码，本规范改用「复用收益」定义 Depth。
- 将 **Interface** 狭义等同于 TypeScript `interface` 关键字或类的公开方法：定义过窄，本文档里 Interface 包含调用方需要知晓的全部约束信息。
- 使用 **Boundary**：和 DDD 限界上下文概念重名歧义，统一改用 **seam** / **interface**。