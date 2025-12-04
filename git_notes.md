# Git 与 GitHub 协同开发笔记

## 1. Git 基础配置与指令

### 1.1 SSH 密钥配置
**作用**：实现本地与 GitHub 的免密安全通信。

1.  **生成密钥 (Ed25519 算法)**:
    ```bash
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
2.  **添加 SSH Agent**:
    ```bash
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519
    ```
3.  **部署到 GitHub**:
    - 复制公钥内容: `cat ~/.ssh/id_ed25519.pub`
    - GitHub -> Settings -> SSH and GPG keys -> New SSH key。
4.  **测试连接**:
    ```bash
    ssh -T git@github.com
    # 成功回显: Hi username! You've successfully authenticated...
    ```

### 1.2 仓库基本操作
- **初始化**: `git init`
- **关联远程**: `git remote add origin git@github.com:User/Repo.git`
- **查看状态**: `git status`
- **添加文件**: `git add .`
- **提交更改**: `git commit -m "message"`
- **推送代码**: `git push -u origin main` (首次), `git push` (后续)
- **拉取代码**: `git pull origin main`

### 1.3 分支管理
- **查看分支**: `git branch -a`
- **创建并切换**: `git checkout -b new_branch`
- **删除本地分支**: `git branch -d branch_name`
- **删除远程分支**: `git push origin --delete branch_name`
- **重命名分支**: `git branch -m old new`

---

## 2. GitHub 多设备多账号协同开发指南

### 2.1 场景描述
- **设备**：多台电脑 (A, B, C)
- **账号**：多个 GitHub 账号 (主账号 `Main`, 副账号 `Sub`)
- **目标**：在同一台电脑上，针对不同项目使用不同账号进行 Push/Pull。

### 2.2 权限配置 (Collaborator)
若 `Main` 账号想向 `Sub` 账号的仓库推送代码，必须先获得权限：
1. 登录 `Sub` 账号。
2. 仓库 Settings -> Collaborators -> Add people。
3. 邀请 `Main` 账号。
4. `Main` 账号接受邀请。

### 2.3 多账号 SSH 配置 (核心方案)
通过 `~/.ssh/config` 文件，根据不同的 Host 别名自动使用不同的密钥。

#### 步骤 1: 生成多对密钥
```bash
# 主账号密钥
ssh-keygen -t ed25519 -C "main@ex.com" -f ~/.ssh/id_ed25519_main

# 副账号密钥
ssh-keygen -t ed25519 -C "sub@ex.com" -f ~/.ssh/id_ed25519_sub
```

#### 步骤 2: 配置 Config 文件
编辑 `~/.ssh/config`:
```ssh
# 默认 GitHub (主账号)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_main

# 副账号 GitHub (别名 github-sub)
Host github-sub
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_sub
```

#### 步骤 3: 使用别名克隆/关联
- **主账号项目**:
  `git clone git@github.com:MainUser/Repo.git`
- **副账号项目**:
  `git clone git@github-sub:SubUser/Repo.git`
  *(注意：这里将域名 `github.com` 替换为了配置文件中的 `github-sub`)*

#### 实战案例：解决 "Permission denied"
**现象**: 推送时提示 `Permission to xxx denied to user yyy`.
**原因**: 使用了错误的密钥（身份）。
**解决**:
1. `git remote -v` 查看当前远程地址。
2. 修改远程地址，使用正确的 Host 别名：
   ```bash
   git remote set-url origin git@github-sub:SubUser/Repo.git
   ```

### 2.4 协同工作流 (Workflow)
**原则**: **Pull before Push** (先拉后推)。

1.  **电脑 A** (例如主账号):
    ```bash
    git add .
    git commit -m "Feature A done"
    git push origin main
    ```
2.  **电脑 B** (例如副账号):
    ```bash
    git pull origin main   # 开工前同步
    # ... 进行修改 ...
    git add .
    git commit -m "Feature B done"
    git pull origin main   # 推送前再次确认
    git push origin main
    ```
3.  **其他设备**：复用上述流程，始终保持“先拉后推”。

### 2.5 常见问题
- **冲突 (Conflict)**: `git pull` 报冲突 → 手动解决 → `git add` → `git commit`。
- **暂存 (Stash)**: 临时保存未提交修改 `git stash`，恢复 `git stash pop`。
- **清理远程分支**: 远程已删除、但本地仍保留 → `git remote prune origin`。
- **身份标识配置**：针对不同账号或设备，在项目目录下使用局部配置区分提交者：
  ```bash
  git config user.name "Your Name"
  git config user.email "you@example.com"
  ```

---

## 3. SSH 密钥维护清单

### 3.1 检查现有密钥

```bash
ls -al ~/.ssh
``` 

- 私钥权限需为 `600 (-rw-------)`，公钥权限 `644 (-rw-r--r--)`。

### 3.2 新建密钥（Ed25519）

```bash
ssh-keygen -t ed25519 -C "注释备注"
```

1. 默认保存路径（回车即可），也可指定新文件名。
2. 建议设置 passphrase 以增强安全性。
3. 启动代理并添加私钥：
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```
4. 复制公钥：`cat ~/.ssh/id_ed25519.pub` 上传到 GitHub。
5. 测试连接：`ssh -T git@github.com`。

> 如果使用多个 Host 别名，建议分别验证：
> ```bash
> ssh -T git@github.com
> ssh -T git@github-sub
> ```

### 3.3 清理旧密钥

1. 在 GitHub 删除旧公钥。
2. 本地删除：`rm ~/.ssh/旧私钥文件`。

---

## 4. 常见推送场景脚本

### 4.1 全新仓库从 0 到 1

1. GitHub 创建仓库 `name1`。
2. 本地执行：
   ```bash
   git init
   git remote add name2 git@github.com:lzqsyc/name1.git
   git add .
   git commit -m "initial commit"
   git branch -M name3
   git push -u name2 name3  # 将本地 name3 与远程同名分支建立跟踪
   ```

### 4.2 推送到已存在的远程分支

```bash
git init
git remote add name2 git@github.com:lzqsyc/name5.git
git add .
git commit -m "initial commit"
git branch -M name3
git push -u name2 name3:name6  # 本地 name3 → 远程 name6
git branch --set-upstream-to=name2/name6 name3
```

---

## 5. 分支与远程管理进阶

### 5.1 本地分支改名与删除

- 改名：`git branch -m <旧> <新>`（当前分支可省略旧名）
- 删除：
  - 安全删除（已合并）：`git branch -d <分支>`
  - 强制删除：`git branch -D <分支>`

### 5.2 远程别名管理

- 重命名远程：`git remote rename <旧> <新>`
- 删除远程：`git remote remove <别名>`

### 5.3 删除远程分支并解除跟踪

```bash
git push <远程别名> --delete <分支>
git checkout <本地分支>
git branch --unset-upstream
```

### 5.4 建立与调整上游关系

```bash
git branch --set-upstream-to=<远程>/<分支> <本地分支>
git push -u <远程> <本地分支>:<远程分支>
```

### 5.5 状态与引用查询

```bash
git status
git branch -v
git branch -a
git remote -v
git ls-remote <远程>
git fetch <远程>
git checkout -b 分支 远程/分支
```

---

## 6. 其他速查

- Git 与多账号协同：保持 SSH 别名与远程地址一致（`git@github-sub:user/repo.git`）。
- 若推送遇到 `Permission denied`，优先确认当前使用的 Host 别名与密钥。
- 建议习惯性 `pull` → `commit` → `pull --rebase` → `push`，减少冲突。
