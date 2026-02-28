# Claude Code 基础配置实战：MCP 与 CLAUDE.md

### 第一阶段：项目准则与本地指令 (CLAUDE.md)

在 AI Agent 的工作流中，`CLAUDE.md` 是其理解项目背景的"第一窗口"。

#### 1. 核心指令：默认简体中文
为了提高沟通效率，我们在 `CLAUDE.md` 中明确增加了一项关键指令。

##### 语言偏好设置
在 `Language Preference` 章节中加入：`请默认使用简体中文回复。` 这确保了 Agent 在处理复杂技术任务时，能始终保持母语沟通，降低理解成本。

### 第二阶段：MCP 工具链的深度配置

MCP (Model Context Protocol) 是赋予 Agent 外部操作能力的核心协议。

#### 1. 配置层级

Claude Code 支持两个层级的 MCP 配置：

- **全局配置**：`~/.claude/settings.json`，对所有项目生效
- **项目级配置**：`.mcp.json`（项目根目录），只对当前项目生效，可提交至 Git（注意将含有敏感信息的文件加入 `.gitignore`）

与某些 Agent 工具不同，Claude Code 的项目级 `.mcp.json` **完全生效**，实现真正的工作区隔离。

#### 2. 核心服务器能力

##### GitHub MCP
通过配置个人访问令牌 (PAT)，Agent 获得了操作远程代码库的能力，包括搜索仓库、处理 Issue 以及直接推送代码。

##### Playwright MCP
赋予 Agent 浏览器操控能力。配置 `msedge` 作为目标浏览器后，Agent 可以实时访问网页并抓取动态内容，使其具备了获取互联网实时信息的能力。

##### ~~Filesystem MCP~~ —— 对 Claude Code 无效，请勿配置

> **踩坑记录（2026-02-28）**
>
> 在配置过程中，我们尝试引入 `@modelcontextprotocol/server-filesystem`，期望让 Agent 能跨目录访问本地文件。然而经过反复排查（含调试包装脚本抓包验证），得出结论：
>
> **Filesystem MCP 对 Claude Code 用户几乎没有实际价值。**
>
> 原因如下：
>
> 1. **Claude Code 自带文件操作能力**：`Read`、`Write`、`Edit`、`Grep`、`Glob` 等内置工具已能完整覆盖本地文件读写需求，无需额外 MCP。
>
> 2. **Claude Code 有自己的安全层**：即便在 `.mcp.json` 中配置了多个目录路径，Claude Code 会在底层拦截 Filesystem MCP 的响应，将文件访问权限强制限制在**当前启动目录**。调试日志证实，服务器进程确实收到了两个目录路径，但 Claude Code 的安全机制过滤掉了非当前目录。
>
> 3. **配置混乱的陷阱**：`@modelcontextprotocol/server-filesystem` 包在多实例场景下存在**工具名称冲突**问题——两个实例注册的工具名完全相同（如 `read_file`、`list_directory`），Claude Code 会将所有调用路由到第一个实例，导致第二个实例形同虚设。
>
> **结论**：直接删掉 Filesystem MCP，用 Claude Code 内置文件工具即可，省时省力。

### 实践过程中的要点与对比

在配置过程中，我们总结了几个与其他 Agent 工具的关键差异点。

#### 1. 项目级配置完全生效

##### 优势对比
与 Gemini CLI 不同，Claude Code 的项目级 MCP 配置（`.mcp.json`）**真正有效**。无需退回到全局配置，每个项目可以独立管理自己的 MCP 工具链，实现干净的工作区隔离。

#### 2. 权限审批机制

##### Agent 的权限边界
Claude Code 内置了清晰的权限审批机制。对于高风险操作（如写入文件、执行命令、推送代码），Agent 会主动提示用户确认，而非静默执行。这种透明度使用户始终掌握操作主动权。

#### 3. 环境依赖的"最后一公里"

##### 运行环境的完整性
即便在配置中正确定义了 Playwright 服务器，也需要确保目标浏览器（如 Microsoft Edge）已安装于本地。工具的注册并不代表环境的就绪，配置完成后需验证底层依赖是否满足。

### 第三阶段：总结与后续预留

本次配置完成了 Claude Code 从"聊天机器人"到"行动派 Agent"的初步转型。

#### 1. 当前最终配置（`.mcp.json`）

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-pat>" }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--browser", "msedge"]
    }
  }
}
```

#### 2. 待续点
后续我们将探索如何基于此环境构建更复杂的自动化工作流，并尝试开发自定义的 Skills 以应对特定业务场景。

---
**日期**：2026-02-28  
**参与者**：jeese168 & Claude Code
