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

## 4. 其他

### Logisim 安装
- **环境**：需 Java 环境 (`sudo apt install default-jre`)。
- **运行**：`java -jar logisim.jar`
- **脚本化**：创建 `/usr/local/bin/logisim` 脚本以便直接运行。

### C 语言数据类型与位运算
- **数据类型大小** (32/64位系统)：
  - `int`: 4字节
  - `long`: 4字节(32位) / 8字节(64位)
  - `pointer`: 4字节(32位) / 8字节(64位)
- **移位运算**：
  - `<<`：左移，低位补0。
  - `>>`：右移。逻辑右移补0，算术右移补符号位。
