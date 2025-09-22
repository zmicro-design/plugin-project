# Git SVN 支持使用指南

本项目现在支持通过 `git svn` 来管理 SVN 仓库，让您可以使用统一的命令接口管理 Git 和 SVN 项目。

## 前置要求

确保您的系统已安装 `git-svn`：

```bash
# Ubuntu/Debian
sudo apt-get install git-svn

# CentOS/RHEL
sudo yum install git-svn

# macOS
brew install git
```

## 基本使用流程

### 1. 添加 SVN 仓库到工作空间

```bash
# 添加标准布局的 SVN 仓库 (trunk/branches/tags)
project add http://svn.example.com/myproject myproject -s

# 添加自定义布局的 SVN 仓库
project add http://svn.example.com/myproject myproject --trunk=main --branches=feature --tags=release

# 从特定修订号开始添加
project add http://svn.example.com/myproject myproject -s --revision=1000
```

### 2. 进入项目并查看信息

```bash
cd myproject

# 查看 Git 状态（适用于所有类型的仓库）
git status

# 查看分支信息
git branch -a

# 查看 SVN 信息（仅限 git-svn 仓库）
git svn info
```

### 3. 日常开发工作流

```bash
# 拉取最新更改 (相当于 svn update)
project pull

# 查看当前状态
git status

# 添加文件
git add .

# 提交更改 (本地 Git 提交)
git commit -m "Add new feature"

# 推送到 SVN 仓库 (相当于 svn commit)
project push

# 同步所有更改 (拉取 + 推送)
project sync
```

### 4. 分支管理

```bash
# 查看所有远程分支
git branch -r

# 切换到 SVN 分支（使用 use 命令）
project use branches/feature-123

# 或者使用标准 Git 命令切换到远程分支
git checkout -b feature-123 remotes/origin/feature-123

# 在 SVN 服务器上创建新分支（使用标准 git svn 命令）
git svn branch new-feature

# 合并分支 (使用标准 Git 命令)
git merge feature-123
```

### 5. 标签管理

```bash
# 查看所有标签
git tag

# 创建标签（使用标准 git svn 命令）
git svn tag v1.0.0

# 切换到标签
project use tags/v1.0.0
```

## 命令对照表

| 操作 | SVN 命令 | 项目命令 | Git SVN 等效命令 |
|------|----------|----------|------------------|
| 克隆仓库 | `svn checkout` | `project add <svn-url>` | `git svn clone` |
| 更新代码 | `svn update` | `project pull` | `git svn rebase` |
| 提交代码 | `svn commit` | `project push` | `git svn dcommit` |
| 查看状态 | `svn status` | `git status` | `git status` |
| 查看信息 | `svn info` | `git svn info` | `git svn info` |
| 创建分支 | `svn copy` | `git svn branch` | `git svn branch` |
| 创建标签 | `svn copy` | `git svn tag` | `git svn tag` |
| 切换分支 | `svn switch` | `project use <branch>` | `git checkout` |
| 查看日志 | `svn log` | `git log` | `git log` |

## 主要特性

### 自动检测
- 项目会自动检测当前目录的仓库类型（git/git-svn/none）
- 根据仓库类型自动选择相应的操作逻辑
- 无需记忆不同的命令集

### 统一接口
- `project add` - 支持 Git 和 SVN 仓库的克隆
- `project pull` - 支持 Git pull 和 SVN update
- `project push` - 支持 Git push 和 SVN commit  
- `project sync` - 支持完整的同步操作
- `project use` - 支持 Git 和 SVN 的分支切换

### 向后兼容
- 完全兼容现有的 Git 仓库工作流
- 现有用户的使用方式不受任何影响

## 使用示例

### 新 SVN 项目
```bash
# 添加 SVN 仓库到工作空间
project add http://svn.example.com/repo myproject -s

# 进入项目目录
cd myproject

# 日常开发
echo "new feature" > feature.txt
git add feature.txt
git commit -m "Add new feature"
project push

# 更新代码
project pull

# 同步操作
project sync
```

### 现有 Git 项目
```bash
# 原有工作流完全不变
project pull
git add .
git commit -m "changes"
project push
```

### SVN 分支操作
```bash
# 查看所有远程分支
git branch -r

# 切换到现有分支
project use branches/develop

# 创建新分支
git svn branch feature-new

# 切换到新创建的分支
project use branches/feature-new
```

## 高级用法

### 查看 SVN 和 Git 信息

```bash
# 查看 SVN 详细信息
git svn info

# 查看 Git 分支
git branch -a

# 查看远程 SVN 分支
git branch -r

# 查看 SVN 修订号对应关系
git log --oneline --grep="git-svn-id"
```

### 处理冲突

```bash
# 如果在 push 时发生冲突
project pull  # 先拉取最新更改
# 解决冲突后
git add .
git commit -m "Resolve conflicts"
project push
```

### 重置到特定 SVN 修订号

```bash
# 查看 SVN 修订历史
git svn log

# 重置到特定修订号
git svn reset -r 1234
```

## 配置选项

### project add 命令的 SVN 选项

- `--stdlayout` 或 `-s`: 使用标准 SVN 布局 (trunk/branches/tags)
- `--trunk=<dir>`: 指定 trunk 目录 (默认: trunk)
- `--branches=<dir>`: 指定 branches 目录 (默认: branches)  
- `--tags=<dir>`: 指定 tags 目录 (默认: tags)
- `--revision=<rev>`: 从特定修订号开始

### 示例
```bash
# 标准布局
project add http://svn.example.com/repo myproject -s

# 自定义布局
project add http://svn.example.com/repo myproject --trunk=main --branches=feature

# 从特定修订号开始
project add http://svn.example.com/repo myproject -s --revision=1000
```

## 注意事项

1. **仓库检测**：项目会自动检测当前目录是 Git 仓库还是 Git-SVN 仓库，并使用相应的命令。

2. **分支切换**：对于 SVN 仓库，使用 `project use branches/branch-name` 来切换分支。

3. **冲突处理**：使用标准的 Git 命令来处理合并冲突。

4. **本地分支**：您可以创建本地 Git 分支进行开发，然后推送到 SVN。

5. **历史记录**：使用 `git log` 查看提交历史，它会显示对应的 SVN 修订号。

## 故障排除

1. **权限问题**：确保您有 SVN 仓库的读写权限。

2. **网络问题**：SVN 操作可能比 Git 慢，请耐心等待。

3. **编码问题**：如果遇到中文乱码，设置正确的编码：
   ```bash
   git config --global core.quotepath false
   ```

4. **SSL 证书问题**：如果 SVN 服务器使用 HTTPS，可能需要接受证书。

5. **分支问题**：如果无法切换到 SVN 分支，尝试先 `git svn fetch` 获取最新信息。

通过这种方式，您可以继续使用熟悉的项目管理命令，同时无缝地支持 SVN 仓库，而无需学习新的命令。