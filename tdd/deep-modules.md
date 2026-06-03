# Deep Modules（深层模块规范）
源自《A Philosophy of Software Design》：

**Deep module（深层模块）** = 精简 interface + 厚重实现
```
┌─────────────────────┐
│   Small Interface   │  ← 方法少、参数简洁
├─────────────────────┤
│                     │
│                     │
│  Deep Implementation│  ← 复杂逻辑全部封装在内
│                     │
│                     │
└─────────────────────┘
```

**Shallow module（浅层模块）** = 庞大 interface + 单薄实现（应当规避）
```
┌─────────────────────────────────┐
│       Large Interface           │  ← 方法繁多、参数繁杂
├─────────────────────────────────┤
│  Thin Implementation            │  ← 仅做参数透传转发
└─────────────────────────────────┘
```

设计 interface 时自问三问：
- 能否缩减方法数量？
- 能否简化入参结构？
- 能否把更多复杂逻辑封装至内部？