# Claude Code Skills：可复用的 AI 工作流模板

### Skills 是什么

#### 本质

Skills 是存放在 `.claude/skills/` 目录下的 Markdown 文件，每个文件夹代表一个可复用的工作流模板。核心是 `SKILL.md`，包含 YAML 元数据和给 Claude 的执行指令。

调用方式：在对话中直接描述需求，Claude 会自动匹配并触发对应的 Skill。也可以在提示词中点名："用 artifacts-builder 生成一个..."。

#### 和 MCP 的区别

二者经常被混淆，但定位完全不同：

| | MCP | Skills |
|---|---|---|
| 本质 | 工具扩展（给 Claude 新能力） | 工作流模板（告诉 Claude 怎么做） |
| 形式 | 独立进程，JSON-RPC 通信 | Markdown 文件，提示词注入 |
| 例子 | Playwright MCP（操控浏览器）| artifacts-builder（怎么生成前端应用）|
| 依赖 | 需要运行 Server 进程 | 无运行时依赖 |

MCP 让 Claude **能做**某件事，Skills 让 Claude **知道怎么做**某件事。两者可以组合：比如 `webapp-testing` Skill 依赖 Playwright MCP 工具。

### 安装与使用

#### 两个层级

```bash
# 全局（所有项目可用）
~/.claude/skills/skill-name/SKILL.md

# 项目级（仅当前项目，随代码库走）
.claude/skills/skill-name/SKILL.md
```

Claude Code 启动时自动扫描加载，无需任何配置命令。

#### 推荐来源

- **ComposioHQ/awesome-claude-skills**：社区精选集合，覆盖开发、内容创作、效率工具等方向
- **anthropics/skills**：Anthropic 官方出品，专注文档处理（docx/pdf/pptx/xlsx）

### Skills 实战

#### artifacts-builder：终端里生成前端应用

网页版 Claude 可以直接在对话框里渲染交互式 React 组件（称为 Artifacts）。`artifacts-builder` 把这个能力搬到了终端 Claude Code 里。

**技术栈**：React 18 + TypeScript + Vite + Tailwind CSS 3 + shadcn/ui，打包工具用 Parcel，最终输出单个内联 HTML 文件。

##### 完整流程

第一步，初始化项目：

```bash
bash .claude/skills/artifacts-builder/scripts/init-artifact.sh <project-name>
```

脚本自动完成：创建 Vite + React + TypeScript 项目、安装 Tailwind CSS 3.4、安装 40+ shadcn/ui 组件（从预打包的 tarball 解压，速度快）、配置路径别名 `@/`。

第二步，编写组件。直接告诉 Claude 要生成什么，它会写 `src/App.tsx`。设计原则：避免 AI Slop 的典型特征——不要过度居中布局、不要紫色渐变、不要千篇一律的圆角。

第三步，打包：

```bash
cd <project-name>
bash ../.claude/skills/artifacts-builder/scripts/bundle-artifact.sh
```

输出 `bundle.html`，所有 JS/CSS 内联进单文件，可以直接拖进浏览器打开。

##### 实战：MCP 配置面板

以"生成一个展示当前 MCP 配置的仪表板"为例，最终效果：

- 顶部三栏统计：MCP Servers 数量、已加载 Skills 数量、工作流阶段数
- MCP Servers 卡片：展示每个 Server 名称、包名、状态、暴露的工具列表
- MCP 工作流时间轴：5 个阶段（启动→发现→决策→调用→返回），每步附带 JSON-RPC 示例
- Skills 分类卡片：按开发工具、内容创作、效率工具等分类展示
- 当前 `.mcp.json` 内容预览

整个过程：初始化约 30 秒，写组件约 2 分钟，打包约 10 秒，输出 276KB 的独立 HTML。

#### changelog-generator：把 commit 翻译成人话

> 待补充——试完后更新

#### content-research-writer：写论文/文章的全流程协作

> 待补充——写毕业论文时重点试用

#### mcp-builder：辅助构建 MCP Server

> 待补充——准备自己写 MCP Server 时更新

#### webapp-testing：Playwright 自动化测试本地应用

> 待补充

#### document-skills：Anthropic 官方文档处理套件

> 待补充——包含 docx、pdf、pptx、xlsx 四个子 Skill

### 按方向精选 Skills

#### LLM 应用开发方向

| Skill | 核心用途 |
|---|---|
| `claude-developer-platform` | 调用 Anthropic SDK / Claude API 构建应用的专用场景 |
| `mcp-builder` | 用 Python（FastMCP）或 TypeScript 快速构建 MCP Server |
| `langsmith-fetch` | 从 LangSmith 拉取 LangChain/LangGraph agent 执行 trace，用于调试 |
| `webapp-testing` | 配合 Playwright MCP 测试 Web 前端 |

#### 文档处理方向（Anthropic 官方）

| Skill | 核心用途 |
|---|---|
| `docx` | 创建/编辑/分析 Word 文档，支持修订追踪和注释 |
| `pdf` | 提取 PDF 文本、表格、元数据，支持合并与标注 |
| `pptx` | 读取/生成/调整 PowerPoint 幻灯片 |
| `xlsx` | 操作 Excel：公式、图表、数据转换 |

#### 写作与研究方向

| Skill | 核心用途 |
|---|---|
| `content-research-writer` | 研究 + 起草 + 逐节反馈的写作全流程协作 |
| `changelog-generator` | Git 提交历史 → 用户友好的版本更新日志 |

### 自己写 Skills

#### SKILL.md 结构

```markdown
---
name: skill-name
description: 清晰描述这个 Skill 做什么，以及什么时候触发。
              Claude 用这段描述决定是否激活这个 Skill。
---

# Skill Name

## 使用场景
- 什么时候用这个 Skill

## 执行步骤
[给 Claude 的详细操作指南，不是给用户看的说明书]

## 示例
[真实使用示例，帮助 Claude 理解预期输出]
```

#### 关键：description 决定触发时机

YAML 里的 `description` 是 Claude 判断"该不该用这个 Skill"的依据。写得越具体，触发越准确。可以参考 `skill-creator` Skill 来生成新 Skill 的骨架。
