# Gemini CLI 基础配置实战：MCP 与 GEMINI.md

### 第一阶段：项目准则与本地指令 (GEMINI.md)

在 AI Agent 的工作流中，`GEMINI.md` 是其理解项目背景的“第一窗口”。

#### 1. 核心指令：默认简体中文
为了提高沟通效率，我们在 `GEMINI.md` 中明确增加了一项关键指令。

##### 语言偏好设置
在 `Language Preference` 章节中加入：`请默认使用简体中文回复。` 这确保了 Agent 在处理复杂技术任务时，能始终保持母语沟通，降低理解成本。

### 第二阶段：MCP 工具链的深度配置

MCP (Model Context Protocol) 是赋予 Agent 外部操作能力的核心协议。

#### 1. 全局配置 settings.json
虽然我们尝试了项目级的局部配置，但为了确保工具在所有工作空间的一致性，最终将其收拢于全局配置文件 `~/.gemini/settings.json`。

#### 2. 三大核心服务器能力

##### GitHub MCP
通过配置个人访问令牌 (PAT)，Agent 获得了操作远程代码库的能力，包括搜索仓库、处理 Issue 以及直接推送代码。

##### Filesystem MCP
这是 Agent 的“触角”。通过设置路径白名单（如 `/Users/jeese/Projects`），Agent 能够跨越当前项目目录，安全地读取和整合本地其他资源。

##### Playwright MCP
赋予 Agent 浏览器操控能力。配合 `npx playwright install` 命令，Agent 可以实时访问网页并抓取动态内容，使其具备了获取互联网实时信息的能力。

### 第三阶段：总结与后续预留

本次配置完成了 Gemini CLI 从“聊天机器人”到“行动派 Agent”的初步转型。

#### 1. 当前状态
目前的 Agent 已具备基本的本地文件处理、远程代码库读写和初步的网页访问能力。

#### 2. 待续点
后续我们将探索如何基于此环境构建更复杂的自动化工作流，并尝试开发自定义的 Skills 以应对特定业务场景。

---
**日期**：2024-02-28
**参与者**：jeese168 & Gemini CLI
