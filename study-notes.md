# 学习笔记整理

## 1. RTL 代码相关

### Verilog 基础语法与建模
(基于 `rtl_notes.txt`)

#### 基础语法知识
1. **语言要素**：空白符、注释符、标识符、关键字、数值。
2. **数据类型**：
   - **wire (导线)**：用于 `assign` 赋值及模块端口连接。
   - **reg (寄存器)**：用于 `always`、`initial` 语句块内赋值。对应硬件为触发器或锁存器。
   - **使用原则**：看左边定类型，`assign` 用 `wire`，`always` 用 `reg`。
3. **存储器**：
   - `reg [7:0] ram [0:1023];` (声明一个 1024 x 8位的 RAM)
4. **抽象数据类型**：integer, real, string, time (无对应硬件，不可综合，用于仿真)。

#### 赋值语句：阻塞 vs 非阻塞
- **阻塞赋值 (`=`)**：
  - 顺序执行，立即计算并赋值。
  - 对应组合逻辑。
- **非阻塞赋值 (`<=`)**：
  - 并行执行，块结束后统一更新。
  - 对应时序逻辑（触发器）。
- **原则**：组合逻辑用 `=`，时序逻辑用 `<=`。

#### 运算符
- **算术**：`+ - * / %`
- **关系**：`> < >= <=`
- **相等**：`== !=` (逻辑相等), `=== !==` (全等，包括 x/z)
- **逻辑**：`! && ||`
- **按位**：`~ & | ^ ~^`
- **归约**：单目运算，缩减为 1 bit。
- **移位**：`<< >>` (逻辑), `<<< >>>` (算术)
- **条件**：`? :` (MUX)
- **拼接**：`{a, b}`, `{{n{a}}}`

#### 模块与实例化
- **端口声明**：输入通常为 `wire`，输出视驱动方式而定。
- **实例化**：
  ```verilog
  Module_Name #(.Param(Value)) Instance_Name (.Port(Signal), ...);
  ```

#### 行为级建模
- **过程语句**：`initial` (仿真), `always` (硬件)。
- **条件语句**：
  - `if-else` (优先级逻辑)。
  - `case` (多路选择器，需注意 default 避免锁存器)。
- **循环语句**：`for` (可综合), `while/repeat/forever` (仿真)。

### Verilator 与 GTKWave 环境配置
(基于 `linux.txt`)

#### Linux 下 Verilator 安装
1. **安装依赖**：
   ```bash
   sudo apt update
   sudo apt install build-essential git perl python3 make autoconf g++ flex bison ccache libgoogle-perftools-dev numactl perl-doc libfl-dev zlib1g zlib1g-dev
   ```
2. **源码编译安装**：
   ```bash
   git clone https://github.com/verilator/verilator
   cd verilator
   git checkout stable
   autoconf
   ./configure
   make -j$(nproc)
   sudo make install
   ```

---

## 2. Linux 指令相关

### 基础文件与目录操作
- **目录切换**：`cd ~`, `cd /`, `cd -` (上次目录), `pwd` (当前路径)。
- **查看文件**：`ls -l` (详细), `ls -a` (包含隐藏), `ls -lh` (易读大小)。
- **创建与删除**：
  - `mkdir -p a/b/c` (多级目录)
  - `touch file` (创建文件)
  - `rm -rf dir` (强制删除目录)
- **复制与移动**：
  - `cp -r src dest` (复制目录)
  - `mv old new` (重命名或移动)

### 文本处理与搜索
- **查看内容**：`cat`, `less`, `more`, `head`, `tail`。
- **重定向**：`>` (覆盖), `>>` (追加)。
- **管道 (`|`)**：将上一个命令输出作为下一个命令输入。
- **文本四剑客**：
  - **grep**：搜索文本。`grep -r "text" .`
  - **sed**：流编辑。`sed 's/old/new/g' file`
  - **awk**：列处理。`awk '{print $1}' file`
  - **cut**：剪切列。`cut -d' ' -f1 file`
- **查找文件 (find)**：
  - `find . -name "*.c"` (按名查找)
  - `find . -type f` (按类型)
  - `find . -mtime -1` (1天内修改)
  - `find . -name "*.tmp" -delete` (删除找到的文件)
- **xargs**：参数传递。`find . -name "*.log" | xargs rm`

### 权限管理 (chmod)
- **符号模式**：`chmod u+x file` (用户增加执行权限)
- **数字模式**：`chmod 755 file` (rwx=4+2+1)

### 磁盘管理
- **df**：磁盘使用量 (`df -h`)。
- **du**：目录大小 (`du -sh dir`)。
- **其他**：`fdisk` (分区), `mkfs` (格式化), `mount` (挂载)。

### 编译与构建 (GCC/Make)
- **GCC**：
  - `gcc -g -o out src.c` (生成调试信息)
  - `gcc -Wall` (开启警告)
- **Make**：
  - 变量：`$@` (目标), `$^` (所有依赖), `$<` (第一个依赖)。
  - 伪目标：`.PHONY: clean`

### 调试工具 (GDB)
- **启动**：`gdb ./program`
- **常用命令**：
  - `b main` (断点)
  - `r` (运行)
  - `n` (单步跳过), `s` (单步进入)
  - `p var` (打印变量)
  - `bt` (查看堆栈)
  - `x/10x addr` (查看内存)
  - `watch var` (监控变量)

### 性能分析 (Valgrind)
- **Memcheck**：检测内存泄漏、越界、未初始化使用。
- **Callgrind**：性能分析，函数调用频率。

### 交叉编译与模拟 (RISC-V)
- **工具链**：`riscv64-linux-gnu-gcc` / `clang --target=riscv64-linux-gnu`
- **QEMU**：
  - 安装：`sudo apt install qemu-user qemu-system-misc`
  - 运行：`qemu-riscv64 -L /usr/riscv64-linux-gnu ./program`
  - 调试：`qemu-riscv64 -g 1234 ./program` (配合 `gdb-multiarch`)

---

## 3. GitHub/Git 指令相关

### SSH 配置
1. **生成密钥**：`ssh-keygen -t ed25519 -C "comment"`
2. **添加代理**：`eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519`
3. **GitHub 添加**：复制 `.pub` 内容到 GitHub Settings -> SSH keys。
4. **测试**：`ssh -T git@github.com`

### 仓库基本操作
- **初始化**：`git init`
- **关联远程**：`git remote add origin git@github.com:user/repo.git`
- **提交**：`git add .`, `git commit -m "msg"`
- **推送**：`git push -u origin main`

### 分支管理
- **查看**：`git branch -a`, `git branch -v`
- **创建/切换**：`git checkout -b new_branch`
- **删除**：`git branch -d name` (本地), `git push origin --delete name` (远程)
- **重命名**：`git branch -m old new`

---

## 4. GitHub 多设备多账号协同开发指南

### 场景描述
- **设备**：3台电脑 (电脑A, 电脑B, 电脑C)
- **账号**：2个 GitHub 账号 (Account_Main, Account_Sub)
- **目标**：针对同一个项目 (Repo_Target) 进行协同开发。

### 权限配置 (前置准备)
由于是私有项目或为了规范权限，建议将 Account_Sub 添加为仓库的协作者 (Collaborator)。
1. 登录 Account_Main 的 GitHub。
2. 进入项目仓库 -> **Settings** -> **Collaborators**。
3. 点击 **Add people**，输入 Account_Sub 的用户名或邮箱邀请。
4. 登录 Account_Sub 接受邀请。

### 多账号 SSH 配置 (单台电脑使用多账号)
如果某台电脑需要同时使用两个账号，或者不同电脑使用不同账号，建议通过 SSH Config 管理。

#### 1. 生成 SSH Key
为每个账号生成独立的 Key：
```bash
# 为主账号生成
ssh-keygen -t ed25519 -C "main@email.com" -f ~/.ssh/id_ed25519_main

# 为副账号生成
ssh-keygen -t ed25519 -C "sub@email.com" -f ~/.ssh/id_ed25519_sub
```

#### 2. 配置 `~/.ssh/config`
编辑或创建 `~/.ssh/config` 文件：
```ssh
# Main Account
Host github.com-main
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_main

# Sub Account
Host github.com-sub
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_sub
```

#### 3. 仓库远程地址设置
在克隆或添加远程仓库时，使用别名：
- 主账号操作：`git clone git@github.com-main:User/Repo.git`
- 副账号操作：`git clone git@github.com-sub:User/Repo.git`

### 协同工作流 (Workflow)

#### 核心原则
**"先拉后推" (Pull before Push)**：每次开始工作前，先 `git pull`；每次推送前，也建议先 `git pull` 以避免冲突。

#### 操作演示
**场景**：电脑A (Account_Main) 更新了代码，电脑B (Account_Sub) 需要继续开发。

1.  **电脑A (Account_Main)**:
    ```bash
    git add .
    git commit -m "Feature A done"
    git push origin main
    ```

2.  **电脑B (Account_Sub)**:
    *   **开始工作前**：
        ```bash
        git pull origin main  # 同步电脑A的修改
        ```
    *   **进行开发**...
    *   **提交更改**：
        ```bash
        git add .
        git commit -m "Feature B done"
        ```
    *   **推送**：
        ```bash
        git push origin main
        ```

3.  **电脑C (Account_Main)**:
    *   **开始工作前**：
        ```bash
        git pull origin main # 同步电脑A和B的修改
        ```

### 常见问题处理
- **冲突 (Conflict)**：
    - 当 `git pull` 提示冲突时，打开冲突文件，手动保留需要的代码。
    - 修改完成后，再次 `git add` -> `git commit` -> `git push`。
- **身份标识 (Git Config)**：
    - 确保每台电脑的本地 Git 用户名和邮箱配置正确，以便区分是谁提交的代码。
    - 局部配置（推荐）：在项目目录下 `git config user.name "Sub_Name"` 和 `git config user.email "sub@email.com"`。
