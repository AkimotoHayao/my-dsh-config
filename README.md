# my-dsh-config

我的 DeepSeek Harness 插件集合与使用指南，本人目前研0，后续也会去更新，目前就是我这几天使用 dsh 的体验、安装的插件以及遇到的问题。

---

## 🧩 DeepSeek Harness 插件集合

这个仓库包含了我为 [DeepSeek Harness](https://github.com/deepseek-ai/dsh) 收集和配置的插件列表，旨在提升开发和工作效率。

---

## 📋 插件列表

| 插件名称 | 版本 | 功能描述 |
| :--- | :--- | :--- |
| `@deepseek-ai/dsh-tool-web` | 0.0.1-rc.1 | DeepSeek 官方 Web 工具集成 |
| `@feiyang666/deepseekharnessdesktop` | 1.9.0 | 第三方桌面环境增强插件 |
| `@omdsh-dev/dsh-genui` | 0.8.7 | 通用 UI 生成或界面工具 |
| `dsh-doc` | 0.1.1 | 文档处理或管理功能 |
| `dsh-mineru` | 0.1.9 | 数据挖掘与资源处理 |
| `dsh-plugin-writing-guard` | 1.3.0 | 写作辅助与内容检查 |
| `dsh-science` | 0.1.1 | 科学计算与数据处理工具 |
| `dsh-skin` | 0.4.1 | 界面主题或皮肤切换 |
| `dsh-tool-excalidraw` | 0.2.0 | 集成 Excalidraw 绘图工具 |
| `dsh-workbench-plugin` | 0.1.13 | 工作台功能扩展（文件编辑+终端+Git） |
| `dsh-zagens-office` | 0.1.0 | 办公相关功能集成 |

---

## 🧪 各插件详细介绍与使用体验

### 1. `@deepseek-ai/dsh-tool-web` — 官方 Web 工具集成

**使用介绍**：这是 DeepSeek 官方提供的 Web 工具集成插件，安装后会在 Harness 的 Web 界面中增加一系列官方工具入口。具体功能取决于官方版本包含的工具集，通常包括模型调试、数据查看等开发辅助功能。

**体验分享**：作为官方插件，稳定性不错，与 Harness 核心功能的融合度最高。打开 Web 界面后可以在顶栏或侧边栏找到新增的工具入口，操作直观，适合日常开发使用。

---

### 2. `@feiyang666/deepseekharnessdesktop` — 桌面环境增强

**使用介绍**：这是第三方开发者 `feiyang666` 提供的桌面环境增强插件，旨在优化 DeepSeek Harness 在桌面端的使用体验，可能包括更好的窗口管理、快捷键支持或系统托盘集成等功能。

**体验分享**：如果你是桌面端重度用户，这个插件能显著提升操作流畅度。不过作为第三方插件，建议关注其更新频率和兼容性。

---

### 3. `@omdsh-dev/dsh-genui` — 通用 UI 生成工具

**使用介绍**：`dsh-genui` 是一个通用 UI 生成插件，可以帮助你快速创建和定制 Harness 的用户界面组件。适合需要频繁调整界面布局或开发自定义界面的场景。

**体验分享**：对于想要个性化 Harness 界面的用户来说非常实用。通过简单的配置或命令就能生成新的 UI 元素，降低了界面定制的门槛。

---

### 4. `dsh-doc` — 文档处理与管理

**使用介绍**：`dsh-doc` 提供了文档处理和管理功能，可能包括文档的创建、编辑、格式转换、版本管理等能力。安装后可以通过 `dsh doc` 命令或界面入口访问。

**体验分享**：在需要频繁处理技术文档的工作流中非常有用。具体命令可以尝试 `dsh doc --help` 查看支持的操作。

---

### 5. `dsh-mineru` — 数据挖掘与资源处理

**使用介绍**：`dsh-mineru` 专注于数据挖掘和资源处理，可能包括数据抓取、清洗、分析等功能。适合需要进行数据处理和分析的场景。

**体验分享**：如果你经常处理数据集或需要进行资源分析，这个插件值得一试。建议安装后通过 `dsh mineru --help` 探索具体功能。

---

### 6. `dsh-plugin-writing-guard` — 写作辅助与内容检查

**使用介绍**：这是一个写作辅助插件，可以在你写作时提供内容检查、语法纠错、风格建议等功能。安装后会在编辑器或对话界面中集成辅助工具栏。

**体验分享**：对于需要大量写作的用户（如撰写文档、报告或论文）非常实用。它能实时检查内容问题，提升写作质量和效率。不过目前仅支持中文和英文内容检查。

---

### 7. `dsh-science` — 科学计算与数据处理

**使用介绍**：`dsh-science` 提供了科学计算和数据处理能力，可能集成了常用的数学库、统计工具或数据可视化功能。适合科研人员和数据分析师使用。

**体验分享**：在 Harness 中直接进行科学计算非常方便，无需切换到其他工具。具体支持哪些计算功能，可以通过 `dsh science --help` 查看。

---

### 8. `dsh-skin` — 界面主题切换

**使用介绍**：`dsh-skin` 允许你切换 DeepSeek Harness 的界面主题或皮肤，包括深色模式、浅色模式以及自定义配色方案。

**体验分享**：如果你对默认界面审美疲劳，或者需要在不同光线环境下工作，这个插件很实用。切换主题后界面焕然一新，操作也很简单，通常通过设置面板即可完成。

---

### 9. `dsh-tool-excalidraw` — Excalidraw 绘图工具集成

**使用介绍**：这个插件将知名的开源绘图工具 [Excalidraw](https://excalidraw.com/) 集成到了 DeepSeek Harness 中。安装后你可以在 Harness 界面中直接创建手绘风格的图表、流程图、架构图等。

**体验分享**：Excalidraw 的手绘风格非常讨喜，特别适合快速画图并与团队成员分享。在 Harness 中集成后，无需切换到浏览器就能完成绘图工作，提升了工作流的连贯性。打开方式通常是在界面中找到 Excalidraw 的入口按钮。

---

### 10. `dsh-workbench-plugin` — 工作台功能扩展 ⭐ 强烈推荐

**使用介绍**：这是功能最强大的插件之一，将 DeepSeek Harness 打造成一个类似 VSCode 的开发工作台。安装后，在 Conversation 界面中打开 Workbench，聊天区会保持在左侧，右侧新增两列：**编辑区（带语法高亮和终端）** 和 **文件与 Git 管理区**。

核心能力包括：
- **智能终端**：本地 PTY 终端，能自动识别输入是 Shell 命令还是自然语言。自然语言请求会被模型翻译成 Shell 命令并执行。
- **工作区编辑器**：基于 CodeMirror 6，支持 CSS、HTML、JavaScript、JSON、Markdown、Python、XML、YAML 的语法高亮。
- **Markdown 预览**：支持渲染图片和 Mermaid 图表。
- **文件与 Git**：文件树浏览、新建、重命名、删除；Git 侧边栏支持 status、diff、log、branch、commit（支持 AI 生成 commit message）等。
- **中英文双语支持**：界面自带中英文语言包。

**体验分享**：这是目前我用过最惊艳的插件，没有之一！安装后 Harness 直接变成了一个轻量级的 IDE。最让我惊喜的是**智能终端**——我直接输入“帮我安装 pandas”，它就能自动翻译成 `pip install pandas` 并执行，太适合我这种记不住命令的人了。文件树和 Git 管理也让项目操作变得非常顺手，强烈推荐给所有开发者！

**安装后**，重启 `dsh web`，打开 `http://127.0.0.1:3080`，进入 Conversation 后，在顶栏选择 **Workbench** 即可使用。

---

### 11. `dsh-zagens-office` — 办公功能集成

**使用介绍**：`dsh-zagens-office` 提供了办公相关功能的集成，可能包括文档编辑、表格处理、演示文稿制作等办公套件能力。

**体验分享**：如果你需要在 Harness 中处理办公文档，这个插件可以让你无需切换到其他办公软件。具体的办公功能建议安装后通过界面探索。

---

## ⚙️ 安装与使用

### 前提条件
确保你已经安装了 [DeepSeek Harness](https://github.com/deepseek-ai/dsh)。

### 安装插件
你可以使用 `dsh` 命令行工具来安装插件。

**1. 安装所有插件**
你可以逐个执行以下命令：

```bash
dsh plugin add @deepseek-ai/dsh-tool-web@0.0.1-rc.1
dsh plugin add @feiyang666/deepseekharnessdesktop@1.9.0
dsh plugin add @omdsh-dev/dsh-genui@0.8.7
dsh plugin add dsh-doc@0.1.1
dsh plugin add dsh-mineru@0.1.9
dsh plugin add dsh-plugin-writing-guard@1.3.0
dsh plugin add dsh-science@0.1.1
dsh plugin add dsh-skin@0.4.1
dsh plugin add dsh-tool-excalidraw@0.2.0
dsh plugin add dsh-workbench-plugin@0.1.13
dsh plugin add dsh-zagens-office@0.1.0
