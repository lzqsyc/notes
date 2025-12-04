# Linux 指令与开发环境笔记

> 资料来源：`linux.txt` 原始速记。现已整理为章节化的 Markdown，便于检索与扩展。

## 1. 文件与重定向基础

- `echo "hello" > linux.txt`：覆盖写入文件。
- `echo -e "余伟杰\n是大帅哥\n"`：启用转义输出 (`-e`)。
- `cat < linux.txt`：使用输入重定向读取文件。
- `echo hello_world >> linux.txt`：追加写入文件。

### 1.1 管道概念

- 语法：`命令A | 命令B`
- 功能：把命令 A 的标准输出作为命令 B 的标准输入，实现「数据流」连接。
- 示例：`./semester | grep -i "last-modified"`

## 2. 文本处理工具大全

### 2.1 `grep` — 文本搜索

- 基本语法：`grep [选项] "模式" 文件`
- 常用选项：
	- `-i`：忽略大小写
	- `-n`：显示行号
	- `-H`：显示文件名
	- `-l`：仅列出匹配的文件名
	- `-E`：扩展正则
	- `-F`：固定字符串匹配
- 特殊选项：`-o`（只输出匹配内容）、`-c`（统计行数）、`-v`（反选）

### 2.2 `sed` — 流式编辑

- 语法：`sed '命令' 文件`
- 常用操作：
	- `s/原/新/`：单次替换
	- `s/原/新/g`：全局替换
	- `/模式/d`：删除匹配行
	- `-n '3,5p'`：仅打印第 3-5 行
- 示例：`sed 's/比如：/举例：/' linux.txt`

### 2.3 `awk` — 模式处理器

- 基本形式：`awk '模式 { 动作 }' 文件`
- 常用内置变量：
	- `$0`：整行
	- `$1`、`$2`…：按列访问字段（默认空白分隔）
	- `NF`：字段数量
	- `NR`：当前行号
	- `FS`/`OFS`：输入/输出分隔符
- 示例：
	- `awk '{print $1,$2,$3,$4}' access.log`
	- `awk '$9==200 {print $0}' access.log`
	- `awk '$6 ~ /^GET/ {print $0}' access.log`

### 2.4 `cut` — 按列剪切

- 语法：`cut [选项] 文件`
- 选项：
	- `-d '分隔符'`：指定分隔符
	- `-f1-5`：取第 1-5 列
	- `-c1-15`：按字符位置截取
	- `-b1-3`：按字节截取
- 示例：`cut -d' ' -f1-5 access.log`

### 2.5 `wc` — 统计工具

- `wc -c`：字节数
- `wc -m`：字符数
- `wc -l`：行数
- `wc -L`：最长行长度
- `wc -w`：单词数

### 2.6 `uniq` — 去重统计

- 默认对相邻行去重，可搭配 `sort`。
- 常用选项：
	- `-c`：统计重复次数
	- `-D`：只显示重复行
	- `-u`：仅显示唯一行
	- `-f n`：忽略前 n 个字段
	- `-s n`：忽略前 n 个字符
	- `-w n`：限定对照宽度

## 3. 权限与历史命令

### 3.1 `chmod` 权限管理

- 用户类别：`u`（所有者）、`g`（所属组）、`o`（其他）。
- 权限位：`r`=4、`w`=2、`x`=1。
- 符号模式示例：`chmod u+x test.c` → `-rwx-r--r--`
- 八进制模式示例：`chmod 744 test.c`
- 递归：`chmod -R 755 dir`

### 3.2 历史命令检索

- 上下方向键：翻阅历史命令
- `history`：查看历史；可与 `grep` 结合筛选

## 4. 文件查找与批处理

### 4.1 `find` — 递归查找

- 基本语法：`find 路径 [条件] [动作]`
- 常用条件：
	- `-name/-iname`：按文件名（大小写敏感/忽略）
	- `-type f|d|l|b|c|p`：按类型
	- `-size +10M`：按大小
	- `-mtime -1` / `-mmin -30`：按修改时间
	- `-atime` / `-ctime`：访问/状态时间
	- `-perm 664` 或 `-perm /u=rw`
- 动作：
	- `-exec 命令 {} \;`
	- `-exec 命令 {} +`
	- `-ok`：执行前确认
	- `-delete`：直接删除

### 4.2 `xargs` — 架起参数桥梁

- 解决命令不接受标准输入的问题。
- 常见组合：`find . -name "*.tmp" | xargs rm`
- 重要选项：
	- `-I {}`：自定义占位符
	- `-n num`：每次传递的参数数
	- `-t`：执行前回显命令
	- `-p`：执行前询问确认
	- `-0` / `--null`：配合 `find ... -print0`，处理含空格文件名

## 5. 输入读取与 Shell 行为

### 5.1 `read` 命令

- 默认分隔符由 `IFS` 控制（初始为空格、Tab、换行）。
- 可在命令前设置分隔符：`IFS=',' read a b`。
- 字段分配：变量不足时，最后一个变量接收剩余内容。

### 5.2 正则表达式元字符

| 元字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `.` | 匹配任意单字符（不含换行） | `a.c` → `abc` |
| `*` | 前项重复 0 次或多次 | `ba*` → `b`、`baaa` |
| `+` | 前项重复 1 次或多次 | `ba+` |
| `?` | 前项重复 0 次或 1 次 | `colou?r` |
| `^` | 行首 | `^sed` |
| `$` | 行尾 | `sed$` |
| `[]` | 字符集合 | `[abc]` |
| `[^]` | 非集合字符 | `[^0-9]` |
| `()` | 分组、捕获 | `(ab)+` |
| `|` | 逻辑或 | `cat|dog` |
| `\` | 转义 | `\.` 匹配 `.` |

### 5.3 引号与转义策略

- 单引号 `'...'`：完全保留字面含义。
- 双引号 `"..."`：保留大部分字面含义，但允许 `$`、`` ` ``、`\`。
- 反斜杠 `\`：转义下一个字符。
- 正则表达式建议使用单引号，若需要变量展开使用双引号。

### 5.4 Shell 解析流程

1. 分词（Word Splitting）
2. 展开（Expansion）：变量、命令、算术、通配符、波浪线
3. 引号移除（Quote Removal）：展开后去掉引号

## 6. 磁盘与文件系统

### 6.1 `df` — 查看文件系统使用情况

- 常用选项：`-h`（易读单位）、`-T`（显示类型）、`-t`（过滤类型）、`-i`（inode）、`-H`（1000 基）、`-k`、`-a`

### 6.2 `du` — 目录/文件大小

- 常用选项：`-a`、`-h`、`-s`、`-S`、`-k`、`-m`

### 6.3 分区与校验

- `fdisk -l`：查看分区表
- `mkfs -t <文件系统> /dev/sdXn`：格式化
- `fsck -t <类型> -ACay /dev/sdXn`：文件系统检查
- `mount` / `umount`：挂载与卸载

## 7. 图形与模拟工具

### 7.1 Verilator + GTKWave 安装（Ubuntu）

1. 安装依赖：
	 ```bash
	 sudo apt update
	 sudo apt install build-essential git perl python3 make autoconf g++ flex bison ccache \
			 libgoogle-perftools-dev numactl perl-doc libfl-dev zlib1g zlib1g-dev
	 ```
2. 获取源码并编译：
	 ```bash
	 git clone https://github.com/verilator/verilator
	 cd verilator
	 git checkout stable
	 autoconf
	 ./configure
	 make -j"$(nproc)"
	 sudo make install  # 若提示 help2man 缺失：sudo apt install help2man
	 ```

### 7.2 Logisim 安装

1. 确认 Java ≥ 8：`java --version`
2. 安装 JRE/JDK：
	 ```bash
	 sudo apt update
	 sudo apt install default-jre default-jdk
	 ```
3. 将下载的 `logisim-*.jar` 放入 `/opt/logisim`
4. 赋予执行权限：`sudo chmod 755 /opt/logisim/logisim-*.jar`
5. （可选）使用具备 root 权限的编辑器创建脚本，例如：
	```bash
	sudo code /usr/local/bin/logisim --no-sandbox --user-data-dir=/root/.vscode
	# 或者使用 sudo nano /usr/local/bin/logisim
	```
6. 创建快捷脚本 `/usr/local/bin/logisim`：
	 ```bash
	 #!/bin/bash
	 java -jar /opt/logisim/logisim-generic-2.7.1.jar "$@"
	 ```
7. 授权：`sudo chmod +x /usr/local/bin/logisim`
8. 启动：`logisim`

## 8. 开发与调试工具链

### 8.1 GCC 编译回顾

```bash
gcc -g program.c -o program
```

### 8.2 GDB 调试速查

- **启动/退出**：`gdb program`、`quit`
- **运行**：`run [args]`、`set args`、`show args`
- **断点**：`break main`、`break file:line`、`break *地址`、`info breakpoints`、`delete`、`disable/enable`
- **观察点**：`watch`、`rwatch`、`awatch`、`info watchpoints`
- **单步**：`step`、`next`、`finish`、`until`、`continue`
- **栈帧**：`backtrace`、`backtrace full`、`frame n`、`up`、`down`
- **变量/内存**：`print`、`print /x expr`、`display`、`undisplay`、`x/10x addr`、`info registers`、`info locals`、`info args`
- **修改值**：`set var x=42`、`set expr`
- **线程**：`info threads`、`thread n`、`set scheduler-locking on/off`
- **信号处理**：`info signals`、`handle <signal> <action>`
- **代码查看**：`list`、`list file:function`、`set listsize`
- **反向调试**：`record`、`record stop`、`reverse-step`、`reverse-next`、`reverse-continue`
- **核心转储**：`gdb program core`、`bt`、`info registers`
- **其他**：`set follow-fork-mode`、`set detach-on-fork`、`info proc mappings`、`shell <command>`、`source <file>`、`p &variable`

### 8.3 RISC-V 交叉编译与调试

#### 8.3.1 GCC 交叉编译

```bash
riscv64-linux-gnu-gcc -static -g source.c -o program
```

分步流程：预处理 (`-E`)、编译 (`-S`)、汇编 (`-c`)、链接。

#### 8.3.2 Clang 交叉编译

```bash
clang --target=riscv64-linux-gnu -static -g source.c -o program
```

#### 8.3.3 QEMU + GDB 调优

1. 服务端：`qemu-riscv64 -g 1234 ./program`
2. 客户端：
	 ```gdb
	 gdb-multiarch ./program
	 (gdb) set architecture riscv:rv64
	 (gdb) target remote localhost:1234
	 ```

#### 8.3.4 工具链安装选项

- 包管理器：`sudo apt install gcc-riscv64-linux-gnu`
- 源码构建：`riscv-gnu-toolchain`（`--prefix=/opt/riscv`，`make linux -j$(nproc)`）
- 运行动态链接程序：`qemu-riscv64 -L /usr/riscv64-linux-gnu ./program`
- 设置环境变量：
	```bash
	echo 'export QEMU_LD_PREFIX=/usr/riscv64-linux-gnu' >> ~/.bashrc
	```
- 别名示例：
	```bash
	echo "alias rv64_gcc='riscv64-unknown-linux-gnu-gcc'" >> ~/.bashrc
	echo "alias rv64_qemu='qemu-riscv64 -L /usr/riscv64-linux-gnu'" >> ~/.bashrc
	echo "alias rv64_gdb='gdb-multiarch'" >> ~/.bashrc
	```

#### 8.3.5 QEMU 安装

- 包管理器：
	```bash
	sudo apt install qemu-user qemu-user-static qemu-system-misc qemu-utils
	```
- 源码：`git clone https://git.qemu.org/git/qemu.git`，配置 `--target-list=riscv64-softmmu,riscv64-linux-user`。

### 8.4 C 语言数据类型备忘

| 类型 | 32 位 | 64 位 | 说明 |
| :--- | :--- | :--- | :--- |
| `char` | 1 B | 1 B | |
| `short` | 2 B | 2 B | |
| `int` | 4 B | 4 B | 最小 16 位 |
| `long` | 4 B | 8 B | |
| `long long` | 8 B | 8 B | |
| 指针 | 4 B | 8 B | 任意指针 |
| `float` | 4 B | 4 B | |
| `double` | 8 B | 8 B | |

- `<stdint.h>` 精确宽度整数：`int8_t`/`uint8_t`、`int16_t`/`uint16_t`、`int32_t`/`uint32_t`、`int64_t`/`uint64_t`。
- 最小宽度类型：`int_least8_t`、`int_least16_t`、`int_least32_t`、`int_least64_t`（至少对应位宽）。
- 快速整数类型：`int_fast8_t`、`int_fast16_t`、`int_fast32_t`、`int_fast64_t`（系统最快的 ≥ 指定位宽的类型）。

### 8.5 C 语言序列点注意事项

- C 标准未规定函数参数求值顺序，不同编译器结果可能不同。
- 在同一序列点内多次修改同一对象属于未定义行为（undefined behavior）。
- Clang 会给出 `multiple unsequenced modifications` 警告。
- 可通过 GDB 单步排查变量变化。

## 9. 额外提示

- Git 协作、SSH 多账户配置等内容已整理至 `git_notes.md`。
- 建议结合 `history`、`grep`、`less` 等命令形成高效工作流。

