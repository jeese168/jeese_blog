# Mac开发环境配置笔记

## 网络代理设置（ClashX）

网络相关的永久配置一般写在macOS的`.zprofile`文件中。

### 方式一：可视化编辑`.zprofile`文件

使用TextEdit打开配置文件：

```zsh
open -a TextEdit ~/.zprofile
```

在打开的文本文件中添加：

```text
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

保存文件后，重新加载配置：

```bash
source ~/.zprofile
```

### 方式二：命令行直接写入`.zprofile`文件

```bash
echo 'export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890' >> ~/.zprofile
```

然后重新加载配置文件：

```bash
source ~/.zprofile
```

### 代理管理技巧

**查看当前代理设置：**

```bash
echo $https_proxy
echo $http_proxy
echo $all_proxy
```

**临时某个终端取消代理：**

```bash
unset https_proxy http_proxy all_proxy
```

查询终端走有没有走代理（根据ip判断）
```bash
curl ipinfo.io/ip
```

---

## 开发依赖设置（Homebrew）

### 1. Homebrew安装：一切的基石

**安装Homebrew：**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**安装后配置PATH（M系列芯片Mac）：**

```bash
# 配置写入.zprofile文件
echo 'eval $(/opt/homebrew/bin/brew shellenv) #homebrew' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

**验证安装：**

```bash
brew --version
```


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

### 3. 利用Homebrew安装Python相关工具（待使用）

**安装Python版本管理工具pyenv：**

```bash
brew install pyenv
```

**配置Python版本管理工具pyenv：**

```bash
# 添加到 ~/.zshrc 文件
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

# 重新加载配置
source ~/.zshrc
```

**验证配置是否生效：**

```bash
# 检查配置是否正确加载
echo $PYENV_ROOT
# 这个应该输出：/Users/jeese/.pyenv
```

**使用pyenv管理Python版本的不同环境（待使用）：**

```bash
# 查看可安装的Python版本
pyenv install --list
```

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