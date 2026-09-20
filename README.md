# DeepSeek Harness 插件部署与使用教程

> **本机实装版** · 逐包核对版本与工具清单
>
> 本文不是"可能包含……"的推测清单，而是对**当前这台机器上真正挂载运行的插件**逐一开箱核对后写成的教程：每个包都读了它的 `package.json`、`README` 和 `cordis.patch.yml`，工具名直接从注册代码里抓出来。

| 项目 | 值 |
| :--- | :--- |
| DSH 版本 | `@deepseek-ai/dsh` **0.1.1-rc.2** |
| Node | **v22.23.2**（DSH 要求 `^22.19` 或 `>= 24`） |
| 生效 profile | **`web`** → `~/.dsh/profiles/web/` |
| 插件数 | **19 个第三方 bundle**（+ 2 个核心 bundle） |
| 核对时间 | 2026-09-19 |



---

## 目录

1. [先搞懂 3 个概念](#1-先搞懂-3-个概念)
2. [命令速查：安装 / 查看 / 卸载](#2-命令速查安装-查看-卸载)
3. [插件总览（19 个）](#3-插件总览19-个)
4. [分类详解与使用方法](#4-分类详解与使用方法)
   - [A. 核心与工作台](#a-核心与工作台)
   - [B. 生态发现与市场](#b-生态发现与市场)
   - [C. 文档与办公](#c-文档与办公)
   - [D. 科研工作流](#d-科研工作流)
   - [E. 视觉、记忆与阅读](#e-视觉记忆与阅读)
   - [F. 写作与出版](#f-写作与出版)
   - [G. 界面、外观与绘图](#g-界面外观与绘图)
5. [前置条件速查](#5-前置条件速查)
6. [四套推荐组合](#6-四套推荐组合)
7. [排障 FAQ](#7-排障-faq)
8. [与参考仓库的差异](#8-与参考仓库的差异)
9. [参考链接](#9-参考链接)
10. [备份与迁移你的插件配置](#10-备份与迁移你的插件配置)

---

## 1. 先搞懂 3 个概念

DSH 的插件体系比"装个 npm 包"多一层，搞懂这三个词就不会踩坑：

| 概念 | 是什么 | 在本机的位置 |
| :--- | :--- | :--- |
| **Profile** | 一套独立的运行配置（装了哪些插件、用哪些模型）。可以同时存在多个，互不干扰 | `~/.dsh/profiles/web/`、`~/.dsh/profiles/headless/` |
| **Bundle** | 插件包通过 `cordis.patch.yml` 声明自己"插入"哪些引擎/工具/UI，装进 profile 后自动生效 | 各插件的 `cordis.patch.yml` |
| **Host / Client 双面** | Host 半边跑在 Node 进程（提供工具、REST API）；Client 半边跑在浏览器（提供面板、按钮、设置页） | 通常同一 npm 包含 `/lib/index.js`（host）与 `/lib/client.js`（client） |

**一句话**：`dsh plugin --profile web add <包名>` = 把 bundle 装进 `web` 这个 profile，重启后 Host 的工具和 Client 的面板一起上线。

> **重要**：装完插件**必须重启 `dsh web`**（或刷新页面）。DSH 不会热加载新装的插件。

---

## 2. 命令速查：安装 / 查看 / 卸载

### 2.1 安装（本文所有安装命令都基于此）

```bash
# 基本语法（--profile 必填，否则报错）
dsh plugin --profile web add <包名>

# 钉死版本（强烈建议，避免上游破坏性更新）
dsh plugin --profile web add dsh-zotero@0.5.1

# 从 GitHub 源码装（可钉 commit）
dsh plugin --profile web add github:Vncntvx/dsh-zotero
dsh plugin --profile web add github:didclawapp-ai/DSH-Office#<commit-sha>

# 从本地目录 / tarball 装（开发调试用）
dsh plugin --profile web add /path/to/dsh-sci-figure
cd my-plugin && npm pack && dsh plugin --profile web add ./my-plugin-*.tgz
```

> 小技巧：`dsh plugin --profile web add` 底层就是 `pnpm add`（工作目录自动切到 profile）。所以 `@version`、`github:`、本地路径、tarball 这些写法全部通用。

### 2.2 查看已装插件（验证安装）

```bash
dsh plugin --profile web list
```

输出（本机实际结果）：

```
dsh-profile-web /home/user/.dsh/profiles/web (PRIVATE)
│   dependencies:
├── @changfenhuang/dsh-genui@0.9.6
├── @feiyang666/deepseekharnessdesktop@1.9.0
├── @liustack/modlens@3.24.0
├── @vectorize-io/hindsight-coding-agents@0.4.3
├── dsh-deepread@1.0.0
├── dsh-doc@0.1.1
├── dsh-find-plugin@0.3.7
├── dsh-latex-tools@0.1.2
├── dsh-mineru@0.1.9
├── dsh-plugin-writing-guard@1.6.1
├── dsh-sci-figure@link:../../../ppt_skills/dsh-sci-figure
├── dsh-science@0.2.0
├── dsh-skin@0.4.1
├── dsh-tool-excalidraw@0.2.0
├── dsh-univer-office@0.2.10
├── dsh-workbench-plugin@0.1.31
├── dsh-zagens-office@0.1.0
├── dsh-zotero@0.5.1
└── dshmarket@1.18.0

19 packages
```

只要出现在这个列表里 = 已部署。

### 2.3 更新 / 卸载 / 启停

```bash
# 更新到最新（或指定版本）
dsh plugin --profile web add dsh-zotero@latest

# 更新全部
dsh plugin --profile web update

# 卸载
dsh plugin --profile web remove dsh-zotero

# 临时禁用（不删除）—— 在 ~/.dsh/profiles/web/cordis.patch.yml 对应行加 disabled: true
```

### 2.4 检查插件是否真的生效

```bash
# 看最终配置里有没有该插件的层
dsh --profile web --dump-config | grep -i "<插件名>"

# 看启动日志
# 多数 host 插件会打印 "[<plugin-id>] plugin loaded"
```

---

## 3. 插件总览（19 个）

| # | 包名 | 版本 | 分类 | 一句话 |
| :-: | :--- | :--- | :--- | :--- |
| 1 | `dsh-workbench-plugin` | 0.1.31 | 工作台 | 把 Web UI 变成三栏 IDE：编辑器 + 智能终端 + 文件 / Git / 用量 |
| 2 | `dshmarket` | 1.18.0 | 生态 | 可视化插件市场，Settings → Plugin Market 一键装 |
| 3 | `dsh-find-plugin` | 0.3.7 | 生态 | 让 agent 自己去 GitHub 搜插件（`find_dsh_plugin`） |
| 4 | `@changfenhuang/dsh-genui` | 0.9.6 | 交互 | 把回复渲染成可交互 UI（表格 / 图表 / 表单 / 测验 / Mermaid / 3D） |
| 5 | `dsh-doc` | 0.1.1 | 文档 | **纯本地** PDF / Office / 图片 / OCR → Markdown、text、JSON |
| 6 | `dsh-mineru` | 0.1.9 | 文档 | MinerU 云端多模态解析，全格式 → 结构化 Markdown |
| 7 | `dsh-deepread` | 1.0.0 | 阅读 | 精读文章 / 书 / PDF，输出可溯源观点、证据、置信度、知识地图 |
| 8 | `dsh-zagens-office` | 0.1.0 | 办公 | 调本机 zagens-office CLI 生成 / 编辑 PPTX / DOCX / XLSX / PDF |
| 9 | `dsh-univer-office` | 0.2.10 | 办公 | Univer 表格 / 文档 / 幻灯片 / 多维表 / 画布，带协作 Gateway 与审阅卡片 |
| 10 | `dsh-science` | 0.2.0 | 科研 | ReAct 科研循环 + 版本化工件溯源 + SSH/HPC 远程计算（31 个工具） |
| 11 | `dsh-sci-figure` | 0.1.0（本地 link） | 科研制图 | 出版级图表渲染 + QC + .pptx 生成 / 审计 |
| 12 | `dsh-zotero` | 0.5.1 | 文献 | Zotero 库检索、笔记批注、按问题提取证据、生成引用 |
| 13 | `dsh-plugin-writing-guard` | 1.6.1 | 写作 | 论文写作守卫：AI 味 / 修改残留 / 主张漂移 + 期刊契合度 |
| 14 | `@liustack/modlens` | 3.24.0 | 视觉 | 给纯文本模型"眼睛"，图片直接粘贴即可读 |
| 15 | `@vectorize-io/hindsight-coding-agents` | 0.4.3 | 记忆 | 项目长期记忆 + 知识页，跨会话可查 |
| 16 | `dsh-skin` | 0.4.1 | 外观 | 15 套皮肤、亮 / 暗切换、字号缩放、13 个配色角色 |
| 17 | `@feiyang666/deepseekharnessdesktop` | 1.9.0 | 监控 | token 用量与消耗统计、峰谷计费、余额查询、CSV/JSON/PNG 导出 |
| 18 | `dsh-latex-tools` | 0.1.2 | 界面 | 悬停 LaTeX 公式即可复制 TeX 或导出独立 SVG |
| 19 | `dsh-tool-excalidraw` | 0.2.0 | 绘图 | agent 直接创建 / 编辑 Excalidraw 白板，导出 SVG |

**另外两点**：

- `dsh-model-tier@0.1.0`（模型分档路由）作为 `dsh-science` 的依赖被自动带上，并在 `dsh-science` 的 patch 里挂载（虚拟 provider「智能分档」）。
- `dsh-pdf-edit@0.2.1` 存在于 `node_modules`，但**不在** profile 的 dependencies / bundles 里，等同于未挂载（见 [排障](#7-排障-faq)）。

---

## 4. 分类详解与使用方法

### A. 核心与工作台

#### 核心 bundle（DSH 自带，无需安装）

| 包 | 版本 | 作用 |
| :--- | :--- | :--- |
| `@deepseek-ai/dsh-base` | 0.1.1-rc.2 | Agent 循环、工具系统、会话、沙箱、权限等全部基础能力 |
| `@deepseek-ai/dsh-web-app` | 0.1.1-rc.2 | Web UI（`dsh web` 打开的那个界面） |

核心自带的工具（不用装任何插件就能用）：`bash`、`str_replace_editor`、`git_*`、`todo`、`subagent`、`web_search`、`skill` 等。

```bash
dsh web                 # 启动 Web UI（默认 http://127.0.0.1:3080）
dsh                     # 交互式 TUI
```

---

#### 1. `dsh-workbench-plugin` — 三栏工作台 ⭐

**定位**：本机最"能打"的插件。打开后 Conversation 留在左侧，右侧加两栏：中间是**编辑器 + 智能终端**，最右是**文件 / Git / 用量 / Ultra Slash**。

**核心能力**：

| 模块 | 说明 |
| :--- | :--- |
| **Smart terminal** | 本地 PTY。真命令直接执行；**自然语言**按 `Alt+I` 由 AI 翻译成命令后填入当前终端（不会自动执行）；危险命令黑名单拦截 |
| **Agent Control Plane** | 执行轨迹鱼骨图（用户 → LLM → 工具 → 回复，可展开 I/O）+ 当前会话能力清单 |
| **Workspace editor** | CodeMirror 6，支持 CSS/HTML/JS/JSON/Markdown/Python/XML/YAML 高亮；Markdown 与 Canvas 预览 / 分栏 |
| **Files** | 文件树、过滤、隐藏文件、`.gitignore` 标记、新建 / 重命名 / 删除、外部编辑器打开 |
| **Git** | status / stage / commit（AI 生成 message）/ fetch / pull / push / branch / merge / 提交图 |
| **Usage** | 官方 API 余额 + 本机观测消耗 + 本会话 token / 上下文 |
| **Ultra Slash** | 不打断当前轮次的斜杠指令（`/steer`、`/new`、`/skill`、`/docs`、`/canvas`） |
| **Canvas** | `.canvas/*.canvas.tsx` 实时 React 预览 |
| **Notification sounds** | 会话结束 / 等待审批时播放提示音，可循环提醒 |

**安装**：

```bash
dsh plugin --profile web add dsh-workbench-plugin@0.1.31
```

**使用**：重启 `dsh web` → 打开 `http://127.0.0.1:3080` → 进入 Conversation → 顶栏点 **Workbench**。

**示例**：

```text
让我把 chat 里的报错修一下        → 智能终端翻译成命令
/canvas 一个数据看板原型          → 打开 Canvas 预览
在终端里输入：帮我装 pandas       → AI 转成 pip install pandas 填入终端
```

---

### B. 生态发现与市场

#### 2. `dshmarket` — 可视化插件市场

**定位**：DSH 内置的"应用商店"，浏览 / 搜索 / 一键安装社区插件（1500+），还带主题、备份恢复、更新检查、热禁用。

**安装**：

```bash
dsh plugin --profile web add dshmarket@1.18.0
```

**使用**：重启 `dsh web` → **Settings → Plugin Market**。

> ⚠️ 要求宿主 `dsh web ≥ 0.1.0-rc.6`。若菜单不出现，多半是宿主版本太旧（插件会在浏览器 console 里说明原因）。

**功能**：分类筛选 / 星标排序 / 双语描述 / 截图轮播 / 主题标签页 / 一键安装 / 备份到 WebDAV 或 Gist / 更新检查 / 热启用禁用（写 `disabled: true` 进 patch，HMR ~1s 生效）。

---

#### 3. `dsh-find-plugin` — 让 agent 自己找插件

**定位**：`/find-skills` 的 DSH 版。你用自然语言描述需求，agent 调用 `find_dsh_plugin` 去 GitHub 的 `dsh-plugin` topic 里按星标搜，返回描述 + 可直接运行的安装命令。

**安装**：

```bash
dsh plugin --profile web add dsh-find-plugin@0.3.7
```

**使用**：重启后直接说话即可，agent 会自己调用 `find_dsh_plugin`。

**工具**：`find_dsh_plugin`

**示例**：

```text
有什么终端 TUI 插件？
任务跑完想微信通知我，有插件吗？
帮我找个能在 DSH 里 review git diff 的插件
```

> 搜索结果自带 `dsh plugin add` 命令，可以直接让 agent 帮你装。第三方代码，建议先看源码并钉 commit。

---

#### 4. `@changfenhuang/dsh-genui` — 把回复变成交互界面

**定位**：让模型的回答不止是文字——在回复里内联渲染**可交互 UI**（排序表格、本地视频、可拖拽图表、测验、Mermaid、3D 场景、持久面板），不替换周围的文字。

**安装**：

```bash
dsh plugin --profile web add @changfenhuang/dsh-genui@0.9.6
```

**使用**：agent 通过 `dsh-ui` 代码围栏（三道反引号 + 语言标记 `dsh-ui`）输出 GenUI 规格；模型侧工具为 `render_ui`，写之前用 `validate_dsh_ui` 校验 JSON。

**工具**：`render_ui`、`validate_dsh_ui`（附 `genui` skill，教模型写围栏语法）

---

### C. 文档与办公

#### 5. `dsh-doc` — 纯本地文档智能

**定位**：把 PDF / Word / Excel / PowerPoint / Markdown / HTML / CSV / 扫描件交给 agent，拿回干净 Markdown、纯文本或结构化 JSON。**全程离线**，无 Docker、无 HTTP 服务、无 API Key，文件不出本机。

**安装**：

```bash
dsh plugin --profile web add dsh-doc@0.1.1
```

**使用**（4 个工具）：

| 工具 | 用途 |
| :--- | :--- |
| `dshdoc_health` | 检查本地解析引擎是否就绪 |
| `dshdoc_extract` | **首选**：本地文件 → md / text / json |
| `dshdoc_convert_file` | 同上，兼容入口；支持 `ocr`、`page_range`、`table_mode` |
| `dshdoc_convert_url` | 兼容占位（本地引擎不接受 URL，请先下载到本地） |

**常用参数**：`output_format`（md/text/json）、`ocr`、`ocr_languages`（如 `["chi_sim","eng"]`）、`table_mode`（fast/accurate）、`page_range`。

**示例**：

```text
把 reports/2024.pdf 转成 Markdown，保留表格
读一下 scan.png，用中英双语 OCR
```

> 权限：session 工作区自动可读；工作区外的目录要加进 `cordis.patch.yml` 的 `allowedLocalRoots`。

---

#### 6. `dsh-mineru` — MinerU 多模态解析

**定位**：接入 MinerU（OpenDataLab 高精度文档解析引擎），把 PDF / Word / PPT / Excel / HTML / 图片转成结构化 Markdown，表格、公式、图片都保留。适合"纯文本模型读文档"。

**两种 API 模式（自动选）**：

| 模式 | 条件 | 支持 |
| :--- | :--- | :--- |
| 🎯 Precision API | 配了 MinerU token | 全格式（含 `.doc`/`.ppt`/`.xls`、HTML） |
| ⚡ Agent API | 不填 token | PDF、`.docx`、`.pptx`、`.xlsx`、图片（IP 限流） |

**安装**：

```bash
dsh plugin --profile web add dsh-mineru@0.1.9
```

**使用（渐进式工具曝光）**：只有 `mineru_activate` 一直可见，调用一次后解锁全套：

| 工具 | 用途 |
| :--- | :--- |
| `mineru_activate` | 引导工具，调用一次解锁其余 |
| `mineru_parse` | 解析单个文件 / URL |
| `mineru_batch_parse` | 批量解析（自动分块，respect 官方限额） |
| `mineru_task` | 查询任务状态 |

**示例**：

```text
解析 papers/ 目录下全部 PDF，每篇输出 Markdown
把这 30 个合同 PDF 批量解析，然后提取甲方乙方和金额
```

---

#### 7. `dsh-zagens-office` — Office 文档生成 / 编辑

**定位**：用本机 `zagens-office` CLI 生成和编辑 PPTX / DOCX / XLSX / PDF。没有引擎时模型会自行下载到 `~/.zagens-pro/bin`。

**安装**：

```bash
dsh plugin --profile web add github:didclawapp-ai/DSH-Office
# 钉版本：dsh plugin --profile web add github:didclawapp-ai/DSH-Office#<commit-sha>
```

**使用（4 个工具）**：

| 工具 | 作用 |
| :--- | :--- |
| `office_schema` | 取 JSON 契约；**写 / 改之前先调** |
| `office_write` | 新建 pptx / xlsx / docx / pdf |
| `office_edit` | 按 op 修改已有文档 |
| `office_read` | 读回文档核对 |

**示例**：

```text
做一份 8 页的季度复盘 PPT
把这个 Excel 的表头加粗、冻结首行
```

> 启动日志出现 `[zagens-office] plugin loaded` 即成功。卸载：`dsh plugin --profile web remove dsh-zagens-office`。

---

#### 8. `dsh-univer-office` — Univer 办公套件

**定位**：给 DSH 一整套 Univer 办公环境：表格、文档、幻灯片、画布、关系表，带数据联动、校验、版本化变更和**多 agent 隔离 worktree**。每个改动都经过校验，留在对话里让你预览 / 批准 / 丢弃。

**安装**：先 `Ctrl+C` 停掉 DSH，再装，然后重启：

```bash
dsh plugin --profile web add dsh-univer-office@0.2.10
```

**使用**：自然语言描述想要的结果，agent 创建 / 编辑 / 校验，你在对话里实时看着并审阅。需要交付标准文件时说一句"导出为 `.xlsx` / `.docx` / `.pptx`"。

**工具（14 个）**：`univer_new`、`univer_status`、`univer_unit`、`univer_worktree`、`univer_import`、`univer_export`、`univer_execute`、`univer_inspect`、`univer_lint`、`univer_screenshot`、`univer_api`、`univer_resources`、`univer_compile_svg`

**示例**：

```text
做一个班级成绩表，加条件格式和数据图
帮我把这个 pptx 导入，改完第 3 页再导出
```

> 卸载：`dsh plugin --profile web remove dsh-univer-office`

---

### D. 科研工作流

#### 9. `dsh-science` — Claude Science 式科研工作台 ⭐

**定位**：面向 genomics / pathogens / bioinformatics 的研究工作台，三大引擎 + 11 个 science skills。

**三大引擎（31 个工具）**：

| 引擎 | 工具 | 作用 |
| :--- | :--- | :--- |
| **ReAct 研究循环** | `research_init`、`research_state`、`research_hypothesis`、`research_experiment`、`research_findings`、`research_phase`、`research_review`、`research_report` | 问题 → 假设 → 实验 → 观察 → 分析 → 结论 → 下一问，状态存在 `research-manifest.json` |
| **版本化工件溯源** | `artifact_save`、`artifact_list`、`artifact_show`、`artifact_diff`、`artifact_verify`、`artifact_deprecate`、`artifact_reproduce` | 每个结果存 `artifacts/<name>/v<N>/`，逐文件 SHA-256 + `artifact.json` + `provenance.md` |
| **远程计算（SSH / HPC）** | `remote_host_add`、`remote_host_probe`、`remote_host_notes`、`remote_host_list`、`remote_host_show`、`remote_host_remove`、`remote_host_allow`、`remote_host_revoke`、`remote_host_allowlist`、`remote_run`、`remote_status`、`remote_logs`、`remote_pull`、`remote_cancel`、`remote_exec`、`remote_jobs` | 用 `~/.ssh/config` 别名连实验室工作站 / HPC；workstation 用 `nohup+setsid`，SLURM 用 `sbatch`，断连不死 |

**附带的 11 个 skills**：research-loop、science-project-setup、artifact-provenance、scientific-reviewer、literature-connector、parallel-delegation、manuscript-writing、bioinformatics-toolkit、conda-environments、data-inventory、remote-compute。

**附带模型分档路由**：`dsh-model-tier`（也可单独装：`dsh plugin add dsh-model-tier`），把辅助请求（会话标题、压缩）和子 agent 路由到轻量档，复杂任务升到强档，每档可指向**不同 provider**。

**安装**：

```bash
dsh plugin --profile web add dsh-science@0.2.0
# 或从 GitHub：dsh plugin --profile web add "github:biociao/dsh-science"
```

**首次使用（典型闭环）**：

```text
research_init → 建 research-manifest.json + 项目骨架
research_hypothesis → H1 ...
research_experiment → E01 ...（自动建 experiments/E01/）
research_findings → 追加 log.md、更新假设状态、推进循环
artifact_save → 把值得引用 / 复现的结果存档
remote_host_add → remote_run → remote_status → remote_pull → artifact_save（需要算力时）
```

**安全护栏**：首次使用某主机需审批，通过后写入**本项目**白名单（`.dsh/remotes/allowlist.json`）；同工作区多个研究项目各自独立，授权不跨项目泄漏。无人值守可设 `requireHostAccess: false`。

---

#### 10. `dsh-sci-figure` — 出版级图表与 PPT 控制

**定位**：把"大概意思和方向"变成投稿级图表 + 摆好位置的 PPT。渲染、单位换算、QC、`.pptx` 读写全部跑在内置 Python 引擎里——确定性、离线、哈希溯源。

**流水线**：

```
intent ──► fig_plot ──► fig_qc ──► fig_layout ──► deck_build ──► deck_audit
           渲染         QC        px/mm/EMU 映射   写 PPT        只读审计
```

**7 个工具**：

| 工具 | 作用 |
| :--- | :--- |
| `fig_env` | 探测 / 修复运行时（`probe`、`setup tier=deck\|pdf-qc\|full`、`describe`） |
| `fig_plot` | 按 mm 精确尺寸渲染 Nature/Cell/Science 风格图（声明式 spec 或 matplotlib 脚本） |
| `fig_qc` | 确定性 QC：尺寸、dpi、PDF 里真实字号、字体嵌入、留白、色盲友好性 |
| `fig_layout` | 像素空间映射：`measure` `fit` `grid` `plan` `required_px` `anchor` `convert` |
| `deck_build` | 建 / 追加 .pptx，精确 EMU 定位，**永不拉伸**，原子写 + 备份 |
| `deck_audit` | 只读审计：形变、有效 dpi、溢出、字号下限 |

**安装**（本机是本地 link 版）：

```bash
dsh plugin --profile web add /path/to/dsh-sci-figure
# 本机实际指向：/home/user/ppt_skills/dsh-sci-figure
```

**首次使用**：

```text
fig_env op=probe          # 探测 Python / matplotlib / python-pptx / PyMuPDF
fig_env op=setup          # 缺啥补啥（创建 ~/.dsh/sci-figure/venv，继承系统科学栈）
```

**示例**：

```text
画一张 Nature 单栏（89mm）的折线图，含误差棒和显著性标记
把 fig1.pdf 放进 16:9 幻灯片，配 caption，再整体审计一遍
```

> 无 npm 运行时依赖、无需构建。Python 能力按调用时发现。

---

#### 11. `dsh-zotero` — 把 Zotero 变成 agent 的证据库

**定位**：agent 直接从你本地 Zotero 库搜文献、看元数据和笔记、**按问题提取证据段落**、打开原文 PDF、生成引用和参考文献表。

**前置条件**：

- Zotero ≥ 7 桌面版，**设置 → 高级 → 允许其他应用程序与 Zotero 通信**
- 本地 API：`http://127.0.0.1:23119/api`（无认证，只读）
- Node ≥ 22.19，宿主 dsh 0.1.1-rc.2 系列

**安装**：

```bash
dsh plugin --profile web add dsh-zotero@0.5.1
# GitHub：dsh plugin --profile web add github:Vncntvx/dsh-zotero
```

**8 个工具**：

| 工具 | 用途 |
| :--- | :--- |
| `zotero_search` | 按标题 / 作者 / 年份搜（library / collection / savedSearch / publications 作用域），`everything` 模式连全文索引一起搜 |
| `zotero_browse` | 发现库结构：库、合集树、保存的检索、标签 facet、条目类型及字段 |
| `zotero_get` | 读单条文献元数据，可选返回笔记 / 批注 / 附件；`fields:"all"` 保留全部 |
| `zotero_children` | 探索子对象图：笔记、附件、挂在 PDF 下的批注 |
| `zotero_retrieve` | 按查询词返回最相关证据段落（批注 / 笔记 / 摘要 / 全文），支持多附件 |
| `zotero_changes` | 基于本地版本的增量感知：哪些条目 / 合集 / 全文索引变了、什么被删了 |
| `zotero_attachment` | 把文献 ref 解析为已验证的磁盘路径或链接 URL |
| `zotero_export` | 生成引用、参考文献表、BibTeX / BibLaTeX / RIS / CSL JSON |

**使用**：装完重启新建会话即可。**Settings → Plugins** 里有配置卡片（API 地址、并发、全文检索开关等），保存即生效。

**示例**：

```text
帮我找 Risk 相关的期刊论文
把这条文献里支撑"样本量不足"的证据段落抽出来
给这 5 篇生成 BibTeX
```

> PDF 全文检索**只读**，验证后再引用。

---

### E. 视觉、记忆与阅读

#### 12. `@liustack/modlens` — 给纯文本模型装眼睛

**定位**：DeepSeek / GLM 的旗舰对话模型是纯文本的，读不了图。ModLens 是插件式视觉引擎，**图片直接粘贴进聊天就能读**，不用先存成文件再传路径。

**两种粘贴方式**：

1. **直接粘贴** —— 图片落成私有临时文件，路径进入输入框，`modlens_read_image` 接手（与 OpenCode / Pi 一致）。
2. **选 `(modlens vision)` 模型条目** —— 在模型选择器里选一次（会记住），之后粘贴的缩略图留在消息里，接近 Codex app 的体验；请求时转成结构化证据。

插件会自动发现承载纯文本 DeepSeek / GLM 模型的 provider 路由并加包装条目（标准安装会有 `DeepSeek-V4-Flash (modlens vision)` 和 `DeepSeek-V4-Pro (modlens vision)`）。只有元数据**明确**为纯文本的模型才会被接管；未确认的一律不碰，所以视觉模型保留原生粘贴。

**安装**：

```bash
dsh plugin --profile web add @liustack/modlens@3.24.0
# 官方一行命令：npx -y @deepseek-ai/dsh plugin --profile web add @liustack/modlens@3.24.0
```

**工具**：`modlens_read_image`（路径或 URL → 全文字转写 + 阅读顺序版式区域 + 语义 + 不确定性列表）

**示例**：

```text
（粘贴截图）这个报错怎么修？
（粘贴图表）把这张图的数据点读出来
```

> 需要配置 modlens 引擎；终端里跑 `npx @liustack/modlens doctor` 检查。

---

#### 13. `@vectorize-io/hindsight-coding-agents` — 项目长期记忆

**定位**：为 coding agent 提供长期项目记忆，由 [Hindsight](https://vectorize.io/hindsight) 支撑。一个包支持多个 harness（opencode、Kilo、Cline、**DSH**、Claude Code、Codex、Cursor 等）。**摄取完全自动**，没有 setup 命令——仓库的 git 历史和对话会在后台流入记忆库。

**核心洞见**：真实的修复大部分能从代码推导，但**最后一公里**常常取决于项目特有的决定（取整规则、重试白名单、平局策略），这些不在代码里，而在 git 历史和过去的对话里。这个包把它们在该动手时放到 agent 面前，并维护一组**知识页**（架构、约定、进行中的 initiative），让未来的会话从那里起步。

**安装**（注意：这是**独立 CLI**，不是 `dsh plugin add`）：

```bash
npx @vectorize-io/hindsight-coding-agents install all          # 所有检测到的 agent
npx @vectorize-io/hindsight-coding-agents install deepseek-harness   # 只装 DSH
npx @vectorize-io/hindsight-coding-agents uninstall all        # 精确移除它加的东西
```

安装时可选择记忆存放位置：Hindsight Cloud / 自建服务器 / 本机 local daemon；脚本化安装用 `--server cloud|self-hosted|daemon`。**更新就是再跑一次 `install`**。

**本机工具（9 个）**：`hindsight_search_knowledge_pages`、`hindsight_list_knowledge_pages`、`hindsight_read_knowledge_page`、`hindsight_reflect`、`hindsight_capture_initiative`、`hindsight_ingest_document`、`hindsight_diagnose`、`hindsight_sync_status`

**使用要点**：

- 做非平凡任务**前**先 `hindsight_search_knowledge_pages` / `list_knowledge_pages`，别重复造轮子
- 需要"为什么这么设计"的根因时用 `hindsight_reflect`（较慢）
- 用户批准新 feature 后、写代码**前**用 `hindsight_capture_initiative` 登记
- 发现记忆过时 → `hindsight_ingest_document` 写一条 `Correction: <topic>`

> ⚠️ 本机当前**未配置 API key**，调用会返回 401。需要在 Hindsight 侧配好 token 才能用。

---

#### 14. `dsh-deepread` — 证据优先的精读

**定位**：把长文章、书、PDF、文档集变成**可溯源的主张、证据、置信度、知识地图和复习问题**。

**安装**：

```bash
dsh plugin --profile web add dsh-deepread@1.0.0
```

**使用**：重启 `dsh web`，用 📖 阅读面板，或在对话里调 `deepread` 工具。也有可移植 Agent Skill 形态（Codex / Claude Code）：`npx skills@latest add xiehuan123/dsh-deepread`。

**五种模式**：

| 模式 | 适合 | 关键输出 | 成本 |
| :--- | :--- | :--- | :--- |
| `quick` | 扫一眼这篇讲了啥 | 一句话摘要、核心主张、论证结构、引用、关键概念、批判性问题 | 单次调用，最快 |
| `deep`（默认） | 仔细读一篇 | 概述、核心主张、论证结构（主张 + 证据 + 原文引用）、论证流、分节要点 | 长文自动分段 |
| `map` | 研究、引用前的事实核查 | 核心问题 / 结论、十类内容、主张—证据配对、五要素数据表、八种关系、**四档置信度**、Mermaid 思维导图、XMind 大纲、5 个主动回忆问题 | 结构化多轮 |
| `feynman` | 真学会并讲给别人 | 11 步闭环（目录 → 提问 → 分章 → 观点数据证据 → 章节导图 → 合书讲解 → 找缺口 → 回原文修正 → 合并导图 → 再讲一次 → 1/3/7/14/30 天间隔复习） | 输出最长 |
| `book` | 整本书 / 超长文本 | 目录、章节脉络、分部分精读后汇总 | 分部分处理 |

**输入**：微信公众号链接（`mp.weixin.qq.com`）、文件（`.txt/.md/.html/.pdf`）、粘贴文本；也支持 `batch` 传 2–10 篇做跨篇对比。

**导出**：默认只在会话展示；`export` 可选 `md` / `mm`（FreeMind，XMind 可导入）/ `html` / `all`，写到工作区 `deepread-output/`。

**成本预览**：`estimate: true` 不调模型，只预估 token、调用次数和耗时。

**示例**：

```text
精读 docs/architecture.pdf，用知识地图模式
把这两篇文章做个跨篇对比，重点看冲突点
先 estimate 一下这篇 PDF 的预算
```

> 长文 / 大 PDF / 批量会自动转后台任务，可用 `job_output` 轮询进度（"精读第 3/20 段…"、"解析 PDF 中… 42%"），`job_kill` 取消。

---

### F. 写作与出版

#### 15. `dsh-plugin-writing-guard` — 论文写作守卫

**定位**：不是"写完再大规模 Humanize"，而是 **写作前给规则 → 写作中自动守卫 → 修改后自动审计**。所有规则是本地正则 / 统计，**零网络、零 LLM、毫秒级**。

**双锁机制**：

- **Scholarship Lock**：保护数字、百分数、p 值、CI、引用、图表编号、DOI 不被语言润色悄悄改动
- **Epistemic Lock**：主张强度漂移（`associated` → `caused`）、否定 / 零结果标记翻转、scope 边界消失
- **证据状态守恒**：`reported/observed/measured/estimated/simulated` 消失或被替换时报警（"participants reported improvement" 不能变成 "participants improved"）

**4 个工具**：

| 工具 | 用途 |
| :--- | :--- |
| `writing_rules` | 写作前加载学术写作纪律速查清单 |
| `writing_audit` | 检查 AI-style patterns、修改残留、防御性表达、LLM 高频表达、篇章节奏；传 `original` 开双锁对比，传 `styleProfile` 查风格漂移，传 `journalProfile` 查期刊契合度 |
| `writing_style_profile` | 从作者历史论文统计句长 / 段长"节奏指纹"（零 LLM） |
| `writing_journal_profile` | 从目标期刊代表论文蒸馏写作分布（零 LLM） |

**安装**：

```bash
dsh plugin --profile web add dsh-plugin-writing-guard@1.6.1
```

**使用**：直接调工具；写 `.md` / `.tex` / `.txt` 论文文件时会**自动审计**（v0.8 起自动捕获修改前文本，直接跑双锁，无需手动传 `original`）。

**示例**：

```text
写作前：调 writing_rules
写完：writing_audit(filePath="manuscript/intro.md", profile="manuscript",
                   original=<修改前文本>, styleProfile=<作者风格 JSON>)
目标期刊契合：先 writing_journal_profile(learnDir="refs/ieee/") 再传进 writing_audit
```

---

### G. 界面、外观与绘图

#### 16. `dsh-skin` — 界面外观自定义

**定位**：设置 → 通用 →「个性化外观」展开行。

- **显示模式**：亮 / 暗 / 跟随系统
- **15 套预设皮肤**：海盐白、石墨灰、莓果红、珊瑚红、樱花粉、暖阳橙、摩卡棕、奶油米、柠檬黄、薄荷绿、森林绿、极光青、天际青、海盐蓝、薰衣草紫
- **字号**：小 / 中 / 大 / 特大（覆盖官方硬编码界面文字）
- **自定义样式**：13 个颜色角色（背景 / 卡片 / 浮层 / 侧栏 / 文字 / 边框 / 主题色 / 错误 / 成功 / 警告）+ 取色器 / HEX 输入；已保存的微调不被预设覆盖
- **恢复默认**：一键还原
- **中英双语**：跟随 DSH 语言设置

**安装**：

```bash
dsh plugin --profile web add dsh-skin@0.4.1
```

**使用**：重启 → **Settings → 通用 → 个性化外观**。持久化在 `localStorage`，重启自动恢复。

> ⚠️ 字号 / 控件联动依赖当前 DSH 构建的**类名哈希**（`.uV2eYG_*` 等），DSH 升级后这些覆盖可能静默失效（颜色令牌覆盖不受影响）。

---

#### 17. `@feiyang666/deepseekharnessdesktop` — 用量与消耗统计

**定位**：记录每次模型调用的 token 用量与消耗。WebUI 顶部「对话」「轨迹」之后会多出 **「用量与消耗」** 和 **「剩余余额查询」** 两个 tab。

**能力**：

- **用量与消耗**：输入（未命中 / 缓存命中 / 缓存写入）/ 输出 / 推理 / 结束原因；按 DeepSeek **峰谷计费**（北京时间 9:00–12:00、14:00–18:00 为高峰）；按模型 + 按 API 服务商 × 模型明细表
- **用量日历**：月度热力图，悬停看高峰 / 空闲拆分，点击进当天明细
- **缓存命中列表**：按时间筛选，分页渲染（每页 100 条）
- **价格表**：DeepSeek 官方价格，面板内可直接编辑并持久化
- **剩余余额查询**：用当前 `DEEPSEEK_API_KEY` 查余额
- **导出**：CSV / JSON / **PNG 长图**（最多 2000 条）
- **导入**：JSON / CSV 合并（按时间去重）
- **持久化**：`<会话工作区>/dsh-usage/usage-records.json`，上限 100000 条

**安装**：

```bash
dsh plugin --profile web add @feiyang666/deepseekharnessdesktop@1.9.0
```

**使用**：重启 → 顶栏 tab。

---

#### 18. `dsh-latex-tools` — 公式复制 / 导出

**定位**：悬停任意 LaTeX 公式（行内或块级），弹出工具条：**复制 LaTeX**（TeX 源码进剪贴板）或 **导出 SVG**（MathJax 渲染的自包含矢量图，字形轮廓内嵌为 path，无外部字体依赖），下载为 `formula-<slug>.svg`。

**安装**：

```bash
dsh plugin --profile web add dsh-latex-tools@0.1.2
```

**使用**：装完随 Web UI 自动加载，**无需手动启用**（没有配置项，所以不出现在 Settings → Plugins 列表里）。完全离线：MathJax 由插件自身 host 半边提供，首次导出按需加载，之后浏览器缓存。

**验证**：

```bash
dsh --profile web --dump-config | grep latex-tools     # 应看到 "# == dsh-latex-tools" 层
# 浏览器：/plugins/dsh-latex-tools/client.js
```

---

#### 19. `dsh-tool-excalidraw` — Excalidraw 白板

**定位**：让 agent 通过工具调用创建和编辑 **Excalidraw** 白板（`.excalidraw` v2 JSON，可直接在 [excalidraw.com](https://excalidraw.com) 打开继续编辑），导出独立 SVG（roughjs 手绘渲染）或 JSON。渲染用 roughjs（与 Excalidraw 同一引擎），纯 Node 运行，无需浏览器。

**安装**：

```bash
dsh plugin --profile web add dsh-tool-excalidraw@0.2.0
```

**7 个工具**：

| 工具 | 作用 |
| :--- | :--- |
| `excalidraw_create` | 新建白板（可带初始元素和画布尺寸） |
| `excalidraw_add_elements` | 追加元素，返回新元素 id |
| `excalidraw_update_elements` | 按 id 更新（移动 / 缩放 / 改文字 / 改颜色 / `isDeleted: true` 删除） |
| `excalidraw_group` | 按名称组合元素（同组在编辑器里联动，成员自动重排连续） |
| `excalidraw_organize` | 智能层级组织：按几何包含建父子 / 多层组合，检测同层重叠，可选自动避让 |
| `excalidraw_get` | 列出元素（`includeFull: true` 返回完整记录） |
| `excalidraw_export` | 导出 SVG（手绘）或 JSON |

**元素参数要点**：`type`（rectangle / ellipse / diamond / text / arrow / line / freedraw）、`x`/`y`、`width`/`height`、`text`（`\n` 换行）、`points`（line/arrow/freedraw 相对顶点）、`strokeColor`、`backgroundColor`、`fillStyle`（solid / hachure / cross-hatch / zigzag / dotted）、`strokeStyle`、`strokeWidth`、`roughness`（0–2）、`fontSize`、`group`。

**示例**：

```text
画一张微服务架构图，用 Excalidraw 手绘风格
把这个白板导出成 SVG
```

---

## 5. 前置条件速查

| 插件 | 额外依赖 / 注意 |
| :--- | :--- |
| 全部插件 | Node `^22.19` 或 `>= 24`；宿主 dsh `0.1.1-rc.2` 系列（peer 依赖） |
| 全部插件 | 装完**必须重启 `dsh web`** |
| `dshmarket` | 宿主 `dsh web ≥ 0.1.0-rc.6`；缺 pnpm 时市场会提示一键装 |
| `dsh-zotero` | Zotero ≥ 7 桌面版 + 开启本地 API（`127.0.0.1:23119`） |
| `dsh-doc` | 离线 OCR 运行时（Windows x64 有预编译包；其他平台用 `engine: node` 回退） |
| `dsh-mineru` | 可选 MinerU token（不填走限流的 Agent API） |
| `dsh-zagens-office` | 本机 `zagens-office` CLI（缺失时自动下载到 `~/.zagens-pro/bin`） |
| `dsh-sci-figure` | Python + matplotlib / numpy / scipy；`fig_env op=setup` 建 venv |
| `dsh-science` | 远程计算走系统 OpenSSH；零第三方依赖 |
| `dsh-univer-office` | Node ≥ 22.19；安装前建议先停 DSH |
| `@liustack/modlens` | 需要 modlens 引擎（`npx @liustack/modlens doctor` 检查） |
| `@vectorize-io/hindsight-*` | **不是** `dsh plugin add`，用它的独立 installer；需要 API token |
| `dsh-deepread` | 无；可选 export 落盘 |

---

## 6. 四套推荐组合

### 🧑💻 开发者日常（最省心）

```bash
dsh plugin --profile web add dsh-workbench-plugin dshmarket dsh-find-plugin dsh-skin
```

三栏 IDE + 插件市场 + 插件搜索 + 换肤。**先装这四个，体验提升最大。**

### 📄 文档 / 办公处理

```bash
dsh plugin --profile web add dsh-doc dsh-mineru dsh-zagens-office dsh-univer-office
```

本地解析兜底 + 云端高精度 + 两套 Office 生成 / 编辑。

### 🔬 科研全流程

```bash
dsh plugin --profile web add dsh-science dsh-sci-figure dsh-zotero dsh-deepread dsh-plugin-writing-guard
```

研究循环 → 文献证据 → 精读 → 制图 → 投稿前写作审计。

### 🌐 多媒体 / 交互

```bash
dsh plugin --profile web add @liustack/modlens @changfenhuang/dsh-genui dsh-tool-excalidraw dsh-latex-tools
```

看图 + 交互 UI + 画图 + 公式导出。

---

## 7. 排障 FAQ

**Q：`dsh plugin --profile web list` 没输出？**
A：`--profile` 是必填的；不写会报 `required option '--profile <name>' not specified`。另外某些终端下输出可能被缓冲，重定向到文件再 `cat` 最稳：`dsh plugin --profile web list > p.txt 2>&1; cat p.txt`。

**Q：装完没反应 / 工具不出现？**
A：99% 是没重启。`Ctrl+C` 停掉 `dsh web` 再启。少数插件（如 `dsh-univer-office`）明确建议先停再装。

**Q：Settings → Plugins 里找不到某个插件？**
A：那个列表**只显示有配置项的插件**。像 `dsh-latex-tools`、`dsh-skin` 没有配置项，不会出现在那里——它们是靠 UI 行为工作的。

**Q：`dsh-pdf-edit` 在 node_modules 里，怎么没生效？**
A：它不在 profile 的 `dependencies` 也不在 `bundles` 里，等于装了包但没挂载。要用就显式加：`dsh plugin --profile web add dsh-pdf-edit`。

**Q：装插件会不会污染别的 profile？**
A：不会。每个 profile 有独立的 `package.json` 和 `node_modules`。远程主机白名单也是**按项目**隔离的。

**Q：第三方插件安全吗？**
A：`dsh-find-plugin` 的结果都带 `dsh plugin add` 命令，但**插件是第三方代码**。建议先看源码并钉 commit（`github:owner/repo#<sha>`）。

**Q：怎么回滚某个插件？**
A：`dsh plugin --profile web add <包名>@<旧版本>` 即可覆盖。

**Q：Hindsight 报 401？**
A：本机未配 API key。需要在 Hindsight 侧配置 token（`npx @vectorize-io/hindsight-coding-agents install ...` 时选择记忆位置并配置）。

---

## 8. 与参考仓库的差异

参考仓库 [AkimotoHayao/my-dsh-config](https://github.com/AkimotoHayao/my-dsh-config) 是一份很好的入门清单，但有几处需要更新 / 纠正。本机实装与它的差异：

| 项目 | 参考仓库 | 本机实装 | 说明 |
| :--- | :--- | :--- | :--- |
| 插件数量 | 11 个 | **19 个** | 新增了 deepread、sci-figure、science、zotero、univer-office、find-plugin、latex-tools、market、genui、modlens、hindsight 等 |
| `dsh-workbench-plugin` | 0.1.13 | **0.1.31** | 变化很大：新增 Canvas、Notification sounds、Add to chat、Ultra Slash、AI commit message 等 |
| `dsh-science` | 0.1.1 | **0.2.0** | 新增远程计算引擎（16 工具）与模型分档路由 `dsh-model-tier` |
| `dsh-plugin-writing-guard` | 1.3.0 | **1.6.1** | 新增 Journal Profile、篇章统计层、证据状态守恒、双锁扩展 |
| `@omdsh-dev/dsh-genui` | 0.8.7 | `@changfenhuang/dsh-genui` **0.9.6** | **包名变了**，用旧名会装不到 |
| `@deepseek-ai/dsh-tool-web` | 列为独立插件 | 已并入核心 | 无需单独安装，随 `@deepseek-ai/dsh-base` 提供 |
| `dsh-zagens-office` | 0.1.0（npm） | 0.1.0（**GitHub 源**） | 官方安装方式是从 GitHub 装，不是 npm |
| `dsh-research-library` | "不建议安装" | 未安装 | 已由 `dsh-science` 覆盖同类能力 |
| 安装命令 | 不带 `--profile` | **必须带 `--profile web`** | 新版 CLI 强制 |
| 描述方式 | 多为"可能包含……" | **逐包核对** | 本文所有工具名来自注册代码，不是推测 |

**教训**：参考仓库里大量"可能包括……"的描述，是因为作者没核对源码。插件能力**必须**看 `package.json` 的 `description`、`README.md` 和 `cordis.patch.yml`，最好直接 grep 工具注册代码。

---

## 9. 参考链接

- **DeepSeek Harness 官方**：https://github.com/deepseek-ai/dsh
- **插件发现**：https://github.com/topics/dsh-plugin
- **Awesome DSH Plugin**：https://awesome-dsh-plugin.com
- **参考仓库**：[AkimotoHayao/my-dsh-config](https://github.com/AkimotoHayao/my-dsh-config)
- **本文涉及插件源码**（均取自各包 `package.json` 的 `repository` 字段，已核对）：

| 插件 | 仓库 |
| :--- | :--- |
| dsh-science / dsh-model-tier | https://github.com/biociao/dsh-science |
| dsh-zotero | https://github.com/Vncntvx/dsh-zotero |
| dsh-deepread | https://github.com/xiehuan123/dsh-deepread |
| dsh-plugin-writing-guard | https://github.com/xmutfyh/dsh-plugin-writing-guard |
| dsh-mineru | https://github.com/Lee-Hilex/dsh-mineru |
| dshmarket | https://github.com/dsh-market/dsh-market |
| @liustack/modlens | https://github.com/liustack/modlens |
| @changfenhuang/dsh-genui | https://github.com/omdsh-dev/dsh-genui |
| dsh-find-plugin | https://github.com/awesome-dsh-plugin/dsh-find-plugin |
| dsh-doc | https://github.com/Sqhao-O/dsh-docs |
| dsh-latex-tools | https://github.com/liuup/dsh-latex-tools |
| dsh-skin | https://github.com/Highjobop/dsh-gadgets |
| dsh-univer-office | https://github.com/dream-num/dsh-univer-office |
| dsh-zagens-office | https://github.com/didclawapp-ai/DSH-Office |
| @feiyang666/deepseekharnessdesktop | https://github.com/feiyang-dev/dsh-usage-plugin |
| Hindsight | https://vectorize.io/hindsight |

- **未公开仓库、只有 npm 页面的**（这些包 `package.json` 里没有 `repository` 字段，不要轻信二手链接）：
  - `dsh-workbench-plugin`：https://www.npmjs.com/package/dsh-workbench-plugin
  - `dsh-tool-excalidraw`：https://www.npmjs.com/package/dsh-tool-excalidraw
  - `dsh-sci-figure`：本机为本地 link（`/home/user/ppt_skills/dsh-sci-figure`），无公开发布

---

## 10. 备份与迁移你的插件配置

参考仓库叫 `my-dsh-config`，核心价值其实就是**把这套配置存下来**。DSH 的配置分散在几个文件里，想换机 / 重装时把它们一起带走即可。

### 需要备份的文件

| 文件 | 内容 | 重要性 |
| :--- | :--- | :--- |
| `~/.dsh/profiles/web/package.json` | **装了什么插件（含版本）** + `dsh.profile.bundles` 挂载顺序 | ⭐⭐⭐ 最关键 |
| `~/.dsh/profiles/web/cordis.patch.yml` | 插件的自定义配置（如 `allowedLocalRoots`、远程计算审批开关） | ⭐⭐⭐ |
| `~/.dsh/settings.yaml` | 模型 provider、默认模型、权限、UI 偏好 | ⭐⭐⭐ |
| `~/.dsh/profiles/web/pnpm-lock.yaml` | 锁定所有传递依赖的精确版本 | ⭐⭐ 想完全复刻时要 |
| `~/.dsh/skills/` | 自装 skills（本机有 19 个 nature-* 系列） | ⭐⭐ |
| `~/.dsh/profiles/web/node_modules/` | 依赖本体 | ❌ **不要备份**，几百 MB 且可用 lockfile 还原 |

> ⚠️ `.credentials.yaml` 含密钥，**别提交到公开仓库**。参考仓库那种公开 config 仓库尤其要注意。

### 备份

```bash
mkdir -p ~/my-dsh-config/profiles/web
cp ~/.dsh/profiles/web/package.json      ~/my-dsh-config/profiles/web/
cp ~/.dsh/profiles/web/cordis.patch.yml  ~/my-dsh-config/profiles/web/
cp ~/.dsh/profiles/web/pnpm-lock.yaml    ~/my-dsh-config/profiles/web/
cp ~/.dsh/settings.yaml                  ~/my-dsh-config/
cd ~/my-dsh-config && git init && git add -A && git commit -m "backup dsh config"
```

### 在新机器还原

```bash
# 1. 先装 DSH 本体
npm i -g @deepseek-ai/dsh

# 2. 还原 profile 清单，然后让 pnpm 按 lockfile 装齐
mkdir -p ~/.dsh/profiles/web
cp ~/my-dsh-config/profiles/web/{package.json,cordis.patch.yml,pnpm-lock.yaml} ~/.dsh/profiles/web/
cd ~/.dsh/profiles/web && pnpm install --frozen-lockfile

# 3. 还原 settings
cp ~/my-dsh-config/settings.yaml ~/.dsh/

# 4. 重启验证
dsh plugin --profile web list
```

> 本地 link 安装的插件（本机的 `dsh-sci-figure` 指向 `/home/user/ppt_skills/dsh-sci-figure`）在新机器上路径不存在，需要先 clone 源码再 `dsh plugin --profile web add /新路径/dsh-sci-figure`。

### 更省事的办法：用 dshmarket

`dshmarket` 自带**备份与恢复**：

- 导出 profile 的插件列表 + 配置为可读 JSON，导入到另一台机器
- 存到 WebDAV（每日自动备份）或私有 GitHub Gist 同步
- 恢复是**合并**语义（备份之后新装的插件会保留），写入前校验，失败回滚

入口：**Settings → Plugin Market → 备份与恢复**。

---

## 核对方式

本文数据来源（可在本机复现）：

```bash
# 1. 已装插件清单
dsh plugin --profile web list

# 2. 每个包的版本 / 描述 / dsh 声明
cd ~/.dsh/profiles/web/node_modules/<包名> && cat package.json

# 3. 工具名（从注册代码抓）
grep -rhoE "name: *[\"'][a-z][a-z0-9_]{2,}[\"']" <包名>/lib <包名>/dist <包名>/engines \
  | sed "s/name: *[\"']//;s/[\"']//" | sort -u

# 4. 挂载方式
cat <包名>/cordis.patch.yml   # 或 profile 的 package.json → dsh.profile.bundles

# 5. 核心包版本
ls ~/.nvm/versions/node/v22.23.2/lib/node_modules/@deepseek-ai/dsh/node_modules/@deepseek-ai/
```

---

*本文由 agent 基于本机实装环境核对生成。插件更新频繁，安装前建议再跑一次 `dsh plugin --profile web list` 核对版本。*
