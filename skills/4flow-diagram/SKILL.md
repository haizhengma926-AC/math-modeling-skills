---
name: 4flow-diagram
description: "数学建模非数据型图示绘制阶段。根据 ANALYSIS_MODELING_REPORT.md、RESULTS_REPORT.md 和已有 figures/ 生成技术路线图、子问题求解流程图、模型结构图、数据处理流程图等 HTML（内嵌 SVG）图，并导出论文可引用 PDF。"
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Agent, WebSearch, WebFetch
---

# HTML 非数据图示绘制（流程图 / 架构图 / 概念图）

本 skill 承接 `3coding-visual`。它只负责论文中**无公式的非数据型图示**，例如技术路线图、求解流程图、数据处理流程图、指标体系图、决策树等；含公式的模型结构图、变量关系图交给 `5writing` 用 TikZ 绘制。

> 这些图一律用 **HTML（内嵌 SVG）代码** 手写绘制，再用无头浏览器或 SVG 转 PDF 工具导出矢量 PDF。

## 数学建模规范参考

如需领域判断，读取 `../_references/math_modeling_norms.md` 中的“图表与可视化”和“非数据图工具选择”小节。该文件只作为规范知识库，不要求为了凑数量生成额外图示。

## 阶段边界

- 本阶段负责：HTML（内嵌 SVG）源文件、非数据图 PDF、图示生成记录。
- 本阶段不负责：折线图、柱状图、散点图、热力图、箱线图、雷达图等数据图。这些由 `3coding-visual` 生成。
- 本阶段不负责：含数学公式/符号需精确标注的图（变量关系图、概率图模型、贝叶斯网络、带公式的模型结构图）。这些交给 `5writing` 用 TikZ 在 LaTeX 中直接绘制，与正文同字体、公式可引用。
- 本阶段不重跑模型、不修改 `code/`，不改写 `reports/RESULTS_REPORT.md` 的数值结论。

## 必须产出

在当前工作目录创建或更新：

```text
figures/
  fig_roadmap.html     # HTML 源文件（内嵌 SVG）
  fig_roadmap.pdf      # 论文引用用的矢量 PDF
  fig_flow_q1.html
  fig_flow_q1.pdf
  ...
reports/DIAGRAM_REPORT.md
```

如果某类图不需要生成，必须在 `reports/DIAGRAM_REPORT.md` 中说明原因。竞赛论文通常至少需要一张 `fig_roadmap` 技术路线图。

## 工作流程

### Step 1: 盘点已有图表和需求

先读取以下文件（存在则读取）：`reports/ANALYSIS_MODELING_REPORT.md`、`reports/RESULTS_REPORT.md`、`figures/` 目录列表。

然后从前序文档提取非数据图需求，输出一个清单：

```text
FIG PLAN CHECKLIST:
[ ] fig_roadmap      技术路线图，放在问题分析末尾
[ ] fig_flow_q1      问题一求解流程图
[ ] fig_flow_q2      问题二求解流程图
[ ] fig_flow_q3      问题三求解流程图
[ ] fig_pipeline     数据处理流程图
[ ] fig_model        模型结构/变量关系图
```

清单不是固定模板，要根据题目实际删减或增补。不要为了凑图生成无意义图示。

### Step 2: 判定图类型

常见图示选择：

| 图类型 | 文件名建议 | 适用场景 |
| --- | --- | --- |
| 技术路线图 | `fig_roadmap` | 展示整体解题路线、章节逻辑、方法串联 |
| 子问题求解流程图 | `fig_flow_q1`, `fig_flow_q2` | 展示单个子问题的输入、判断、算法、输出 |
| 数据处理流程图 | `fig_pipeline` | 展示数据清洗、特征构造、建模输入 |
| 模型结构图 | `fig_model` | 展示模块关系、变量关系、模型层次 |
| 指标体系图 | `fig_index_system` | 展示目标层、准则层、指标层 |
| 决策树/规则图 | `fig_decision_tree` | 展示分类规则、设备选择、策略分支 |

不要用 HTML 画这些数据图：

- 结果对比柱状图
- 预测误差曲线
- 灵敏度曲线
- 相关性热力图
- 分布图和箱线图

也不要用 HTML 画含公式的精确图（交给 `5writing` 用 TikZ）：

- 带公式的变量关系图
- 概率图模型 / 贝叶斯网络
- 带公式的模型结构图（如 Logistic 回归、LDA 结构）

### Step 3: 生成 HTML（内嵌 SVG）源文件

每张图一个 `.html` 文件，放在 `figures/`。HTML 文件自包含、零外部依赖：外层是极简 HTML 外壳（只做预览背景和画布尺寸控制），真正绘制内容是一段内嵌 `<svg>`。

生成时必须严格遵守下方「国奖级 HTML 流程图绘制规范」，并使用其末尾的提示词模板逐张生成。

生成大 SVG 时，分段写入，避免截断。示例（骨架）：

```bash
mkdir -p figures
cat << 'HTMLEOF' > figures/fig_roadmap.html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<style>
  @page { size: 900px 640px; margin: 0; }
  html, body { margin: 0; padding: 0; background: #ffffff; }
  svg { display: block; }
</style>
</head>
<body>
<svg viewBox="0 0 900 640" xmlns="http://www.w3.org/2000/svg"
     font-family="SimSun, 'Microsoft YaHei', sans-serif">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#666666"/>
    </marker>
  </defs>
  <!-- 画布浅色背景与外框 -->
  <rect x="0" y="0" width="900" height="640" fill="#FAFBFD" stroke="#CFD6E4" stroke-width="2"/>

  <!-- 起点（胶囊） -->
  <rect x="360" y="60" width="180" height="56" rx="28" fill="#E8F0FA" stroke="#2F5597" stroke-width="2"/>
  <text x="450" y="88" text-anchor="middle" dominant-baseline="central" font-size="18" fill="#1F1F1F">读取数据</text>

  <!-- 连线 -->
  <line x1="450" y1="116" x2="450" y2="160" stroke="#666666" stroke-width="2" marker-end="url(#arrow)"/>

  <!-- 处理步骤（圆角矩形） -->
  <rect x="360" y="160" width="180" height="56" rx="8" fill="#E8F0FA" stroke="#2F5597" stroke-width="2"/>
  <text x="450" y="188" text-anchor="middle" dominant-baseline="central" font-size="17" fill="#1F1F1F">数据预处理</text>

  <line x1="450" y1="216" x2="450" y2="260" stroke="#666666" stroke-width="2" marker-end="url(#arrow)"/>

  <!-- 判断（菱形） -->
  <polygon points="450,260 560,320 450,380 340,320" fill="#F7E8C8" stroke="#B8934A" stroke-width="2"/>
  <text x="450" y="320" text-anchor="middle" dominant-baseline="central" font-size="16" fill="#1F1F1F">是否收敛？</text>
</svg>
</body>
</html>
HTMLEOF
```

### Step 4: 导出 PDF

优先用无头浏览器（Windows 自带 Edge / Chrome）把 HTML 打印为矢量 PDF：

```bash
HTML2PDF="$(command -v msedge 2>/dev/null || command -v chrome 2>/dev/null || command -v chromium 2>/dev/null || true)"
if [ -z "$HTML2PDF" ] && [ -f "/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" ]; then
  HTML2PDF="/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
elif [ -z "$HTML2PDF" ] && [ -f "/c/Program Files/Google/Chrome/Application/chrome.exe" ]; then
  HTML2PDF="/c/Program Files/Google/Chrome/Application/chrome.exe"
fi

if [ -n "$HTML2PDF" ]; then
  "$HTML2PDF" --headless --disable-gpu --no-pdf-header-footer \
    --print-to-pdf="figures/fig_roadmap.pdf" "figures/fig_roadmap.html"
else
  echo "无头浏览器未找到，改用 cairosvg 兜底或记录导出失败。"
fi
```

兜底方案（浏览器不可用时）：把内嵌 `<svg>` 块抽成 `figures/fig_roadmap.svg`，再用 cairosvg 转 PDF：

```bash
pip install cairosvg
python -c "import cairosvg; cairosvg.svg2pdf(url='figures/fig_roadmap.svg', write_to='figures/fig_roadmap.pdf')"
```

如果无法导出 PDF，保留 `.html`，在 `reports/DIAGRAM_REPORT.md` 记录失败原因和建议导出命令。

### Step 5: 自检和修复

每张图必须检查：

- `.html` 文件非空，且内嵌 `<svg>` 结构完整、无未闭合标签。
- 若导出成功，`.pdf` 文件非空。
- 节点没有明显重叠。
- 箭头不穿过核心节点、不交叉。
- 字号、颜色、边框风格一致。
- 文件名和图意一致。
- 没有与 `3coding-visual` 的数据图重复。

发现问题要修 `.html` 并重新导出，不要只在报告里解释。

### Step 6: 写生成记录

创建 `reports/DIAGRAM_REPORT.md`，至少包含：

```markdown
# 非数据图示生成报告（HTML/SVG）

## 图示清单
| 文件 | 类型 | 来源依据 | 用途 | 状态 |
| --- | --- | --- | --- | --- |

## 未生成图示及原因

## 导出与自检记录

## 给论文阶段的嵌入建议
```

嵌入建议只说明每张图适合放入哪个章节和建议 caption，不生成 `*_latex_includes.tex`。最终的图表插入代码（LaTeX 的 `\begin{figure}...\end{figure}`）由 `5writing` 根据论文结构决定。

---

## 国奖级 HTML 流程图绘制规范（具体要求）

以下规范必须在每张图中逐条落实，缺一不可。

### 1. 画布与尺寸

- 用 `viewBox` 固定坐标系，按内容决定宽高比：横向路线图约 `900×640`，纵向流程图可 `760×900`，比例以「图面饱满、不留大片空白」为准。
- 所有坐标、字号都以 viewBox 坐标系为准，导出 PDF 后再由 LaTeX `\includegraphics[width=...]` 统一缩放，因此**图内不纠结物理字号**，只保证 viewBox 内比例协调。
- 字号层级：节点主文字 17–18，节点副说明 14，避免小于 13（缩放后看不清）。

### 2. 配色（低饱和、学术化，禁止纯黑白与霓虹）

统一使用下面这张低饱和学术配色表，全文图表保持一致：

| 用途 | 填充 fill | 描边 stroke | 说明 |
| --- | --- | --- | --- |
| 画布背景 | `#FAFBFD` | `#CFD6E4` | 极浅灰蓝，配清晰外框 |
| 普通步骤节点 | `#E8F0FA` | `#2F5597` | 淡蓝底 + 深蓝边 |
| 起止节点 | `#E3EFE6` | `#4A7C59` | 淡绿底 + 墨绿边 |
| 判断节点 | `#F7E8C8` | `#B8934A` | 淡金棕底 + 金棕边 |
| 强调/结论节点 | `#F3E8E0` | `#A65A4A` | 淡暗红底 + 暗红边 |
| 文字 | `#1F1F1F` | — | 深灰，不用纯黑 |
| 箭头/连线 | `#666666` | — | 中灰，不用纯黑 |

禁止：霓虹色、大面积高饱和色、强烈渐变、阴影滤镜（`filter`）、卡通装饰、花哨背景。

### 3. 节点形状

- 起止：胶囊（`rect` 且 `rx`=高度一半）或圆角矩形。
- 处理步骤：圆角矩形（`rect`，`rx=8`）。
- 判断：菱形（`polygon`，四顶点）。
- 数据/输入输出：平行四边形（`polygon`，斜边）。
- 同类节点样式严格统一（同一 fill、stroke、rx、字号、描边宽度）。

### 4. 文字规范

- **框内文字一律居中**：`text-anchor="middle"` 且 `dominant-baseline="central"`，`x`/`y` 取节点几何中心。
- 节点文字短，一般 ≤10 字；确需说明时双行，两行字数接近，禁止一行很长一行很短。
- 字体：中文用 `SimSun`（宋体，与论文正文一致），回退 `Microsoft YaHei`；图内文字语言与论文一致。
- 图内**不写大标题**，标题交给论文 caption（LaTeX `\caption{}`）。
- 图内不写大段解释，解释留给论文正文。

### 5. 箭头与连线

- 用 `<defs><marker>` 定义统一箭头（见示例），`marker-end="url(#arrow)"`。
- 连线走水平/垂直正交折线（`polyline`/`path` 的 L 型），避免斜线乱飞。
- 箭头方向清晰、指向单一，不穿过任何节点、不交叉；必要时绕行。

### 6. 布局与排版

- 主流程沿一条主轴（自上而下或自左而右）推进，分支清晰、层级分明。
- 同类元素左对齐或居中对齐，间距均匀。
- 每张示意图配清晰浅色外框与浅色背景（见配色表），背景不得有渐变或纹理。
- 图面要素紧凑，避免大片空白；也不要塞得过满导致文字与连线重叠。
- 成品在论文中应为矢量、可无损缩放；不得导出位图（PNG/JPG）冒充。

### 7. 禁止项汇总

- 禁止纯黑白风格；禁止阴影、渐变、霓虹色、科技炫彩。
- 禁止图内写大标题、长句、大段文字。
- 禁止箭头穿过节点或互相交叉。
- 禁止用 HTML 重复绘制 `3coding-visual` 已生成的数据图。
- 禁止导出位图；必须矢量 PDF。

---

## 可复用提示词模板

逐张生成图时，把 `【图名】`、`【文件名】`、`【内容依据】`、`【画布尺寸】` 替换后直接使用：

```text
请用 HTML（内嵌 SVG）绘制【图名】，输出到 figures/【文件名】.html 并导出 PDF，要求：

1. 内容：严格依据 ANALYSIS_MODELING_REPORT.md 中【内容依据】的真实方法/结构，不臆造步骤、不改写数值结论。
2. 画布：viewBox 设为【画布尺寸】，按内容决定横向或纵向，图面饱满、不留大片空白。
3. 配色：只用低饱和学术配色——画布背景 #FAFBFD / 外框 #CFD6E4，普通节点 #E8F0FA / #2F5597，
   起止节点 #E3EFE6 / #4A7C59，判断节点 #F7E8C8 / #B8934A，强调节点 #F3E8E0 / #A65A4A，
   文字 #1F1F1F，箭头 #666666。禁止阴影、渐变、霓虹色。
4. 节点：起止用胶囊，处理步骤用圆角矩形(rx=8)，判断用菱形，数据用平行四边形；同类节点样式严格统一。
5. 文字：全部 text-anchor="middle" + dominant-baseline="central"，x/y 取节点中心；
   字体 SimSun（回退 Microsoft YaHei）；节点文字 ≤10 字，必要说明双行且两行字数接近；
   字号 17–18（主）/14（副）；图内不写大标题，标题交给 caption。
6. 连线：defs 里定义统一箭头 marker，marker-end 指向；走水平/垂直正交折线，不穿过节点、不交叉。
7. 语言：图内文字与论文语言一致（中文）。
8. 完成自检：无重叠、无交叉、样式统一、文件名与图意一致，导出矢量 PDF 并确认非空。
```

## 质量要求

- 图示服务论文论证，不为装饰而画。
- 每张图必须能对应到 `reports/ANALYSIS_MODELING_REPORT.md` 中的真实方法。
- 数据型图表不得在本阶段重复生成。
- 论文阶段引用的非数据图都应有 `.html` 源文件和 PDF，或者在 `reports/DIAGRAM_REPORT.md` 说明导出失败。
