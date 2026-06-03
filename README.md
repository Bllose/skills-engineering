# Skills Engineering

Claude Code 自定义技能集，覆盖软件工程全流程：行为准则、需求评审、架构优化、TDD 开发、故障排查、PRD 生成。

## 技能清单

| 技能 | 入口 | 用途 |
|---|---|---|
| **karpathy-guidelines** | `/karpathy-guidelines` | 规避大模型编码常见错误的行为准则：避免过度设计、精准小范围改动、明示隐含前提、定义可校验的验收标准 |
| **diagnose** | `/diagnose` | 疑难 Bug 与性能退化标准化排查：复现→猜想→埋点→修复→回归 |
| **grill-with-docs** | `/grill-with-docs` | 以"拷问式"评审对技术方案进行压力测试，对照 CONTEXT.md 打磨术语，落地 ADR |
| **improve-codebase-architecture** | `/improve-codebase-architecture` | 挖掘架构优化切入点，将浅层 Module 改造为深层 Module，提升可测试性与 AI 可读性 |
| **tdd** | `/tdd` | Red-Green-Refactor 循环的测试驱动开发，纵向切片、探针式迭代 |
| **to-prd** | `/to-prd` | 基于当前会话上下文生成 PRD，发布至项目需求看板 |
| **zoom-out** | `/zoom-out` | 提升抽象层级，从架构视角梳理模块与调用方关系 |

## 技能间冲突与重叠说明

### karpathy-guidelines 与 tdd 的触发竞争

两者在 description 中都声明了"编写代码"类触发条件。当用户提出"在存量项目中新增一个功能"而不指定 skill 时，两个 skill 同时命中，模型可能随机二选一：

| 选中结果 | 行为差异 |
|---|---|
| **tdd** | 进入 red-green-refactor 循环，先写失败用例，再编码实现 |
| **karpathy-guidelines** | 进入编码规范模式，强调最简实现、精准改动，但不会强制执行 TDD 流程 |

**本质原因**：karpathy-guidelines 是元层面的行为准则，不是独立工作流。它和 tdd 不是"二选一"关系——理想情况下，TDD 流程中应当同时遵守 karpathy-guidelines。

**缓解建议**：如果两个 skill 同时部署，建议将 karpathy-guidelines 的 description 中的 `编写、评审、重构代码时启用` 收紧为更精准的触发词（如"这段代码是否过度设计？""审查代码是否有多余改动"），避免它和 tdd 争抢通用编码场景。或者将其移除 skill 机制，放入 CLAUDE.local.md 中作为 always-applied 行为约束（注意不要放入 CLAUDE.md，会被 `/init` 覆盖）。

### karpathy-guidelines 与 improve-codebase-architecture 的理念张力

| karpathy-guidelines (#3) | improve-codebase-architecture |
|---|---|
| "只改动必要代码，不重构无故障的原有逻辑" | 主动扫描浅层 Module，提出深度改造方案 |
| "发现无关无效代码仅做备注，不擅自删除" | 合并紧耦合模块，消除冗余间接层 |

两者适用于不同阶段：karpathy-guidelines 约束**日常增量开发**，improve-codebase-architecture 驱动**专项架构评审**。如果 description 不够精准，模型可能在架构评审场景误选 karpathy-guidelines，给出"不动存量代码"的建议，与评审目标相悖。

**缓解建议**：日常开发遵守 karpathy-guidelines 的最小改动原则；架构优化时显式调用 `/improve-codebase-architecture`，让它在独立上下文中工作，不受 karpathy-guidelines 约束。

## 特别提醒：Skill、CLAUDE.md、CLAUDE.local.md 的定位区分

Claude Code 项目中同时存在三种"承载信息、约束行为"的机制，极易混淆。三者生命周期和写入主体完全不同，选错载体会导致内容被覆盖或行为不符合预期。

### 对比

| | **CLAUDE.md** | **CLAUDE.local.md** | **Skill** |
|---|---|---|---|
| **定位** | 项目上下文 | 行为约束 | 可触发的工作流/规范 |
| **典型内容** | 项目结构、技术栈、构建命令、GitNexus 配置 | 编码准则、领域术语要求、禁止操作清单 | TDD 流程、故障排查步骤、架构评审方法 |
| **写入方式** | `/init` 自动生成并覆盖 | 用户手动编辑 | 用户手动创建 SKILL.md |
| **被覆盖风险** | **高** — `/init` 会全量重写 | **无** — `/init` 不触碰此文件 | **无** — 独立目录管理 |
| **生效时机** | 会话启动时自动加载 | 会话启动时自动加载 | 命中 description 触发条件时激活 |
| **是否参与 skill 竞争** | 否（始终生效） | 否（始终生效） | 是（多 skill 命中时随机选择） |

### 实际影响

假设你把 Karpathy 编码准则写入 CLAUDE.md：

1. 模型每次会话都遵守这些行为约束 ✓
2. 某天执行 `/init` 刷新项目信息 → **Karpathy 准则被一并抹掉** ✗

两种内容的生命周期完全不同：项目结构信息需要随代码库迭代而刷新；行为约束需要长期稳定、不受自动化工具干扰。

### 正确做法

把两种性质的内容分开存放：

```
my-project/
├── CLAUDE.md              ← /init 生成（项目结构、技术栈、GitNexus）
├── CLAUDE.local.md        ← 手动维护（Karpathy 准则、项目特定禁止项）
└── .claude/
    └── skills/            ← 独立工作流（tdd、diagnose、grill-with-docs 等）
```

**核心原则**：行为约束类内容（编码准则、工作规范）一律写入 CLAUDE.local.md 或独立 skill；CLAUDE.md 仅用于自动生成的项目上下文信息，不与手动维护的内容混合。

## 技能分类与部署建议

Claude Code 支持两种技能部署位置：

| 部署位置 | 路径 | 作用域 |
|---|---|---|
| **全局** | `~/.claude/skills/` | 所有项目均可调用 |
| **项目** | `<project>/.claude/skills/` | 仅当前项目可调用 |

### 适合全局部署

这类技能提供通用工程方法论，不依赖特定项目的文档或代码结构，放在全局目录下跨项目复用：

- **karpathy-guidelines** — 通用编码行为准则，源自 Andrej Karpathy 对 LLM 编码弊病的总结。零项目依赖，适合全局部署作为一种"基础约束层"，在所有项目中生效。
- **diagnose** — 标准化排查流程（复现→猜想→埋点→修复），与具体项目解耦。附带 `scripts/hitl-loop.template.sh` 用于人工介入的循环复现场景。
- **zoom-out** — 纯粹的行为指令（`disable-model-invocation: true`），指示代理切换抽象层级，零项目依赖。

部署方式：

```bash
# 将技能目录复制到全局 skills 目录
cp -r karpathy-guidelines ~/.claude/skills/
cp -r diagnose ~/.claude/skills/
cp -r zoom-out ~/.claude/skills/
```

### 适合项目级部署

这类技能深度依赖项目自身的领域文档（CONTEXT.md）和架构决策记录（docs/adr/），放在目标项目内才能发挥完整能力：

- **grill-with-docs** — 直接读写项目内的 `CONTEXT.md` 和 `docs/adr/`，是项目领域知识的"校对器"。
- **improve-codebase-architecture** — 依托项目的 CONTEXT.md 术语和 ADR 决策来挖掘优化点，术语体系与项目绑定。
- **tdd** — 用例命名、接口设计需对齐项目领域术语和 ADR 决策；TDD 方法论本身通用，但高质量产出依赖项目上下文。
- **to-prd** — PRD 全文需使用项目 CONTEXT.md 术语，遵从对应模块的 ADR 决策。

部署方式：

```bash
# 将技能目录复制到目标项目的 .claude/skills/ 下
cp -r grill-with-docs ~/my-project/.claude/skills/
cp -r improve-codebase-architecture ~/my-project/.claude/skills/
cp -r tdd ~/my-project/.claude/skills/
cp -r to-prd ~/my-project/.claude/skills/
```

### 灵活部署

部分技能同时适合两种位置，取决于使用习惯：

- **diagnose** 和 **tdd**：方法论本身是通用的，放在全局即可跨项目使用；若希望诊断报告/测试用例的命名严格对齐某项目的领域术语，也可额外放在该项目下。
- **to-prd**：如果所有项目共用同一套需求看板配置，全局一份即可；若不同项目看板配置不同，则按项目部署。
- **karpathy-guidelines**：全局部署即可覆盖所有项目。若有项目需要特殊的行为约束规则，可在项目级 CLAUDE.md 中补充，无需按项目复制。

### 前置依赖

以下技能在调用前，目标项目需要具备对应文件：

| 技能 | 依赖项 | 说明 |
|---|---|---|
| karpathy-guidelines | 无 | 零依赖，即装即用 |
| grill-with-docs | `CONTEXT.md`、`docs/adr/` | 懒加载创建——首次使用时会自动新建 |
| improve-codebase-architecture | `CONTEXT.md`、`docs/adr/` | 读取现有文档作为优化参照 |
| tdd | `CONTEXT.md`（可选） | 有则对齐术语，无则按通用方式执行 |
| to-prd | 需求看板配置 | 缺失时执行 `/setup-matt-pocock-skills` 完成配置 |

### 部署后的目录结构示例

全局部署后：

```
~/.claude/skills/
├── karpathy-guidelines/
│   └── SKILL.md
├── diagnose/
│   ├── SKILL.md
│   └── scripts/
│       └── hitl-loop.template.sh
└── zoom-out/
    └── SKILL.md
```

项目级部署后：

```
my-project/
├── .claude/
│   └── skills/
│       ├── grill-with-docs/
│       │   ├── SKILL.md
│       │   ├── CONTEXT-FORMAT.md
│       │   └── ADR-FORMAT.md
│       ├── improve-codebase-architecture/
│       │   ├── SKILL.md
│       │   ├── LANGUAGE.md
│       │   ├── DEEPENING.md
│       │   ├── INTERFACE-DESIGN.md
│       │   └── HTML-REPORT.md
│       ├── tdd/
│       │   ├── SKILL.md
│       │   ├── deep-modules.md
│       │   ├── interface-design.md
│       │   ├── mocking.md
│       │   ├── refactoring.md
│       │   └── tests.md
│       └── to-prd/
│           └── SKILL.md
├── CONTEXT.md          # grill-with-docs 首次运行时懒加载创建
└── docs/
    └── adr/            # 首次写入 ADR 时懒加载创建
```

## 关联关系

技能之间存在协作链路，karpathy-guidelines 作为基础行为约束层贯穿全过程：

```
karpathy-guidelines（基础行为约束：最简实现、精准改动、目标驱动）
    ↓
zoom-out（梳理架构全貌）
    ↓
grill-with-docs（评审方案、打磨术语、落地 ADR）
    ↓
improve-codebase-architecture（基于术语与决策挖掘优化点）
    ↓
tdd（在优化后的架构上 TDD 开发）
    ↓
diagnose（上线后故障排查 → 复盘时反馈给架构优化）
    ↓
to-prd（排查结论或新需求沉淀为 PRD）
```

## 仓库结构

```
skills-engineering/
├── karpathy-guidelines/              # 编码行为准则（来源：Karpathy）
│   └── SKILL.md
├── diagnose/                        # 故障排查
│   ├── SKILL.md
│   └── scripts/
│       └── hitl-loop.template.sh    # 人工介入循环复现脚本模板
├── grill-with-docs/                 # 方案评审 & 文档打磨
│   ├── SKILL.md
│   ├── CONTEXT-FORMAT.md            # CONTEXT.md 编写规范
│   └── ADR-FORMAT.md                # ADR 文档编写规范
├── improve-codebase-architecture/   # 架构优化
│   ├── SKILL.md
│   ├── LANGUAGE.md                  # 统一架构术语规范
│   ├── DEEPENING.md                 # 模块深度改造规范
│   ├── INTERFACE-DESIGN.md          # 接口设计方案
│   └── HTML-REPORT.md               # HTML 报告格式规范
├── tdd/                             # 测试驱动开发
│   ├── SKILL.md
│   ├── deep-modules.md              # 深层模块设计规范
│   ├── interface-design.md          # 面向可测试性的接口设计
│   ├── mocking.md                   # Mock 使用规范
│   ├── refactoring.md               # 重构候选项清单
│   └── tests.md                     # 优劣用例区分规范
├── to-prd/                          # PRD 生成
│   └── SKILL.md
├── zoom-out/                        # 抽象层级提升
│   └── SKILL.md
├── .gitignore
└── README.md
```
