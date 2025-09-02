# Node.js 升级/安装指南

## 版本要求

- **Claude Code 需要 Node.js 18 或更高版本**
- 当前版本如果是 v16.x.x 需要升级



## 升级 Node.js 到 18+ 版本

### 方法一：官网下载（推荐）

1. 访问 https://nodejs.org/
2. 下载 LTS 版本（通常是 v18 或 v20）
3. 安装时会自动覆盖旧版本





### 方法二：使用 winget（Windows 包管理器）

```powershell
# 卸载旧版本
winget uninstall OpenJS.NodeJS

# 安装最新版本
winget install OpenJS.NodeJS
```

### 方法三：使用 nvm-windows（版本管理工具）

```powershell
# 安装指定版本
nvm install 18.17.0
nvm use 18.17.0
```

## 验证升级结果

```powershell
node --version
npm --version
```

**期望结果**：应该显示 v18.x.x 或更高版本


## 常见问题

### Q: 升级后原来的项目会受影响吗？

A: 一般不会，Node.js 向后兼容性较好，但建议升级后测试重要项目。

### Q: 如何卸载旧版本？

A: 新版本安装时会自动覆盖，也可以通过控制面板手动卸载。

### Q: npm 权限问题怎么办？

A: 在 Windows 上建议以管理员身份运行 PowerShell 执行全局安装命令。



## 注意事项

- 升级前建议备份重要项目
- 企业环境可能需要 IT 部门协助
- 某些旧项目可能需要使用 nvm 管理多个 Node.js 版本