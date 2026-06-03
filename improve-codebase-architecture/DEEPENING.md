# Deepening（模块深度改造规范）
说明在存在依赖的前提下，如何安全地将一组浅层 module 做深度封装。全文统一使用 [LANGUAGE.md](LANGUAGE.md) 定义术语：**module**、**interface**、**seam**、**adapter**。

## 依赖分类
评估待深度改造的候选模块时，先对依赖进行归类；依赖类型决定改造后如何依托 seam 开展测试。

### 1. In-process（进程内依赖）
纯计算逻辑、内存状态，无任何 I/O 操作。**可直接做深度封装**：合并零散 module，通过新 interface 直接编写测试，无需额外 adapter。

### 2. Local-substitutable（本地可替换依赖）
可使用本地替身组件完成测试的依赖（例如用 PGLite 替代 Postgres、内存文件系统替代真实磁盘）。存在可用替身即可深度改造；改造后在测试环境使用替身组件运行用例。Seam 置于模块内部，对外暴露的 external interface 不开放 port。

### 3. Remote but owned (Ports & Adapters)（自有远程依赖）
跨网络的自研内部服务（微服务、内部接口）。在 seam 处定义 **port（端口接口）**；深层 module 承载核心业务逻辑，通信实现以 **adapter** 形式注入。测试环境使用内存版 adapter，生产环境使用 HTTP/gRPC/消息队列 adapter。

推荐文案示例：
*"Define a port at the seam, implement an HTTP adapter for production and an in-memory adapter for testing, so the logic sits in one deep module even though it's deployed across a network."*

### 4. True external (Mock)（第三方外部依赖）
不受我方管控的第三方服务（Stripe、Twilio 等）。深度改造后的 module 通过注入 port 接入外部依赖，测试时传入 mock 类型的 adapter。

## Seam 设计约束
- **One adapter means a hypothetical seam. Two adapters means a real one.**
仅一个 adapter 代表预留的潜在 seam；具备两套及以上 adapter 才算成型可用的真实 seam。若无生产+测试至少两套落地实现，不新增 port；单一 adapter 的 seam 只会徒增间接调用层级。
- **Internal seams vs external seams**
深层 module 可同时拥有：仅内部实现可见、供自身单元测试使用的 internal seams，以及暴露在对外 interface 上的 external seam。不能因为单元测试需要就把内部 seam 暴露至对外接口。

## 测试策略：替换原有用例，不叠加分层用例
- 原有针对浅层 module 的单元测试在新增接口侧用例落地后失去价值，直接删除。
- 全部新用例编写在改造后 module 的 interface 层面。遵循 **interface is the test surface**。
- 用例只通过接口校验可观测的业务结果，不校验内部状态。
- 用例需要做到不受内部实现重构影响：用例描述业务行为而非实现细节。若实现改动就必须修改用例，代表测试越界穿透了 interface。