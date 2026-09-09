---
name: 5writing
description: "数学建模竞赛论文撰写阶段（LaTeX / xelatex）。根据 ANALYSIS_MODELING_REPORT.md、RESULTS_REPORT.md 和 figures/*.pdf 选择比赛模板、组织章节，并在论文正文中按章节直接插入图表。"
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Agent, WebSearch, WebFetch
---

# 竞赛论文撰写（LaTeX）

本 skill 承接 `3coding-visual`、`4flow-diagram` 和 `7figure-lab`。前序阶段只提供真实数据、图表 PDF 和记录文件；本阶段负责选择比赛模板、组织论文结构，并决定每张图表放入哪个章节。

论文统一使用 LaTeX（xelatex 编译）。

## 数学建模规范参考

如需领域判断，读取 `../_references/math_modeling_norms.md` 中的“论文写作”“图表与可视化”和“非数据图工具选择”小节。该文件只作为规范知识库，论文结构仍按比赛模板和当前赛题内容决定。

## 模板族

本技能内捆绑的 LaTeX 模板位于：

```text
templates/zh/<竞赛>-latex/main.tex   # 中文 LaTeX 模板
templates/en/<竞赛>-latex/main.tex   # 英文 LaTeX 模板
```

支持的中文模板（`-latex` 后缀，xelatex 编译）：

```text
apmcm, changsanjiao, cumcm, default, diangongbei, dongsansheng,
huashubei, huaweibei, huazhongbei, mathorcup, mcm, shuweibei, stats, wuyibei
```

华为杯、华中杯、五一杯统一使用 `huaweibei`、`huazhongbei`、`wuyibei` 作为模板。

支持的英文模板（`-latex` 后缀）：

```text
apmcm, default, mcm
```

论文中的所有数值图表结论必须来自 `reports/RESULTS_REPORT.md` 或 `figures/*`。不得编造、估算或使用不同的四舍五入方式。

## 工作流

### 步骤 1：选择语言和模板

除非用户明确要求中文，否则 MCM/ICM/COMAP 一律使用英文。所有中文竞赛名称使用中文。

模板键示例：

```text
长三角 -> zh/changsanjiao-latex
APMCM 英文版 -> en/apmcm-latex
全国赛/国赛/CUMCM -> zh/cumcm-latex
统计建模 -> zh/stats-latex
MCM/ICM/COMAP -> en/mcm-latex
```

### 步骤 2：准备模板

用以下命令检查捆绑模板是否可访问（`SKILL_DIR` 为本 skill 所在目录）：

```bash
ls "$SKILL_DIR/templates/zh/<竞赛>-latex/main.tex" 2>/dev/null && echo "OK" || echo "MISSING"
```

- **文件存在（OK）**：将 `templates/zh/<竞赛>-latex/` 整目录复制到 `paper/`。
- **文件不存在（MISSING）**：说明 skill 未完整安装，此时依照本 SKILL.md 步骤 4 列出的对应节文件结构，从零重建最小可编译 LaTeX 框架，并在 `paper/` 内注明“重建自 default-latex 结构”。

存在匹配模板时，绝不从零开始写论文。

### 步骤 3：构建图表规划

在写正文各节之前，根据 `figures/*.pdf`、`reports/RESULTS_REPORT.md`、`reports/DIAGRAM_REPORT.md` 和 `reports/FIGURES_MENU.md`（如果存在）构建图表规划：

```text
图表规划
fig_roadmap.pdf -> 问题分析末尾（总体技术路线图）
fig_flow_q1.pdf -> 问题一模型构建
fig_flow_q2.pdf -> 问题二模型构建
fig_pipeline.pdf -> 数据预处理/方法节
结果图 -> 对应的结果节
```

图片路径相对于写入该图片的文件：写在 `paper/main.tex` 中通常用 `../figures/xxx.pdf`，写在 `paper/sections/*.tex` 中通常用 `../../figures/xxx.pdf`。

图片插入：

```latex
\begin{figure}[H]
  \centering
  \includegraphics[width=0.85\textwidth]{../../figures/fig_q1_error_dist.pdf}
  \caption{问题一预测误差分布}
  \label{fig:q1_error}
\end{figure}
```

英文论文使用英文图注。

### 步骤 4：撰写各节

章节文件统一使用 `.tex` 扩展名，文件名以 `main.tex` 中 `\input{}` 引用的为准。

国赛 LaTeX 模板（`zh/cumcm-latex`）：

```text
1_restatement.tex
2_analysis.tex
3_assumptions.tex
4_symbols.tex
5_problem1.tex
6_problem2.tex
7_problem3.tex
...          - 根据题目调整问题数量
8_sensitivity.tex
9_evaluation.tex
A_code.tex
```

MCM/ICM LaTeX 模板（`en/mcm-latex`）：

```text
1_introduction.tex
2_assumptions.tex
3_model_design.tex
4_solution.tex
5_sensitivity.tex
6_strengths_weaknesses.tex
7_conclusions.tex
A_code.tex
```

其余模板（`changsanjiao-latex`、`default-latex`、`huashubei-latex`、`mathorcup-latex`、`wuyibei-latex`、`huazhongbei-latex`、`huaweibei-latex`、`diangongbei-latex`、`dongsansheng-latex`、`shuweibei-latex`、`stats-latex`、`apmcm-latex`、`mcm-latex`、`en/apmcm-latex`、`en/default-latex`）的章节文件命名与上述结构类似，以 `main.tex` 中 `\input{}` 引用的文件名为准。

**正文写作应使用连贯的学术段落。避免在最终论文中出现工作流内部名称，如 `reports/`、`figures/` 或 `CLAUDE.md`。**

**模板 section 文件是起步示例，不是最终成品。** 正式写作时先在示例基础上展开，再按提示词规范补足数量与表述：模型假设 4–6 条且不得写“数据真实有效”之类的空泛假设、模型评价优点 4 条 / 缺点 1–2 条 / 推广 3 条。示例中少于该数量的内容必须补充到位，示例中与提示词冲突的表述必须纠正。

**模型建立与求解部分：公式引出必须逻辑连贯、自然过渡。** 禁止用一两句话直接引出公式（如“根据题意建立如下模型”后紧跟公式）；每个核心公式之前必须交代问题背景、变量定义和推导依据，公式之后解释符号含义与物理意义，保证读者能顺着文字读懂公式。公式之间用逻辑关系衔接，不得写成“一句话 → 公式 → 一句话 → 公式”的拼接。

**亮点要显式呈现，不能埋没。** 每个问题求解后若做出了亮点（独到洞察 / 巧妙耦合 / 深刻机理 / 有价值的延伸），必须在摘要、结果分析或延伸分析中用明确的语言点出，让评委一眼看到，而不是藏在正文里等评委自己挖。亮点必须来自真实计算，禁止编造。

### 步骤 5：参考文献

只使用真实存在的参考文献，文件名用 `paper/references.tex`。

```latex
\begin{thebibliography}{99}
  \bibitem{ref1} 作者. 题名[J]. 期刊名, 年份, 卷(期): 页码.
  \bibitem{ref2} Author. "Title." Journal, year.
\end{thebibliography}
```

正文引用用 `\cite{ref1}` 或 `\cite{ref1,ref2}`。

### 步骤 6：最后撰写摘要或总结

在所有章节完成后撰写中文摘要或英文 Summary Sheet。必须包含每个子问题的方法和精确的数值结果，并提炼全文最亮眼的一两个点（独到洞察 / 巧妙耦合 / 深刻机理 / 有价值的延伸）在摘要中突出。

## LaTeX 写作要点

### 编译命令

```bash
# 中文模板（xelatex，跑两遍解决交叉引用）
xelatex main.tex && xelatex main.tex

# 英文模板（xelatex，同样跑两遍）
xelatex main.tex && xelatex main.tex
```

### 文档结构

```latex
\documentclass[a4paper,12pt]{article}   % 英文
\documentclass[fontset=windows,12pt]{ctexart}   % 中文（Windows 字体）

\usepackage{...}   % 宏包加载
\usepackage{graphicx}   % 图片支持
\usepackage{booktabs}   % 三线表
\usepackage{amsmath,amssymb}   % 数学公式
\usepackage[colorlinks=true,linkcolor=black,citecolor=black,urlcolor=black]{hyperref}   % 交叉引用（需两遍编译）
```

### 图表插入

```latex
\begin{figure}[H]
  \centering
  \includegraphics[width=0.85\textwidth]{../../figures/fig_q1.pdf}
  \caption{图注}
  \label{fig:q1}
\end{figure}

% 三线表
\begin{table}[htbp]
  \centering
  \caption{表注}
  \begin{tabular}{ccc}
    \toprule
    \textbf{列1} & \textbf{列2} & \textbf{列3} \\
    \midrule
    数据 & 数据 & 数据 \\
    \bottomrule
  \end{tabular}
\end{table}
```

### 含公式的精确图（TikZ）

含数学公式/符号需精确标注的图（变量关系图、概率图模型、贝叶斯网络、带公式的模型结构图），不要用 HTML 画，直接用 TikZ 在正文中绘制——与正文同字体、公式可排版可交叉引用，观感远优于外置 SVG。

```latex
% 前置声明（放入导言区）
\usepackage{tikz}
\usetikzlibrary{positioning, arrows.meta, calc}

% 用法示例：Logistic 回归模型结构
\begin{figure}[H]
  \centering
  \begin{tikzpicture}[
      box/.style={draw, rounded corners, minimum width=2.4cm,
                  minimum height=0.9cm, align=center, font=\small}
    ]
    \node[box] (x) {特征\\ $x_1,\dots,x_p$};
    \node[box, right=of x] (z) {线性组合\\ $z=\beta^T x$};
    \node[box, right=of z] (p) {概率\\ $P(y=1\mid x)=\sigma(z)$};
    \draw[-{Stealth}] (x) -- (z);
    \draw[-{Stealth}] (z) -- (p);
  \end{tikzpicture}
  \caption{Logistic 回归模型结构}
  \label{fig:logistic_structure}
\end{figure}
```

TikZ 图不占 `figures/` 目录、不单独导出 PDF，直接在 `.tex` 里写；公式用 LaTeX 原生排版，交叉引用、字体与正文完全一致。

### 交叉引用

```latex
如图~\ref{fig:q1}所示，...   % 图片引用
式~(\ref{eq:objective}) 给出...   % 公式引用
见第~\pageref{fig:q1} 页   % 页码引用
```

### 数学公式

```latex
行内公式：$f(x) = \sum_{i=1}^n \theta_i \phi_i(x)$

行间公式：
\begin{equation}
  \mathcal{L}(\theta) = \frac{1}{N}\sum_{i=1}^N (y_i - \hat{y}_i)^2
  \label{eq:objective}
\end{equation}
```

### 章节和强调

```latex
\section{问题重述}
\subsection{问题背景}
\textbf{问题一：} xxx
```
