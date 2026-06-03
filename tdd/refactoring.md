# Refactor Candidates（重构候选项）
TDD 迭代完成后，重点排查以下问题：

- **Duplication（重复代码）** → 抽取为函数/类
- **Long methods（超长方法）** → 拆分为私有辅助方法（测试仍只围绕 public interface）
- **Shallow modules（浅层模块）** → 合并或做深度改造
- **Feature envy（特性依恋）** → 将业务逻辑迁移至数据所属模块
- **Primitive obsession（基本类型迷恋）** → 封装成值对象（value objects）
- **Existing code（存量代码）**：新增代码暴露出设计缺陷的原有代码