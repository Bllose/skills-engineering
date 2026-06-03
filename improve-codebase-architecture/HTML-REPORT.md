# HTML Report Format（HTML报告格式规范）
架构评审结果输出为**单个独立HTML文件**，存放至系统临时目录。页面样式与图表能力均通过CDN引入Tailwind、Mermaid。依赖链路类拓扑图优先使用Mermaid；分层剖面、体量对比等定制化视图采用原生div+内联SVG手写实现。两种方案搭配使用，避免全量依赖Mermaid造成图表样式同质化。

## 页面骨架
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* Tailwind不易直接实现的自定义样式：虚线seam边线、手绘风格箭头等 */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 头部Header
展示仓库名称、生成日期与简易图例：实心方框=module、虚线=seam、红色箭头=逻辑泄露、深色粗框=deep module。不添加开篇介绍文字，直接切入优化候选列表。

## 优化候选卡片
图表为信息主体，正文文字精简，严格沿用[LANGUAGE.md](LANGUAGE.md)规范术语，无需额外释义。
单条优化项包裹在一个`<article>`标签内，结构如下：
- **Title**：简短标题，点明优化方向（例：`Collapse the Order intake pipeline`）
- **标签栏**：优化优先级徽章（`Strong`=翠绿色、`Worth exploring`=琥珀色、`Speculative`=灰石板色）+ 依赖类型标签（`in-process`/`local-substitutable`/`ports & adapters`/`mock`）
- **Files**：等宽字体清单，样式`font-mono text-sm`
- **Before / After diagram**：卡片核心内容，左右双栏排布，参考下文绘图规范
- **Problem**：一句话说明现存痛点
- **Solution**：一句话说明改造方案
- **Wins**：条目列表，每条不超过6个英文单词；示例：`Tests hit one interface`、`Pricing logic stops leaking`、`Delete 4 shallow wrappers`
- **ADR备注栏（按需）**：琥珀色底色单行提示框

禁止大段描述文字；若图表需要大段文字才能看懂，则重新绘制图表。

## 图表绘制规范
按需选用、自由组合多种绘图样式，避免全部图表样式统一。
### 1. Mermaid拓扑图（依赖/调用链路首选）
梳理「X调用Y、Y调用Z、层级臃肿」场景时使用Mermaid `flowchart/graph`；时序对比（改造前6次跨层交互/改造后1次）选用时序图。外层套Tailwind卡片样式，通过classDef配置：泄露链路标红、深层模块深色填充。
```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```
### 2. 原生DIV+SVG手绘框图（Mermaid排版难以实现时）
用带边框div代表各个module，相对容器+绝对定位内嵌SVG线条/箭头。适合绘制：改造后单个粗边框deep module、内部实现灰化隐藏的效果图（该效果Mermaid难以精准控制粗细）。
### 3. 分层剖面图（浅层分层臃肿场景）
横向色块堆叠（`h-12 border-l-4`）模拟调用链路穿过多层代码；改造前：6层窄色块各司其职但逻辑单薄；改造后：合并为1个宽色块承载完整职责。
### 4. 面积体量图（接口与实现复杂度持平场景）
每个module绘制两个矩形：一块代表interface面积、一块代表implementation面积。改造前：接口矩形高度接近实现矩形（shallow浅层）；改造后：接口矮窄、实现高大（deep深层）。
### 5. 调用树收拢图
改造前：嵌套方框展开完整调用树；改造后：整棵树收拢为单个方框，原有内部调用在框内淡化展示。

## 排版配色规范
- 排版偏向轻量化文稿风，摒弃后台大屏仪表盘风格，留白充足；标题可选衬线字体`font-serif`搭配灰石色系。
- 配色克制：主点缀色选用翠绿/靛蓝，红色标记逻辑泄露、琥珀色标注警告。
- 图表高度统一约320px，保障改造前后双栏并排无需滚动。
- 图表内module标签：`text-xs uppercase tracking-wider`，图纸化标识，区别于普通页面文字。
- 页面仅引入Tailwind CDN与Mermaid ESM脚本，其余为静态页面，无自定义交互JS（Mermaid原生渲染除外）。

## 优先优化项板块
单独大号卡片：优化项名称+一句话选型理由+锚点跳转至对应候选卡片，内容从简。

## 行文规范
语言平实精简，架构名词严格照搬[LANGUAGE.md](LANGUAGE.md)。
### 强制用词
必须使用：module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality
### 禁用替换
禁止用component/service/unit代指module；禁止用API/signature代指interface；禁止用boundary代指seam；禁止用layer/wrapper随意指代module。
### 标准句式示例
- `Order intake module is shallow — interface nearly matches the implementation.`
- `Pricing leaks across the seam.`
- `Deepen: one interface, one place to test.`
- `Two adapters justify the seam: HTTP in prod, in-memory in tests.`

### Wins条目写法
必须依托规范术语描述收益：
`locality: bugs concentrate in one module`、`leverage: one interface, N call sites`、`interface shrinks; implementation absorbs the wrappers`
禁止模糊描述：`easier to maintain`、`cleaner code`等非规范话术。

行文杜绝铺垫、冗余客套；能缩写为列表项就不用整句正文，可删减条目直接移除；无规范术语可用时优先选用现有词汇，不自创新词。