---
name: improve-codebase-architecture
description: 依托 CONTEXT.md 中的领域术语与 docs/adr/ 内的决策记录，挖掘代码库架构优化切入点。适用于用户需要优化架构、寻找重构点、合并紧耦合模块、提升代码可测试性与 AI 代码可读性的场景。
---

# Improve Codebase Architecture（代码库架构优化）
梳理架构痛点，输出**深度优化机会点**：通过重构将浅层 Module 改造为深层 Module，最终目标是提升代码可测试性与 AI 可读性。

## 术语对照表
所有优化建议必须严格沿用下述用词，统一术语是核心要求，禁止随意改用 component、service、API、boundary 等替代词。完整释义参考 [LANGUAGE.md](LANGUAGE.md)。
- **Module**：具备 Interface 与实现逻辑的代码单元（函数、类、包、代码切片均可）。
- **Interface**：调用方使用该 Module 所需知晓的全部内容：数据类型、不变约束、异常类型、调用时序、配置项，不局限于函数签名。
- **Implementation**：Module 内部的实现代码。
- **Depth（深度）**：体现在 Interface 的复用价值：精简接口背后承载大量业务逻辑。**Deep（深层）** = 复用价值高；**Shallow（浅层）** = 接口复杂度和内部实现几乎持平。
- **Seam**：Interface 的挂载位置，可在不改动原有内部代码的前提下替换实现逻辑（固定使用本词，禁止改用 boundary）。
- **Adapter**：挂载在 Seam 处、实现对应 Interface 的具体实现类。
- **Leverage（复用收益）**：调用方从深层 Module 中获得的设计收益。
- **Locality（内聚性）**：维护人员从深层 Module 中获得的收益：代码变更、BUG、业务知识收敛在同一处。

核心设计原则（完整细则参考 [LANGUAGE.md](LANGUAGE.md)）：
- **Deletion test（删除校验法）**：设想移除当前 Module。若相关复杂度直接消失，则该 Module 仅做透传转发；若复杂度分散到多个上层调用方，则该 Module 具备存在价值。
- **The interface is the test surface.**：接口即测试边界。
- **One adapter = hypothetical seam. Two adapters = real seam.**：仅一个 Adapter 代表预留的潜在 Seam；存在两个及以上 Adapter 代表成型可用的真实 Seam。

本优化能力**依赖项目领域模型**：领域术语用于定义合理的 Seam 命名；已有 ADR 记录的历史决策不可推翻重议。

## 执行流程
### 1. 代码探查
优先阅读对应业务域的领域术语文档与相关 ADR 文件。
随后通过 Agent 工具、传入参数 `subagent_type=Explore` 遍历代码库，不依赖固化规则，以自然探查方式梳理架构痛点：
- 理解同一个业务概念需要跳转多个零散小 Module 的位置
- 存在大量**浅层 Module**：接口复杂度几乎等同于内部实现
- 为方便单测抽离纯函数，但实际业务缺陷集中在调用逻辑（缺少**Locality**内聚性）
- 紧耦合模块跨 Seam 出现逻辑泄露
- 缺少单测、或依托现有接口难以编写测试用例的代码区域

对疑似浅层 Module 执行**删除校验法**：移除后复杂度是被收拢收敛，还是向外扩散？能收拢复杂度即为有效优化目标。

### 2. 生成候选优化项 HTML 报告
在操作系统临时目录生成独立 HTML 文件，不写入项目代码仓；临时目录优先读取环境变量 `$TMPDIR`，无配置则降级为 `/tmp`（Windows 环境使用 `%TEMP%`），文件命名格式：`<tmpdir>/architecture-review-<timestamp>.html`，每次生成全新文件。
自动唤起文件：Linux 使用 `xdg-open <path>`、macOS 使用 `open <path>`、Windows 使用 `start <path>`，同时告知用户文件绝对路径。

报告规范：
- 页面样式通过 CDN 引入 Tailwind；依赖关系/时序类图表使用 CDN 引入 Mermaid
- 关联拓扑、调用链路、时序图使用 Mermaid；架构剖面、体量对比、折叠交互视图使用原生 div/SVG 手绘
- 每条优化候选项配套**优化前后对比视图**，突出可视化效果

单个候选项统一卡片模板：
- **Files**：涉及的文件/Module 清单
- **Problem**：当前架构引发的具体痛点
- **Solution**：优化方案文字描述
- **Benefits**：从内聚性、复用收益、测试优化三个维度说明优化价值
- **Before / After diagram**：左右分栏对比图，直观体现浅层问题与优化后的深层结构
- **Recommendation strength**：优先级标签，可选 `Strong` / `Worth exploring` / `Speculative`

报告末尾增加 **Top recommendation（优先优化项）**：标注首个推荐落地项与理由。

用词约束：
- 领域描述沿用 CONTEXT.md 定义词汇：例如写作「Order intake module」，禁止使用自定义 Handler 名称、Order service 等非规范命名
- 架构描述沿用 [LANGUAGE.md](LANGUAGE.md) 规范术语

**ADR 冲突处理**：若优化方案与现有 ADR 相悖，仅在痛点明显、值得重新审议历史决策时列入候选，并在卡片中标注警示：_“contradicts ADR-0007 — but worth reopening because…”_，不罗列所有被 ADR 限制的理论重构方案。

完整页面骨架、图表规范参考 [HTML-REPORT.md](HTML-REPORT.md)。

> 现阶段**不输出具体接口设计**，文件生成后询问用户：「Which of these would you like to explore?」

### 3. 方案深度研讨环节
用户选定优化项后，进入深度研讨：围绕约束条件、依赖关系、改造后深层 Module 结构、Seam 下层实现、兼容用例展开方案推演。

研讨过程中同步落地配套文档变更：
- **优化后新增领域概念、CONTEXT.md 无对应术语**：遵循 `/grill-with-docs` 规范补充词条至 CONTEXT.md，文件不存在则懒加载新建（参考 [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md)）
- **研讨中完善模糊的领域定义**：当场更新 CONTEXT.md 内容
- **用户因关键理由否决优化方案**：征询是否生成 ADR 留存结论，文案示例：_“Want me to record this as an ADR so future architecture reviews don't re-suggest it?”_；临时原因（当下没时间改造）、显而易见的理由无需生成 ADR，规范参考 [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md)
- **需要为深层 Module 设计多套备选接口**：参考 [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md)