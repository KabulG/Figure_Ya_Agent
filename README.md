# FigureYa Agent

**从数据出发，找到适合的 FigureYa 绘图基座。**

FigureYa Agent 是面向科研绘图的 Codex 项目技能与辅助工具集。它以 [FigureYa](https://github.com/ying-ge/FigureYa) 的绘图模块为参考，帮助用户理解数据、比较模板、按需读取源码，并根据实际研究任务修改脚本和检查输出。

你可以直接描述绘图需求，无需提前记住模块编号或逐个尝试示例。

> “我有一份表达矩阵和样本分组表，希望同时展示组间差异和批次信息。请先分析数据，再比较合适的 FigureYa 模板，说明选择理由并完成绘图。”

[模板索引](figureya-agent/catalog/INDEX.md) · [使用与命令说明](figureya-agent/README.md) · [验证记录](figureya-agent/VALIDATION.md) · [上游 FigureYa](https://github.com/ying-ge/FigureYa)

## 为什么做这个项目

FigureYa 提供了丰富的科研绘图示例，但“找到一张相似的图”和“找到适合自己数据的脚本”仍有距离：

- 相同图形外观可能对应不同输入，例如表达热图、相关矩阵、CNV 状态和共享克隆计数。
- 某些模块包含完整分析流程；用户已有分析结果时，往往只需要其中的绘图部分。
- 示例中的列名、阈值、数据路径、分组方式和统计检验，需要根据新任务调整。

FigureYa Agent 将这些判断整理为可复用的技能和索引，让模板选择、代码修改和结果核对形成连续的工作流程。

## 核心能力

| 能力 | 如何帮助绘图 |
|---|---|
| 数据画像 | 检查表格字段、数值类型、缺失值、重复记录和可能的数据角色，为后续选型提供依据。 |
| 候选模板比较 | 结合表达目标、数据形态和研究设计筛选候选，并要求说明选择及排除理由。 |
| 模板差异辨析 | 区分同一图族中的不同用途，例如相关性与一致性、普通 ROC 与时间依赖 ROC。 |
| 按需获取源码 | 只取选定模块的具体文件，保留 commit、路径和哈希；无需完整下载上游仓库。 |
| 脚本适配 | 引导 Codex 调整字段映射、预处理、参数、配色、布局和导出方式，并审查示例副作用。 |
| 图件验收 | 引导检查数据对应关系、统计含义、运行结果、标签、图例和实际渲染质量。 |

其中，画像、检索、文件获取和静态审查由 Python 工具辅助完成；最终选型、脚本修改及视觉判断由 Codex 结合任务执行。命令行检索器本身不会自动完成任意数据的科研绘图。

## 工作方式

```text
用户数据与表达目标
        ↓
初步画像：字段、结构、缺失、可能的角色
        ↓
理解研究设计：观测单位、分组、配对、重复测量
        ↓
检索并比较候选模板，记录适用条件和排除理由
        ↓
按需读取选定源码，修改与当前任务有关的部分
        ↓
运行、核对数值、查看实际图件并修正问题
        ↓
交付图件、脚本、参数、来源和验证记录
```

选型时优先考虑数据语义和表达目的。模板是否简单、是否已经下载、是否曾经使用，都不能替代适配性判断。如果没有合适的现成基座，应说明缺口，再编写需要的绘图部分。

## 三个项目技能

| 技能 | 负责的工作 |
|---|---|
| [figureya-select](.agents/skills/figureya-select/SKILL.md) | 理解数据与研究设计，检索、比较和选择绘图基座。 |
| [figureya-adapt](.agents/skills/figureya-adapt/SKILL.md) | 按需读取和审查源码，适配字段、参数与绘图逻辑。 |
| [figureya-qa](.agents/skills/figureya-qa/SKILL.md) | 检查数据一致性、统计语义、实际输出和交付完整性。 |

项目入口 [AGENTS.md](AGENTS.md) 说明了如何使用这三个技能。不需要额外部署模型服务；使用者需要有可用的 Codex 环境。

## 快速开始

### 1. 准备项目与环境

将本 Agent 的 `AGENTS.md`、`.agents/` 和 `figureya-agent/` 放在同一项目目录，并在 Codex 中打开该目录。注意保留 `.agents` 隐藏目录；如果 Agent 位于一个大仓库的子目录中，应打开该子目录。

无需克隆完整的 FigureYa 上游仓库。最小使用范围是 Agent 文件、元数据索引及随后按需获取的源码。`outputs/synthetic_suite/` 是演示结果，可用于了解项目，不是核心工具的运行依赖。

环境要求：

- Python 3.10 或以上：画像、检索和按需获取工具。
- CSV、TSV、TXT 处理使用 Python 标准库；XLSX 读取额外需要 `openpyxl`。
- 实际绘图按所选脚本准备 R/Rscript 或 Python 及相应依赖。
- 获取未缓存的上游文件或刷新索引时需要网络；已有索引可在本地检索。

### 2. 提供数据和目标

在项目中发出这样的请求：

```text
请按本工程 AGENTS.md 使用 FigureYa Agent。

读取 data/ 下的数据，先检查结构和缺失情况。
我的目标是比较不同处理组的测量值分布，样本之间相互独立。
请比较合适的 FigureYa 基座，说明选择理由，然后适配脚本并绘图。
输出 PDF、PNG 预览、最终脚本和验证记录。
```

尽可能说明每行代表什么、字段的单位，以及是否存在配对、重复测量或删失。即使列名看起来熟悉，Agent 也不能仅凭名称确认这些含义。

### 3. 查看交付结果

真实绘图任务通常将产物保存在 `outputs/<任务名>/`，包括数据画像、候选比较、最终脚本、配置、图件、来源和验证说明。若环境或输入不足以完成出图，应明确说明阻塞项和未执行的步骤。

## 可以这样提问

| 任务 | 示例请求 |
|---|---|
| 差异表达结果 | “这是 log2FC 和 FDR 结果，请比较合适的火山图模板，并突出指定候选基因。” |
| 配对测量 | “这些数据来自同一批患者治疗前后，请保留配对关系并选择合适的展示方式。” |
| 方法一致性 | “同一批样本被两种方法测量，我想观察偏差和一致性范围。” |
| 样本结构 | “我有表达矩阵、分组和批次表，希望在 PCA 图中同时体现两层信息。” |
| 双指标矩阵 | “两份矩阵行列完全对应，请比较上下三角与热图叠加气泡的表达方式。” |
| 分类性能 | “这是二元结局和两个模型的预测分数，请比较 ROC，并说明阳性类与分数方向。” |

具体选择取决于数据，而不是固定将某一句提示词绑定到某个模块。

## 模板索引

当前索引快照整理了 **318 个模块目录**（含贡献模板目录）、**324 份 R Markdown 文件的元数据**，并设置了 **27 个检索分组**。

| 文件 | 用途 |
|---|---|
| [INDEX.md](figureya-agent/catalog/INDEX.md) | 可浏览的模块导航和固定版本源码链接。 |
| [modules.json](figureya-agent/catalog/modules.json) | 从远程源码提取的场景、输入、依赖和审查提示。 |
| [curated.json](figureya-agent/catalog/curated.json) | 对部分易混淆模块补充适用条件与辨析说明。 |
| [routing.json](figureya-agent/catalog/routing.json) | 检索分组、输入要求及注意事项。 |
| [upstream-tree.json](figureya-agent/catalog/upstream-tree.json) | 上游文件路径、大小和 Git blob 哈希，用于定位及校验。 |

索引依据上游 commit `f627917b79f28558779fb2e3ea2014cede41b89e` 构建。它是可更新的知识快照，不是实时仓库镜像。源码说明被自动提取，并不表示该脚本已经运行验证；检索分数也不是置信度或科研结论。

## 命令行工具

通常可以让 Codex 根据任务调用工具；下面的命令也可在项目根目录手动运行。先将示例路径替换为自己的数据文件。

```powershell
# 初步画像
python figureya-agent/scripts/figureya.py profile "data/input.csv" --out "outputs/my-task/profile.json"

# 根据目的和数据画像召回候选
python figureya-agent/scripts/figureya.py recommend --intent "两种测量方法的一致性" --profile "outputs/my-task/profile.json" --out "outputs/my-task/candidates.json"

# 检查候选的实际输入、依赖及文件路径
python figureya-agent/scripts/figureya.py inspect FigureYa176BlandAltman

# 完成选择后，获取明确的一份源码
python figureya-agent/scripts/figureya.py fetch FigureYa176BlandAltman --path FigureYa176BlandAltman/FigureYa176BlandAltman.Rmd
```

`profile` 默认读取前 10,000 行，并明确标记是否抽样；正式出图前仍需检查相关字段的全量数据。非标准字段名可用 `--roles` 指定映射，多工作表 XLSX 需用 `--sheet` 选择工作表。

`fetch` 默认单次预算为 5 MiB，缓存总量上限为 32 MiB，按固定 commit 校验文件，不自动执行下载的代码。更多参数见 [工具说明](figureya-agent/README.md)。

## 演示与验证范围

已记录的测试包括 23 项自动化测试，以及五类固定随机种子合成数据的候选检索和 Python 绘图演示。

| 合成任务 | 检索得到的参考模块 | 演示预览 |
|---|---|---|
| 差异表达火山图 | FigureYa59volcanoV2 | [PNG](outputs/synthetic_suite/01_volcano/figureya59volcano_adapted.png) |
| 测量方法一致性 | FigureYa176BlandAltman | [PNG](outputs/synthetic_suite/02_bland_altman/figureya176_bland_altman_adapted.png) |
| 分组与批次 PCA | FigureYa101PCA | [PNG](outputs/synthetic_suite/03_pca_batch/figureya101_pca_batch_adapted.png) |
| 多面板 ROC | FigureYa102multipanelROC | [PNG](outputs/synthetic_suite/04_multipanel_roc/figureya102_multipanel_roc_adapted.png) |
| 多层分类流向 | FigureYa25Sankey_update | [PNG](outputs/synthetic_suite/05_sankey/figureya25_sankey_adapted.png) |

测试期间仅按需获取了这五个模块的 Rmd，并进行了哈希校验。由于当时环境中没有可用的 Rscript，生成的图件采用 Python 演示实现，**没有执行上游 Rmd，也没有验证与原始 R 实现的等价性**。

这轮测试属于预设任务的回归与展示，不是独立盲测，也不能证明自动脚本移植、全部模板兼容性或所有图件的科研正确性。运行报告需要结合实际图件与代码审查；单凭脚本输出的 PASS 字样不足以完成验收。

以下为其中一张合成数据预览，不代表真实科研结果：

![基于合成配对测量的 Bland–Altman 演示图](outputs/synthetic_suite/02_bland_altman/figureya176_bland_altman_adapted.png)

运行自动化回归检查：

```powershell
python -m unittest discover -s figureya-agent/tests -v
```

查看 [验证记录](figureya-agent/VALIDATION.md) 和 [演示代码](figureya-agent/tests/run_demo_suite.py)。重新运行演示脚本会重新生成 `outputs/synthetic_suite/`，请勿将真实研究数据放入该演示目录。

## 项目结构

```text
FigureYa_agent/
├── README.md                  项目介绍
├── AGENTS.md                  Agent 工作入口
├── .agents/skills/
│   ├── figureya-select/       数据理解与模板选型
│   ├── figureya-adapt/        脚本适配
│   └── figureya-qa/           图件验收
├── figureya-agent/
│   ├── catalog/              模块索引与辨析卡片
│   ├── scripts/              画像、检索、获取及审查工具
│   ├── references/           数据、适配和验收参考
│   ├── tests/                自动化测试与合成演示
│   ├── cache/                按需获取的少量上游源码
│   ├── README.md             工具使用说明
│   └── VALIDATION.md         既有验证记录
└── outputs/                  绘图任务与演示产物
```

## 贡献与改进

欢迎补充有证据支持的模块辨析、真实失败用例、统计设计检查和 R 环境执行验证。提交问题时，请尽量提供脱敏的小型数据、表达目标、候选/所选模块、运行环境，以及预期结果与实际结果的差异。

新增检索规则应服务于数据适配性；新增模板支持应核对真实源码和输入要求。避免通过不断提高某个常用模块的权重，使所有任务重新集中到少数模板。

## 来源与致谢

本项目是围绕 FigureYa 构建的社区辅助工具。绘图模块及原始方法归其作者和贡献者所有，集成状态以各仓库实际记录为准。

- [FigureYa 上游仓库](https://github.com/ying-ge/FigureYa)
- [FigureYa 在线结果浏览](https://ying-ge.github.io/FigureYa/)
- [FigureYa 使用专题](https://medaibox.com/ai-skills/figureya.html)
- [AI 科研技能库](https://medaibox.com/ai-skills/)
- [本项目来源记录](figureya-agent/references/sources.md)

索引构建时，上游 README 声明使用 **CC BY-NC-SA 4.0**。复用具体模块时应保留相应来源、作者、许可和引用；这一上游许可说明不替代本项目原创代码的独立授权声明。

使用 FigureYa 相关代码开展研究时，请按上游要求引用：

Lu X, et al. (2025). *FigureYa: A Standardized Visualization Framework for Enhancing Biomedical Data Interpretation and Research Efficiency*. iMetaMed, 1: e70005. [DOI: 10.1002/imm3.70005](https://doi.org/10.1002/imm3.70005).
