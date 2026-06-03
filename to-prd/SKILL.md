---
name: to-prd
description: 基于当前会话上下文生成PRD并发布至项目需求看板。用户提出依托现有内容编写PRD时启用。
---

# 功能说明
本能力依托当前对话内容与代码库现状信息产出PRD，**无需向用户补充问询**，直接整合已有信息。
需求看板与分类标签规范需提前配置，缺失时执行 `/setup-matt-pocock-skills`。

## 执行流程
1. 尚未梳理代码库时，先检索仓库摸清现有代码现状；PRD全文使用项目CONTEXT.md领域术语，遵从对应模块已落地的ADR决策。
2. 梳理功能对应的测试seam，优先复用已有seam，选用层级尽可能靠上层的点位；如需新增seam，在合理最高层级提出设计方案。
和用户确认拟定的seam符合预期。
3. 按下方模板编写PRD，完成后发布至项目需求看板，打上 `ready-for-agent` 分类标签，无需额外评审标记。

<prd-template>

## Problem Statement（问题说明）
站在使用者视角，描述当前面临的业务痛点。

## Solution（方案概述）
站在使用者视角，说明对应的落地解决方案。

## User Stories（用户故事）
长篇有序编号用户故事清单，单条固定格式：
1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>
用户故事需要尽可能详尽，覆盖该功能全场景。

## Implementation Decisions（落地技术决策）
已敲定的技术方案清单，包含：
- 新建/待修改的modules清单
- 对应模块需改动的interfaces
- 开发侧明确的技术细节约定
- 架构选型决策
- 数据表结构变更
- API契约定义
- 模块间交互规则

禁止填写具体文件路径与代码片段（极易随迭代失效）。
例外场景：原型代码可以比文字更精准固化设计（状态机、reducer、结构定义、类型约束），可在对应决策内精简嵌入关键代码片段，并备注源自原型，只保留决策相关核心片段，剔除可运行冗余演示代码。

## Testing Decisions（测试方案决策）
测试相关约定清单：
- 优质用例规范说明：仅校验对外业务行为，不绑定内部实现细节
- 待覆盖测试的modules
- 参考项目内同类现有用例规范

## Out of Scope（本期不纳入范围）
说明本PRD排除、不在迭代范围内的内容。

## Further Notes（补充备注）
功能相关其他补充说明。

</prd-template>