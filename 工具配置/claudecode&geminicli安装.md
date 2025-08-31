## 安装 AI 开发工具

确认 Node.js 版本为 18+ 后，可以安装以下工具：

### Claude Code

```powershell
npm install -g @anthropic-ai/claude-code
```

### Gemini CLI（正确包名）

```powershell
npm install -g @google/gemini-cli
```



## 验证安装

```powershell
# 检查 Claude Code
claude-code --version

# 检查 Gemini CLI
gemini --version

# 查看帮助信息
claude-code --help
gemini --help
```



## 使用方法

### Claude Code

```powershell
# 启动 Claude Code
claude-code

# 在特定目录中使用
cd your-project-folder
claude-code
```

### Gemini CLI

```powershell
# 启动 Gemini CLI
gemini

# 首次使用需要用 Google 账号认证
```

