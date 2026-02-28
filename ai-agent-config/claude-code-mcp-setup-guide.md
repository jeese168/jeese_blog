# Claude Code 基础配置实战：MCP 与 CLAUDE.md

### CLAUDE.md：给 Agent 的项目说明书

#### 作用

在 AI Agent 的工作流中，`CLAUDE.md` 是 Claude Code 理解项目背景的第一入口。每次启动时，Claude Code 会自动读取项目根目录下的 `CLAUDE.md`，将其作为持久上下文注入对话。

项目规范、语言偏好、构建命令、目录约定——都可以写在这里，避免每次对话重复说明。

#### 语言偏好配置

##### 设置默认简体中文

在 `CLAUDE.md` 的 `Language Preference` 章节中加入：

```
请默认使用简体中文回复。
```

这确保了 Agent 在处理复杂技术任务时，始终以母语沟通，降低理解摩擦。

### MCP 配置

#### 两个配置层级

Claude Code 支持两个层级的 MCP 配置：

- **全局配置**：`~/.claude/settings.json`，对所有项目生效
- **项目级配置**：项目根目录下的 `.mcp.json`，只对当前项目生效

与某些 Agent 工具（如 Gemini CLI）不同，Claude Code 的项目级 `.mcp.json` **真正生效**，实现了干净的工作区隔离，每个项目可以独立管理自己的工具链，无需污染全局配置。

> 注意：`.mcp.json` 中通常含有 API Token 等敏感信息，记得加入 `.gitignore`。

#### 可用的 MCP 服务器

##### GitHub MCP

通过配置个人访问令牌（PAT），Agent 获得了操作远程代码库的完整能力：搜索仓库、读写文件、处理 Issue、推送代码。

本文所有的博客更新，均通过 GitHub MCP 直接完成，无需手动 git 操作。

##### Playwright MCP

赋予 Agent 浏览器操控能力。配置 `msedge` 作为目标浏览器后，Agent 可以打开网页、点击按钮、填写表单、下载文件，真正具备与网页世界交互的能力。

> 需要确保目标浏览器（如 Microsoft Edge）已安装于本地。工具注册不代表环境就绪，配置后需实际验证。

##### ~~Filesystem MCP~~ —— 对 Claude Code 无效，请勿配置

> **踩坑记录（2026-02-28）**
>
> 在配置过程中，我们尝试引入 `@modelcontextprotocol/server-filesystem`，期望让 Agent 能跨目录访问本地文件。然而经过反复排查（含调试包装脚本抓包验证），得出结论：
>
> **Filesystem MCP 对 Claude Code 用户几乎没有实际价值。**
>
> 原因有三：
>
> 1. **Claude Code 内置文件工具已足够**：`Read`、`Write`、`Edit`、`Grep`、`Glob` 等内置工具完整覆盖本地文件读写需求，无需额外 MCP。
>
> 2. **Claude Code 有自己的安全层**：即便 `.mcp.json` 中配置了多个目录路径，调试日志证实 Server 进程确实收到了所有路径，但 Claude Code 会在上层将文件访问强制限制在**当前启动目录**，其他路径的访问请求均被过滤。
>
> 3. **多实例工具名冲突**：同一个包的两个实例会注册完全相同的工具名（如 `read_file`、`list_directory`），Claude Code 将所有调用路由到第一个实例，第二个形同虚设。这是 MCP 协议层面要求工具名全局唯一的约束。
>
> **结论**：直接删掉，用内置工具即可。

### MCP 工作原理

#### MCP 是什么

MCP（Model Context Protocol）是 Anthropic 于 2024 年发布的开放标准协议，定义了 AI 模型与外部工具之间的通信规范。

类比：MCP 是 AI 世界的 USB 标准。USB 出现之前，每个设备都有专属接口互不兼容；USB 统一规范后，任何设备即插即用。MCP 做的是同样的事——统一了 AI 与外部工具之间的接口，让工具开发者只需实现一次协议，就能被所有支持 MCP 的 AI 调用。

#### 通信底层：JSON-RPC over stdio

MCP 使用 **JSON-RPC 2.0** 协议，默认通过 **stdio（标准输入输出）** 在 Client 和 Server 之间传输消息。

Claude Code 启动一个 MCP Server 时的实际流程：
1. 以子进程运行 Server 程序（即 `.mcp.json` 里的 `command` + `args`）
2. 通过 **stdin** 向 Server 发送 JSON 格式的请求
3. 从 **stdout** 读取 Server 返回的 JSON 响应

#### 工作流程

##### 阶段一：启动与工具发现

Claude Code 启动时，读取 `.mcp.json`，依次拉起各 Server 子进程，向每个 Server 发送 `tools/list` 请求，获取其暴露的工具列表（工具名、参数 Schema、功能描述）。

这一步完成后，Claude 建立起"我有哪些工具可用"的完整清单。**工具名在此时必须全局唯一**——这是 Filesystem MCP 多实例方案失败的根本原因。

##### 阶段二：模型决策与工具调用

Claude 分析用户的自然语言请求，判断是否需要工具、调用哪个、传入什么参数，然后向对应 Server 发送 `tools/call` 请求。Server 执行操作后以 JSON 格式返回结果。

##### 阶段三：结果整合

Server 返回内容可以是文本（`type: "text"`）、图片（`type: "image"`）或资源链接。Claude 将其作为上下文继续生成最终回复，整个过程对用户透明。

### Playwright MCP 实战：操控浏览器下载文件

以下是一次完整的演示：通过 Playwright MCP 控制 Edge 浏览器，找到湖北理工学院计算机学院官网，下载 2025-2026 学年教学日历。

#### 搜索并导航至官网

Claude Code 将用户意图逐步转化为 MCP 工具调用序列：

```
browser_navigate  →  打开必应
browser_type      →  输入"湖北理工学院计算机学院"并回车
browser_navigate  →  识别官网域名 cs.hbpu.edu.cn，直接导航
browser_hover     →  悬停"下载中心"，展开下拉子菜单
browser_navigate  →  进入"教学日历"页面
browser_click     →  点击 2025-2026 学年校历
```

##### 为什么 Dock 里出现了两个 Edge 图标

Playwright 启动浏览器时，会通过 `--user-data-dir` 参数指定一个**全新的隔离用户目录**，让系统认为这是一个独立的 Edge 实例——与用户日常使用的 Edge 完全隔离，互不影响数据。macOS 因此在 Dock 栏显示了第二个 Edge 图标。关掉 Playwright 控制的窗口后，第二个图标消失。

#### 定位附件并触发下载

在详情页找到附件链接后：

```
browser_click  →  点击"2025-2026学年校历最新20250825.jpg"下载链接
```

Playwright Server 捕获浏览器的下载事件，文件自动保存：

```
Downloaded: 2025-2026学年校历最新20250825.jpg
→ 保存路径: .playwright-mcp/2025-2026学年校历最新20250825.jpg
```

全程 Claude 没有直接"访问网页"，而是通过 MCP 协议指挥 Playwright Server，由 Server 真正操控浏览器完成所有动作。这就是 MCP 架构的价值所在。

### 当前最终配置

#### .mcp.json

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

#### 后续方向

Claude Code 从"聊天机器人"到"行动派 Agent"的基础转型已完成。后续将探索基于此环境构建更复杂的自动化工作流，以及开发自定义 Skills 以应对特定业务场景。

---
**日期**：2026-02-28  
**参与者**：jeese168 & Claude Code
