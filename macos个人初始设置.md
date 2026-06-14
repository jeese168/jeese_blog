# Mac开发环境配置笔记

## 网络代理设置（ClashX Pro / v2rayN）

代理快捷函数写在 `~/.zshrc` 里（`.zshrc` 是交互式终端每次都会加载的文件，放函数和别名最合适；之前放在 `.zprofile` 也能用，但 `.zshrc` 更标准）。

**编辑 `.zshrc`（文件不存在先创建）：**

```bash
touch ~/.zshrc && open -e ~/.zshrc
```

把下面整段粘进去保存：

```zsh
# ========== Proxy Shortcuts ==========
CLASH_PORT=7890      # ClashX Pro 混合端口
V2RAY_PORT=10808     # v2rayN 混合端口

# 内部：按端口设置代理环境变量（大小写都设，兼容更多工具）
_proxy_on() {
  local p="$1"
  export http_proxy="http://127.0.0.1:$p"  https_proxy="http://127.0.0.1:$p"  all_proxy="socks5://127.0.0.1:$p"
  export HTTP_PROXY="http://127.0.0.1:$p"   HTTPS_PROXY="http://127.0.0.1:$p"   ALL_PROXY="socks5://127.0.0.1:$p"
}

# pc - 开 Clash 代理
pc() {
  nc -z -G 1 127.0.0.1 "$CLASH_PORT" 2>/dev/null || { echo "⚠️  Clash 端口 $CLASH_PORT 未监听，请先启动 ClashX Pro"; return 1; }
  _proxy_on "$CLASH_PORT"; echo "✅ Clash proxy ON ($CLASH_PORT)"
}

# pv - 开 V2Ray 代理
pv() {
  nc -z -G 1 127.0.0.1 "$V2RAY_PORT" 2>/dev/null || { echo "⚠️  V2Ray 端口 $V2RAY_PORT 未监听，请先启动 v2rayN"; return 1; }
  _proxy_on "$V2RAY_PORT"; echo "✅ V2Ray proxy ON ($V2RAY_PORT)"
}

# po - 关闭所有代理
po() {
  unset http_proxy https_proxy all_proxy HTTP_PROXY HTTPS_PROXY ALL_PROXY
  echo "❌ Proxy OFF"
}

# ps - 状态 + Google 连通性测试（覆盖系统 ps，查进程用 command ps）
ps() {
  [ -n "$http_proxy" ] && echo "状态: ON  $http_proxy" || echo "状态: OFF"
  echo "--- 连通性测试 (Google) ---"
  local code
  code=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 https://www.google.com/generate_204 2>/dev/null)
  if [ "$code" = "204" ] || [ "$code" = "200" ]; then
    echo "✅ Google 可达，代理生效 (HTTP $code)"
  elif [ -n "$http_proxy" ]; then
    echo "❌ Google 不可达，代理已设置但未生效——检查客户端/端口"
  else
    echo "❌ Google 不可达，且未开启代理"
  fi
}
```

保存后重新加载配置：

```bash
source ~/.zshrc
```

### 用法速查

| 命令 | 作用 |
|------|------|
| `pc` | 开 Clash 代理（ClashX Pro，端口 7890）|
| `pv` | 开 V2Ray 代理（v2rayN，端口 10808）|
| `po` | 关闭所有代理 |
| `ps` | 查看代理状态 + 测试 Google 是否可达 |

> `pc` / `pv` 会先检测对应端口在不在监听，没开客户端会直接提示。
> `ps` 覆盖了系统查进程的 `ps` 命令；要查进程用 `command ps aux`，介意的话把函数名改成 `pst` 即可。

---

## 开发依赖设置（Homebrew）

### 1. Homebrew安装：一切的基石

**安装 Homebrew（国内推荐走清华镜像）：**

官方脚本直连 GitHub，国内经常卡在 `Downloading and installing Homebrew...` 这一步（要 clone 一个 30 多万对象的大仓库）。直接用清华镜像装，又快又稳：

```bash
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git /tmp/brew-install
/bin/bash /tmp/brew-install/install.sh
rm -rf /tmp/brew-install
```

> 装完把这几个临时变量 `unset` 掉，避免长期绑定镜像（以后更新挂代理直连即可）：
>
> ```bash
> unset HOMEBREW_BREW_GIT_REMOTE HOMEBREW_API_DOMAIN HOMEBREW_BOTTLE_DOMAIN
> ```
>
> 如果网络能直连或已挂代理，也可以用官方脚本：
>
> ```bash
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
> ```

**安装后配置 PATH（M 系列芯片 Mac，写入 `.zprofile`）：**

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# 验证
brew --version
```

**关闭 Homebrew 自动更新（国内强烈建议，写入 `.zshrc`）：**

默认每次 `brew install` 前都会先同步配方库，国内这步常比装软件本身还慢。关掉它：

```bash
# 添加到 ~/.zshrc
echo 'export HOMEBREW_NO_AUTO_UPDATE=1' >> ~/.zshrc   # 关闭安装前的自动更新
echo 'export HOMEBREW_NO_ANALYTICS=1' >> ~/.zshrc     # 关闭匿名统计上报（可选）

source ~/.zshrc
```

验证是否生效（输出 `1` 即可）：

```bash
echo $HOMEBREW_NO_AUTO_UPDATE
```

> 代价很小：配方库不再自动更新，偶尔想装新东西发现版本旧了，手动跑一次 `brew update` 即可。


### 2. 利用Homebrew安装Java相关工具

**安装多版本Java：**

```bash
# 安装OpenJDK 17（推荐用于Spring Boot 3.x）
brew install openjdk@17

# 安装OpenJDK 21（最新LTS版本）
brew install openjdk@21
```

**符号链接创建（都要创建）：**

```bash
# 创建JDK 17的符号链接
sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk

# 创建JDK 21的符号链接
sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
```

**配置Java环境变量（默认配置21，特殊项目在IDEA的 `File` → `Project Structure` → `Project SDK` 选择openjdk@17）：**

```bash
# 添加到 ~/.zshrc 文件
echo 'export JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home' >> ~/.zshrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.zshrc

# 重新加载配置
source ~/.zshrc
```

**安装Maven：**

```bash
brew install maven

# 验证安装
mvn --version
```

### 3. 利用Homebrew安装Python相关工具（uv）

**为什么用 uv 不用 pyenv：** uv 一个工具就搞定「Python 版本管理 + 虚拟环境 + 装包 + 依赖锁定」（相当于 pyenv + venv + pip + poetry 合一），还是 Rust 写的、极快。装了 uv 就不需要 pyenv。

**安装 uv：**

```bash
brew install uv
```

**安装 / 管理 Python 版本（替代 pyenv 的活）：**

```bash
uv python install 3.12   # 安装指定版本
uv python list           # 查看已安装 / 可安装的版本
```

**新建项目（自动生成 pyproject.toml）：**

```bash
uv init myproject
cd myproject
```

**装包 / 删包（自动建 .venv 并记录到 pyproject.toml）：**

```bash
uv add requests      # 加依赖
uv remove requests   # 删依赖
```

**运行代码（自动用项目的 .venv，不用手动 activate）：**

```bash
uv run main.py
```

**恢复已有项目的依赖环境：**

```bash
uv sync                              # 按 uv.lock / pyproject.toml 还原环境
uv pip install -r requirements.txt   # 老项目只有 requirements.txt 时用这个
```

> - uv 是「项目级」管理：虚拟环境 `.venv` 就在项目根目录，`uv run` 自动激活，不用 `source .venv/bin/activate`。
> - 不用担心磁盘：uv 有全局缓存，同一版本的包只存一份，各项目通过链接共享，逻辑隔离、物理不重复。

---

## 常用开发工具

### 终端增强

```bash
brew install zsh-autosuggestions
brew install zsh-syntax-highlighting
brew install tree  # 目录树显示
brew install htop  # 系统监控
```

---

### 可选操作（配置了代理，连接官网一般就不需要）

**配置HOMEBREW国内镜像（可选，提升下载速度）：**
**中科大镜像**
```bash
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
```

**配置Maven国内镜像： 创建或编辑 `~/.m2/settings.xml`：**

```xml
<settings>
  <mirrors>
    <mirror>
      <id>aliyun</id>
      <mirrorOf>central</mirrorOf>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>
</settings>
```


---
## 项目迁移注意事项

### Homebrew迁移

所有M系列芯片Mac的Homebrew都安装在 `/opt/homebrew`，迁移时：

1. 在旧机器：`brew bundle dump` 生成Brewfile
2. 在新机器：`brew bundle install` 一键安装所有软件

### 配置文件备份

重要配置文件：

- `~/.zprofile` - 环境变量和别名
- `~/.gitconfig` - Git配置
- `~/.m2/settings.xml` - Maven配置
- `~/.ssh/` - SSH密钥

建议将这些配置文件加入私有dotfiles仓库统一管理。
