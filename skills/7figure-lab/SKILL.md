---
name: 7figure-lab
description: "数学建模图表实验室（多工具画图）。承接 3coding-visual 与 4flow-diagram，负责它们未覆盖的几何/坐标/参数关系示意图和 HTML 交互图，并对关键图用 Python、HTML（SVG/交互）分别生成多版，输出 FIGURES_MENU.md 图单供用户最终挑选。"
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Agent, WebSearch, WebFetch
---

# 图表实验室（Python / HTML 多版本画图）

本 skill 是画图增强阶段，与 `3coding-visual`（数据图）和 `4flow-diagram`（流程图）互补。核心目标：不单调只用一种工具，发挥不同工具的优势，关键图各出多版，最终由用户挑选。

## 定位与阶段边界

- `3coding-visual` 负责由计算结果生成的数据图（matplotlib / seaborn / plotly）。
- `4flow-diagram` 负责技术路线图、求解流程图、体系结构图（HTML 内嵌 SVG + PDF）。
- `7figure-lab` 负责：
  1. 前两者未覆盖的示意图：几何关系图、坐标系示意图、参数关系图、变量关系图。
  2. HTML 交互图（SVG / ECharts），供评审演示或可缩放查看。
  3. 对关键图执行「多工具各出一版」，并汇总成图单供用户挑选。

- 含公式的精确图（变量关系、概率图模型、带公式的模型结构）归 `5writing` 用 TikZ 绘制，本 skill 只画无公式的示意图。

本阶段不重跑模型、不改 `code/`、不改写 `reports/RESULTS_REPORT.md` 的数值结论。

## 数学建模规范参考

如需领域判断，读取 `../_references/math_modeling_norms.md` 的「图表与可视化」和「非数据图工具选择」小节。该文件只作规范知识库，不新增固定产出。

## 必须产出

```text
figures/
  fig_xxx_geom.pdf        # Python 画的几何/坐标/参数示意图
  fig_xxx_geom.html       # HTML/SVG 交互版
  ...
reports/FIGURES_MENU.md   # 图单：每张图的多版本清单 + 建议 + 状态
```

## 工作流

### Step 1: 盘点图需求

读取 `reports/ANALYSIS_MODELING_REPORT.md`、`reports/RESULTS_REPORT.md`、`reports/DIAGRAM_REPORT.md` 和 `figures/`，找出：

- 已被数据图 / 流程图覆盖的图（不重复生成）。
- 尚缺的示意图（几何关系、坐标系、参数关系、变量关系）。
- 值得多工具各出一版的关键图（至少：技术路线图 + 每个问题的核心结果图）。

### Step 2: 工具选型

| 图类型 | 首选 | 备选 | 输出 |
| --- | --- | --- | --- |
| 几何关系图 / 坐标示意图 | Python matplotlib | TikZ | PDF |
| 参数关系 / 变量关系图（含公式） | TikZ | Python matplotlib | PDF（LaTeX 内嵌） |
| 交互网络图 / 可缩放图 | HTML ECharts | Python plotly | HTML → SVG → PDF |
| 技术路线图 / 流程图 | HTML（SVG） | mermaid | PDF（已有则跳过） |

### Step 3: 多工具各出一版

对关键图分别用不同工具生成，保留全部版本（例如同一张核心结果图，同时用 matplotlib 画静态 PDF 版、用 ECharts 画 HTML 交互版）。生成时遵守提示词「三十五、图表视觉风格」：低饱和、学术化、可灰度区分，避免霓虹色和花哨渐变。

### Step 4: 写图单

在 `reports/FIGURES_MENU.md` 列出每张图的候选版本、工具、路径、适用建议，并标注「待用户挑选」。格式：

```markdown
# 图单（供最终挑选）

| 图 | 版本 | 工具 | 路径 | 建议 | 状态 |
| --- | --- | --- | --- | --- | --- |
| 核心结果图 A | 静态版 | matplotlib | figures/fig_A.pdf | 论文默认 | 待选 |
| 核心结果图 A | 交互版 | ECharts | figures/fig_A.html | 评审演示 | 待选 |
```

## 质量要求

- 数据图必须用真实结果，禁止用模拟数据冒充；科研绘图模板若基于模拟数据，只能作风格参考。
- 图中文字与论文语言一致，图内不写长标题，标题交给论文 caption。
- 统一输出矢量 PDF；HTML 版先导出 SVG 再转 PDF 嵌入论文。
- 每张图必须能对应 `reports/ANALYSIS_MODELING_REPORT.md` 中的真实方法或结果。
