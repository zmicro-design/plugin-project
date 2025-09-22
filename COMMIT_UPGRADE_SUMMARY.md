# Commit 功能升级总结

## 问题描述

原有的 `core/commit` 函数只支持 Git 仓库，并且硬编码使用了 `gpm commit`，无法支持 SVN 仓库。

## 解决方案

重写了 `core/commit` 函数以支持 Git 和 Git-SVN 两种仓库类型，同时更新了相关调用的文件。

## 主要修改

### 1. 扩展 `core/git_svn` 函数

添加了以下新函数：

- `git_svn::commit()` - Git-SVN 仓库的本地提交操作
- `git_svn::is_changed()` - 检测是否有未提交的更改（兼容 Git 和 Git-SVN）
- `git_svn::is_ready_to_commit()` - 检测是否准备好提交（兼容 Git 和 Git-SVN）

### 2. 重写 `core/commit`

主要改进：

- **自动检测仓库类型**：支持 git, git-svn, none 三种类型
- **统一的更改检测**：根据仓库类型使用相应的检测函数
- **灵活的提交方式**：
  - Git 仓库：有提交消息时使用 `git commit -m`，否则使用 `gpm commit`
  - Git-SVN 仓库：使用标准的 `git commit`
- **支持提交消息参数**：可以通过第一个参数传递提交消息
- **保持交互性**：如果没有文件 staged，会提示用户选择是否添加所有文件

### 3. 更新调用方 `core/sync`

- 添加仓库类型检测
- 使用统一的更改检测函数
- 保持原有逻辑不变

### 4. 更新调用方 `core/merge`

- 添加仓库类型检测
- 使用统一的更改检测函数
- 保持原有逻辑不变

## 新的函数签名

```bash
# 新的 commit 函数支持可选的提交消息
project::commit [commit_message] [add_type]

# 参数说明：
# commit_message: 可选的提交消息
# add_type: 可选的添加类型（add_all_files, commit_only_without_add, cancel）
```

## 使用示例

### Git 仓库

```bash
# 自动提交（会调用 gpm commit 进行交互）
project::commit

# 带消息提交
project::commit "Fix bug in authentication"

# 指定添加所有文件并提交
project::commit "Add new features" "add_all_files"
```

### Git-SVN 仓库

```bash
# 自动提交（使用 git commit）
project::commit

# 带消息提交
project::commit "Implement new feature"

# 指定添加所有文件并提交
project::commit "Update documentation" "add_all_files"
```

## 行为对比

| 场景 | Git 仓库 | Git-SVN 仓库 |
|------|----------|---------------|
| 有 staged 文件 + 无消息 | 使用 `gpm commit` | 使用 `git commit` |
| 有 staged 文件 + 有消息 | 使用 `git commit -m` | 使用 `git commit -m` |
| 无 staged 文件 | 提示用户选择是否添加文件 | 提示用户选择是否添加文件 |
| 检测更改 | 使用 `git::is_changed` | 使用 `git_svn::is_changed` |
| 检测 staged | 使用 `git::is_ready_to_commit` | 使用 `git_svn::is_ready_to_commit` |

## 兼容性

- **向后兼容**：对于 Git 仓库，行为基本保持不变
- **新功能**：Git-SVN 仓库现在完全支持 commit 操作
- **API 兼容**：原有的调用方式（无参数调用）完全兼容

## 技术细节

### 仓库类型检测
```bash
local repo_type=$(git_svn::detect_repo_type)
case $repo_type in
  "git") # Git 仓库逻辑 ;;
  "git-svn") # Git-SVN 仓库逻辑 ;;
  "none") # 错误处理 ;;
esac
```

### 更改检测
```bash
case $repo_type in
  "git")
    has_changes=$(git::is_changed)
    is_ready=$(git::is_ready_to_commit)
    ;;
  "git-svn")
    has_changes=$(git_svn::is_changed)
    is_ready=$(git_svn::is_ready_to_commit)
    ;;
esac
```

## 测试建议

### Git 仓库测试
```bash
cd git-repo
echo "test" > test.txt
git add test.txt
# 测试有 staged 文件的提交
project::commit "Test commit"

# 测试无参数提交
project::commit
```

### Git-SVN 仓库测试
```bash
cd git-svn-repo
echo "test" > test.txt
git add test.txt
# 测试有 staged 文件的提交
project::commit "Test SVN commit"

# 测试无参数提交
project::commit
```

### 集成测试
```bash
# 测试 sync 命令中的 commit 调用
project sync

# 测试 merge 命令中的 commit 调用
project merge some-branch
```

## 注意事项

1. Git-SVN 仓库的 commit 只是本地提交，要推送到 SVN 服务器还需要使用 `project push`
2. 保持了原有的交互式行为，如果没有 staged 文件会提示用户
3. Git 仓库在有提交消息时会绕过 `gpm commit`，直接使用 `git commit -m`
4. 所有相关的调用方（sync, merge）都已更新以支持新的仓库检测逻辑

通过这些修改，commit 功能现在完全支持 Git 和 Git-SVN 两种仓库类型，同时保持了向后兼容性。
