# Git Hub

### 标准命令

```bash
# 1. 先拉取最新代码（养成好习惯）
git pull origin main

# 2. 修改文件后添加
git add .
# 或添加指定文件：git add file.md

# 3. 提交到本地
git commit -m "feat: 添加 FOC 控制笔记"

# 4. 推送到远程
git push
```

#### `commit`消息规范

```
feat:     新功能
fix:      修复 bug
docs:     文档更新
style:    格式调整（不影响逻辑）
refactor: 代码重构
chore:    构建/工具变更
```







### 工作区架构：

```
┌─────────────────────────────────────────┐
│           Git 工作区架构                 │
├─────────────────────────────────────────┤
│                                         │
│  📁 工作目录 (Working Directory)        │
│     ↓ git add                           │
│  📦 暂存区 (Staging Area)               │
│     ↓ git commit                        │
│  🗄️  本地仓库 (Local Repository)        │
│     ↓ git push                          │
│  ☁️  远程仓库 (Remote Repository)        │
│                                         │
└─────────────────────────────────────────┘
```

### 首次本地仓库推送远程仓库场景：

```bash
# 1. 初始化本地仓库
cd /f/A-Notes/TYpora_note
git init

# 2. 添加并提交文件
git add .
git commit -m "Initial commit: 技术笔记"     # 使用 -m 可以直接带有注释无需使用编辑器

# 3. 关联远程仓库（HTTPS 格式）
git remote add origin https://github.com/wenjun1250/Study_note.git

# 4. 推送（注意分支名匹配），github通常是Main
git push -u origin master:main
```

- 如果远程有文件需要合并的情况下使用以下命令：

- ```bash
  #  先关联远程仓库（HTTPS 格式）
  git remote add origin https://github.com/wenjun1250/Study_note.git
  
  # 正常情况下 Git 会拒绝合并两个没有共同祖先的分支（例如一个全新本地仓库和一个已有远程仓库）。加上此选项后，Git 允许将两个独立的分支历史合并到一起
  git pull origin main --allow-unrelated-histories
  
  ```

### 克隆远程仓库到本地场景：

```bash
# 1. 克隆仓库（自动创建同名目录）
git clone https://github.com/wenjun1250/Study_note.git

# 2. 进入项目目录
cd Study_note

# 3. 查看状态和分支
git status
git branch -a

```

- 克隆到指定目录：

- ```bash
  git clone https://github.com/wenjun1250/Study_note.git my-folder
  ```

  - 克隆到当前目录下的 **`my-folder`** 文件夹



### 日常开发推送流程：

```bash
graph TD
    A[修改文件] --> B[git add .]       
    B --> C[git commit -m "msg"]
    C --> D[git pull 先拉取]           # 拉取远程代码，查看是否有人推送代码，检查是否有冲突
    D --> E{有冲突？}
    E -->|否 | F[git push]         # 如果没有冲突，说明远程更新可以自动合并(或者手动merge两个分支)，直接 git push 将本地提交推送到远程仓库。
    E -->|是 | G[解决冲突]          # 如果产生冲突，说明同文件同位置被你和别人同时修改了，需要手动解决冲突（编辑文件，删除冲突标记）。
    G --> H[git add + commit]
    H --> F
```



### 合并开发与分支场景：

```bash
# 1. 基于 main 创建新分支
git checkout -b feature/foc-notes main

# 2. 在分支上开发
# ... 修改文件 ...
git add .
git commit -m "feat: 添加 PID 控制章节"

# 3. 切换回主分支并合并
git checkout main
git merge feature/foc-notes

# 4. 推送主分支
git push origin main

# 5. 删除已合并的分支（可选）
git branch -d feature/foc-notes
```

- **`-b`** : `--branch` 的简写,用于创建分支

- **`git checkout -b feature/foc-notes main`** 等同于下面两行命令

  - ```bash
    git branch feature/foc-notes main   # 基于 main 创建新分支
    git checkout feature/foc-notes      # 切换到新分支
    ```

    

















