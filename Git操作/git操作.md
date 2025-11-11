# Git 完全指南

## 目录

1. [Git 基础概念](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#git-基础概念)
2. [安装与配置](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#安装与配置)
3. [基础操作](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#基础操作)
4. [分支管理](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#分支管理)
5. [远程仓库操作](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#远程仓库操作)
6. [高级操作](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#高级操作)
7. [撤销与回退](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#撤销与回退)
8. [标签管理](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#标签管理)
9. [Git工作流](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#git工作流)
10. [问题解决](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#问题解决)
11. [最佳实践](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#最佳实践)
12. [Git内部原理](https://claude.ai/chat/b2addd2a-97aa-4f4b-802e-470a15f7d95c#git内部原理)

------

## Git 基础概念

### 什么是Git？

Git是一个分布式版本控制系统，用于跟踪文件的变化，协调多人协作开发。

### 核心概念

#### 三个区域

- **工作区（Working Directory）**：你实际操作文件的目录
- **暂存区（Staging Area/Index）**：临时存储即将提交的文件快照
- **版本库（Repository）**：Git存储项目历史记录的地方

#### 三种状态

- **已修改（Modified）**：修改了文件但未暂存
- **已暂存（Staged）**：修改的文件已标记到下次提交中
- **已提交（Committed）**：数据已安全存储在本地版本库

#### 文件状态

- **未跟踪（Untracked）**：新文件，Git未跟踪
- **已跟踪（Tracked）**：Git正在跟踪的文件
  - 未修改（Unmodified）
  - 已修改（Modified）
  - 已暂存（Staged）

------

## 安装与配置

### 安装Git

#### Windows

```bash
# 下载Git for Windows
# https://git-scm.com/download/win
```

#### macOS

```bash
# 使用Homebrew
brew install git

# 或使用Xcode命令行工具
xcode-select --install
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install git

# CentOS/RHEL/Fedora
sudo yum install git
# 或
sudo dnf install git

# Arch Linux
sudo pacman -S git
```

### 初始配置

#### 用户信息配置

```bash
# 设置全局用户名
git config --global user.name "Your Name"

# 设置全局邮箱
git config --global user.email "your.email@example.com"

# 查看配置信息
git config --list

# 查看特定配置
git config user.name
```

#### 配置级别

```bash
# 系统级别（所有用户）
git config --system

# 全局级别（当前用户）
git config --global

# 仓库级别（当前仓库）
git config --local
```

#### 常用配置

```bash
# 设置默认编辑器
git config --global core.editor "vim"
# 或
git config --global core.editor "code --wait"  # VS Code

# 设置默认分支名
git config --global init.defaultBranch main

# 配置命令别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"

# 设置换行符处理
# Windows
git config --global core.autocrlf true
# macOS/Linux
git config --global core.autocrlf input

# 设置中文显示
git config --global core.quotepath false

# 设置代理（如需要）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

------

## 基础操作

### 初始化仓库

```bash
# 在当前目录初始化Git仓库
git init

# 创建新目录并初始化
git init project-name

# 克隆远程仓库
git clone <repository-url>

# 克隆到指定目录
git clone <repository-url> <directory-name>

# 克隆特定分支
git clone -b <branch-name> <repository-url>

# 浅克隆（只克隆最近的历史）
git clone --depth=1 <repository-url>
```

### 查看状态

```bash
# 查看当前状态
git status

# 简洁输出
git status -s
# 或
git status --short
```

### 添加文件到暂存区

```bash
# 添加指定文件
git add <file1> <file2>

# 添加所有修改的文件
git add .
# 或
git add -A
# 或
git add --all

# 添加所有已跟踪的文件（不包括新文件）
git add -u

# 交互式添加
git add -i

# 添加文件的部分更改
git add -p <file>
```

### 提交更改

```bash
# 提交暂存区的文件
git commit -m "commit message"

# 提交所有已修改的文件（跳过暂存区）
git commit -am "commit message"

# 详细的提交信息（打开编辑器）
git commit

# 修改最后一次提交
git commit --amend

# 修改最后一次提交但不修改提交信息
git commit --amend --no-edit

# 空提交
git commit --allow-empty -m "empty commit"
```

### 查看差异

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与版本库的差异
git diff --staged
# 或
git diff --cached

# 查看工作区与版本库的差异
git diff HEAD

# 查看两个提交之间的差异
git diff <commit1> <commit2>

# 查看特定文件的差异
git diff <file>

# 只显示文件名
git diff --name-only

# 显示简略统计信息
git diff --stat
```

### 查看日志

```bash
# 查看提交历史
git log

# 单行显示
git log --oneline

# 图形化显示
git log --graph

# 显示所有分支
git log --all

# 限制显示数量
git log -n 5
# 或
git log -5

# 显示详细改动
git log -p

# 显示简略统计
git log --stat

# 按作者筛选
git log --author="name"

# 按日期筛选
git log --since="2024-01-01"
git log --until="2024-12-31"
git log --after="2 weeks ago"

# 按提交信息筛选
git log --grep="keyword"

# 自定义格式
git log --pretty=format:"%h - %an, %ar : %s"

# 查看文件的修改历史
git log --follow <file>

# 查看某个文件的每一行是谁修改的
git blame <file>
```

### 移动和删除文件

```bash
# 删除文件
git rm <file>

# 删除文件但保留在工作区
git rm --cached <file>

# 强制删除
git rm -f <file>

# 移动/重命名文件
git mv <old-name> <new-name>
```

------

## 分支管理

### 分支基础操作

```bash
# 查看所有分支
git branch

# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 创建新分支
git branch <branch-name>

# 切换分支
git checkout <branch-name>
# 或（Git 2.23+）
git switch <branch-name>

# 创建并切换分支
git checkout -b <branch-name>
# 或（Git 2.23+）
git switch -c <branch-name>

# 基于远程分支创建本地分支
git checkout -b <branch-name> origin/<remote-branch>

# 删除本地分支
git branch -d <branch-name>

# 强制删除本地分支
git branch -D <branch-name>

# 删除远程分支
git push origin --delete <branch-name>
# 或
git push origin :<branch-name>

# 重命名分支
git branch -m <old-name> <new-name>

# 查看分支最后一次提交
git branch -v

# 查看已合并的分支
git branch --merged

# 查看未合并的分支
git branch --no-merged
```

### 合并分支

```bash
# 合并指定分支到当前分支
git merge <branch-name>

# 快进合并（默认）
git merge <branch-name>

# 禁用快进合并（创建合并提交）
git merge --no-ff <branch-name>

# 只允许快进合并
git merge --ff-only <branch-name>

# 合并但不自动提交
git merge --no-commit <branch-name>

# 中止合并
git merge --abort

# 继续合并
git merge --continue
```

### 变基（Rebase）

```bash
# 将当前分支变基到指定分支
git rebase <branch-name>

# 交互式变基
git rebase -i HEAD~3

# 继续变基
git rebase --continue

# 跳过当前提交
git rebase --skip

# 中止变基
git rebase --abort

# 变基到远程分支
git rebase origin/main
```

### Cherry-pick

```bash
# 选择某个提交应用到当前分支
git cherry-pick <commit-hash>

# 选择多个提交
git cherry-pick <commit1> <commit2>

# 选择范围
git cherry-pick <start-commit>..<end-commit>

# 不自动提交
git cherry-pick -n <commit-hash>
```

------

## 远程仓库操作

### 远程仓库管理

```bash
# 查看远程仓库
git remote

# 查看远程仓库详细信息
git remote -v

# 添加远程仓库
git remote add <remote-name> <repository-url>

# 删除远程仓库
git remote remove <remote-name>
# 或
git remote rm <remote-name>

# 重命名远程仓库
git remote rename <old-name> <new-name>

# 查看远程仓库信息
git remote show <remote-name>

# 修改远程仓库URL
git remote set-url <remote-name> <new-url>
```

### 推送和拉取

```bash
# 推送到远程仓库
git push <remote> <branch>

# 推送并设置上游分支
git push -u origin <branch>
# 或
git push --set-upstream origin <branch>

# 推送所有分支
git push --all

# 推送标签
git push --tags

# 强制推送（危险操作）
git push -f
# 或
git push --force

# 安全的强制推送
git push --force-with-lease

# 删除远程分支
git push origin --delete <branch-name>

# 拉取远程更改
git pull

# 拉取指定远程分支
git pull <remote> <branch>

# 使用rebase而不是merge
git pull --rebase

# 获取远程更改但不合并
git fetch

# 获取所有远程
git fetch --all

# 获取并删除远程已删除的分支
git fetch --prune
# 或
git fetch -p
```

### 跟踪分支

```bash
# 设置上游分支
git branch --set-upstream-to=origin/<branch>
# 或
git branch -u origin/<branch>

# 查看跟踪关系
git branch -vv

# 推送并设置跟踪
git push -u origin <branch>
```

------

## 高级操作

### Stash（暂存工作区）

```bash
# 暂存当前更改
git stash

# 暂存包括未跟踪文件
git stash -u
# 或
git stash --include-untracked

# 暂存并添加说明
git stash save "message"
# 或（新版本）
git stash push -m "message"

# 查看暂存列表
git stash list

# 应用最近的暂存
git stash apply

# 应用指定暂存
git stash apply stash@{n}

# 应用并删除暂存
git stash pop

# 删除暂存
git stash drop stash@{n}

# 清空所有暂存
git stash clear

# 查看暂存内容
git stash show
git stash show -p  # 详细内容

# 从暂存创建分支
git stash branch <branch-name>
```

### 子模块（Submodule）

```bash
# 添加子模块
git submodule add <repository-url> <path>

# 初始化子模块
git submodule init

# 更新子模块
git submodule update

# 初始化并更新子模块
git submodule update --init

# 递归更新子模块
git submodule update --init --recursive

# 克隆包含子模块的仓库
git clone --recursive <repository-url>

# 删除子模块
git submodule deinit <path>
git rm <path>
rm -rf .git/modules/<path>
```

### 清理和优化

```bash
# 清理未跟踪文件
git clean -n  # 查看将要删除的文件
git clean -f  # 删除未跟踪文件
git clean -fd  # 删除未跟踪文件和目录
git clean -fx  # 删除所有未跟踪文件（包括.gitignore中的）

# 优化仓库
git gc

# 压缩仓库历史
git gc --aggressive

# 验证仓库完整性
git fsck

# 清理无用的远程跟踪分支
git remote prune origin
```

### Reflog（引用日志）

```bash
# 查看引用日志
git reflog

# 查看特定分支的reflog
git reflog show <branch>

# 恢复到某个状态
git reset --hard HEAD@{n}

# 查看详细的reflog
git reflog show --all
```

### Bisect（二分查找）

```bash
# 开始二分查找
git bisect start

# 标记当前版本为坏
git bisect bad

# 标记某个版本为好
git bisect good <commit>

# 标记当前版本为好
git bisect good

# 跳过当前版本
git bisect skip

# 结束二分查找
git bisect reset

# 自动化二分查找
git bisect start
git bisect bad HEAD
git bisect good <commit>
git bisect run <test-script>
```

------

## 撤销与回退

### 撤销工作区修改

```bash
# 撤销单个文件的修改
git checkout -- <file>
# 或（Git 2.23+）
git restore <file>

# 撤销所有文件的修改
git checkout -- .
# 或
git restore .

# 从暂存区恢复到工作区
git restore --staged <file>
```

### 重置（Reset）

```bash
# 软重置（保留工作区和暂存区）
git reset --soft <commit>

# 混合重置（保留工作区，清空暂存区）- 默认
git reset --mixed <commit>
# 或
git reset <commit>

# 硬重置（清空工作区和暂存区）
git reset --hard <commit>

# 重置到上一个提交
git reset HEAD~1

# 重置单个文件
git reset HEAD <file>

# 重置到远程分支状态
git reset --hard origin/<branch>
```

### 撤销提交（Revert）

```bash
# 撤销某个提交（创建新提交）
git revert <commit>

# 撤销多个提交
git revert <commit1> <commit2>

# 撤销范围
git revert <start-commit>..<end-commit>

# 撤销但不自动提交
git revert -n <commit>
# 或
git revert --no-commit <commit>

# 继续撤销操作
git revert --continue

# 中止撤销操作
git revert --abort
```

------

## 标签管理

### 创建标签

```bash
# 创建轻量标签
git tag <tag-name>

# 创建附注标签
git tag -a <tag-name> -m "message"

# 给特定提交打标签
git tag <tag-name> <commit>

# 创建签名标签（需要GPG）
git tag -s <tag-name> -m "message"
```

### 查看标签

```bash
# 列出所有标签
git tag

# 列出匹配的标签
git tag -l "v1.*"

# 查看标签信息
git show <tag-name>

# 验证签名标签
git tag -v <tag-name>
```

### 操作标签

```bash
# 删除本地标签
git tag -d <tag-name>

# 删除远程标签
git push origin --delete <tag-name>
# 或
git push origin :refs/tags/<tag-name>

# 推送单个标签
git push origin <tag-name>

# 推送所有标签
git push origin --tags

# 拉取远程标签
git fetch --tags

# 检出标签
git checkout <tag-name>

# 基于标签创建分支
git checkout -b <branch-name> <tag-name>
```

------

## Git工作流

### Git Flow

```bash
# 主要分支
- main/master: 生产环境分支
- develop: 开发分支

# 支持分支
- feature/*: 功能分支
- release/*: 发布分支
- hotfix/*: 紧急修复分支

# 典型流程
# 1. 创建功能分支
git checkout -b feature/new-feature develop

# 2. 开发完成后合并到develop
git checkout develop
git merge --no-ff feature/new-feature

# 3. 创建发布分支
git checkout -b release/1.0.0 develop

# 4. 发布完成后合并到main和develop
git checkout main
git merge --no-ff release/1.0.0
git checkout develop
git merge --no-ff release/1.0.0

# 5. 紧急修复
git checkout -b hotfix/fix-bug main
# 修复后
git checkout main
git merge --no-ff hotfix/fix-bug
git checkout develop
git merge --no-ff hotfix/fix-bug
```

### GitHub Flow

```bash
# 简化的工作流
1. main分支始终可部署
2. 从main创建功能分支
3. 提交代码到功能分支
4. 创建Pull Request
5. 代码审查和测试
6. 合并到main
7. 部署

# 示例
git checkout -b feature/new-feature
# 开发...
git push -u origin feature/new-feature
# 创建PR，审查，合并
```

### GitLab Flow

```bash
# 环境分支策略
- main: 主开发分支
- pre-production: 预生产分支
- production: 生产分支

# 发布分支策略
- main: 主开发分支
- stable: 稳定版本分支
- release-*: 各版本发布分支
```

------

## 问题解决

### 解决合并冲突

```bash
# 1. 尝试合并
git merge <branch>

# 2. 查看冲突文件
git status

# 3. 手动编辑冲突文件
# 查找 <<<<<<< HEAD, =======, >>>>>>> 标记

# 4. 标记为已解决
git add <resolved-file>

# 5. 完成合并
git commit

# 或中止合并
git merge --abort
```

### 常见问题解决

#### 修改已推送的提交

```bash
# 修改最后一次提交（未推送）
git commit --amend

# 修改已推送的提交（需要强制推送）
git commit --amend
git push --force-with-lease
```

#### 删除敏感信息

```bash
# 使用filter-branch（已废弃）
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch <file>" \
  --prune-empty --tag-name-filter cat -- --all

# 使用BFG Repo-Cleaner（推荐）
# 1. 安装BFG
# 2. 运行
bfg --delete-files <file-name>
bfg --replace-text passwords.txt

# 强制推送所有分支和标签
git push --force --all
git push --force --tags
```

#### 恢复误删的分支

```bash
# 查找删除的分支
git reflog

# 恢复分支
git checkout -b <branch-name> <commit-hash>
```

#### 修改历史提交信息

```bash
# 交互式rebase
git rebase -i HEAD~n

# 将要修改的提交从pick改为reword
# 保存后会打开编辑器修改提交信息
```

------

## 最佳实践

### 提交信息规范

#### Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型（type）**：

- feat: 新功能
- fix: 修复bug
- docs: 文档更新
- style: 代码格式（不影响功能）
- refactor: 重构
- perf: 性能优化
- test: 测试相关
- build: 构建系统或外部依赖
- ci: CI/CD配置
- chore: 其他修改

**示例**：

```
feat(auth): add OAuth2 authentication

Implemented OAuth2 authentication flow with Google provider.
Added token refresh mechanism and session management.

Closes #123
```

### .gitignore最佳实践

```gitignore
# 操作系统文件
.DS_Store
Thumbs.db
*.swp
*~

# IDE文件
.vscode/
.idea/
*.sublime-*

# 依赖目录
node_modules/
vendor/
venv/
__pycache__/

# 构建输出
dist/
build/
out/
*.exe
*.dll
*.so

# 日志文件
*.log
logs/

# 环境变量
.env
.env.local
.env.*.local

# 敏感信息
*.key
*.pem
secrets/
```

### 分支命名规范

```bash
# 功能分支
feature/user-authentication
feature/payment-integration

# 修复分支
bugfix/login-error
hotfix/security-patch

# 发布分支
release/v1.0.0
release/2024-01-sprint

# 个人分支
dev/john/experimental-feature
```

### Git Hooks

```bash
# 本地hooks位置
.git/hooks/

# 常用hooks
- pre-commit: 提交前检查
- commit-msg: 检查提交信息
- pre-push: 推送前检查
- post-merge: 合并后操作

# 示例：pre-commit hook
#!/bin/sh
# .git/hooks/pre-commit

# 运行测试
npm test

# 检查代码风格
npm run lint

# 如果失败则阻止提交
if [ $? -ne 0 ]; then
  echo "Pre-commit checks failed"
  exit 1
fi
```

### 安全建议

1. **不要提交敏感信息**

   - 密码、密钥、令牌
   - 使用环境变量或密钥管理服务

2. **定期备份**

   - 使用多个远程仓库
   - 定期推送到远程

3. **保护主分支**

   - 启用分支保护规则
   - 要求代码审查
   - 要求状态检查

4. **签名提交**

   ```bash
   # 配置GPG签名
   git config --global user.signingkey <key-id>
   git config --global commit.gpgsign true
   ```

------

## Git内部原理

### Git对象类型

1. **Blob对象**：文件内容
2. **Tree对象**：目录结构
3. **Commit对象**：提交信息
4. **Tag对象**：标签信息

### 查看Git对象

```bash
# 查看对象类型
git cat-file -t <object-hash>

# 查看对象内容
git cat-file -p <object-hash>

# 查看对象大小
git cat-file -s <object-hash>

# 列出所有对象
find .git/objects -type f
```

### HEAD和引用

```bash
# HEAD：指向当前分支的引用
cat .git/HEAD

# 分支引用
cat .git/refs/heads/<branch-name>

# 远程引用
cat .git/refs/remotes/origin/<branch-name>

# 标签引用
cat .git/refs/tags/<tag-name>
```

### 包文件和松散对象

```bash
# 手动触发打包
git gc

# 解包对象
git unpack-objects < .git/objects/pack/*.pack

# 验证包文件
git verify-pack -v .git/objects/pack/*.idx
```

### 配置文件位置

```bash
# 系统配置
/etc/gitconfig

# 用户配置
~/.gitconfig
~/.config/git/config

# 仓库配置
.git/config
```

------

## 常用命令速查表

### 基础命令

| 命令                  | 说明             |
| --------------------- | ---------------- |
| `git init`            | 初始化仓库       |
| `git clone <url>`     | 克隆仓库         |
| `git add <file>`      | 添加文件到暂存区 |
| `git commit -m "msg"` | 提交更改         |
| `git status`          | 查看状态         |
| `git log`             | 查看日志         |
| `git diff`            | 查看差异         |

### 分支命令

| 命令                   | 说明     |
| ---------------------- | -------- |
| `git branch`           | 列出分支 |
| `git branch <name>`    | 创建分支 |
| `git checkout <name>`  | 切换分支 |
| `git merge <branch>`   | 合并分支 |
| `git branch -d <name>` | 删除分支 |

### 远程命令

| 命令            | 说明         |
| --------------- | ------------ |
| `git remote -v` | 查看远程仓库 |
| `git fetch`     | 获取远程更新 |
| `git pull`      | 拉取并合并   |
| `git push`      | 推送更改     |

### 撤销命令

| 命令                          | 说明           |
| ----------------------------- | -------------- |
| `git restore <file>`          | 撤销工作区修改 |
| `git restore --staged <file>` | 撤销暂存       |
| `git reset <commit>`          | 重置到指定提交 |
| `git revert <commit>`         | 撤销某个提交   |

