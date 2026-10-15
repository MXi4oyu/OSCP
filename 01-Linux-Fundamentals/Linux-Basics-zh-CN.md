---
title: Linux 基础速查学习笔记
tags:
  - Linux
  - OSCP
  - CTF
---

> 用途：比赛或考试期间快速查找命令、理解参数、记录枚举结果。  
> 使用范围：仅在自有设备、授权靶场或明确允许的考试环境中使用。  
> 内容基于 Linux 基础课程笔记，并包含明确标注的 OSCP 实战补充。

Linux 基础速查学习笔记
OSCP 备考 · CTF 比赛 · Linux 枚举与提权基础
用途：比赛或考试期间快速查找命令、理解参数、记录枚举结果。
使用原则：仅在自有设备、授权靶场或明确允许的考试环境中使用。先理解命令，再执行；可能改变系统状态的命令先检查目标与影响。

## 目录 / 快速定位
- 1. 进入 Shell 后的第一轮枚举
- 2. Shell、路径与文件查看
- 3. 管道、重定向与文本处理
- 4. 文件系统、文件搜索与 find
- 5. 用户、组、密码数据库与 sudo
- 6. 权限、SUID/SGID 与计划任务
- 7. 系统信息、进程、服务、日志与存储
- 8. 网络枚举、DNS、SSH 与文件传输
- 9. Netcat 与 Socat
- 10. iptables 与连接状态
- 11. Nano / Vim 快速操作
- 12. 常见问题与易错点
- 13. OSCP / CTF Linux 枚举流程清单
- 14. 命令速查表与学习验收

## 1. 进入 Shell 后的第一轮枚举
目标：先确认当前身份、系统、目录、网络和权限，不要一上来就盲目运行大量命令。

```bash
whoami
id
pwd
hostname
uname -a
cat /etc/os-release
ip addr
ip route
ss -lntup
sudo -l
```

## 2. Shell、路径与文件查看

### 2.1 常见 Shell 与路径

### 2.2 查看文件与目录

```bash
ls
ls -l
ls -la
file suspicious.bin
cat file.txt
less file.txt
head -n 20 file.txt
tail -n 20 file.txt
tail -f /var/log/auth.log
```
速记：ls -la 看隐藏文件和权限；file 判断文件类型；less 适合翻页；head/tail 查看开头或结尾；tail -f 适合实时观察日志。

### 2.3 帮助

```bash
man ls
man passwd
command --help
man -k passwd
man 5 passwd
```
man -k 按关键词搜索手册；man 5 passwd 查看第 5 节的 passwd 文件格式说明。手册章节不同，含义也不同。

## 3. 管道、重定向与文本处理

### 3.1 标准输入输出

### 3.2 grep 与正则

```bash
grep -in "password" file.txt
grep -Rin "passwd" .
grep -n '^root:' /etc/passwd
grep -E '^(root|kali):' /etc/passwd
grep -v '^#' config.conf
grep -E '[0-9]{3}' file.txt
```
BRE/ERE：grep 默认使用 BRE；grep -E 使用 ERE。现代用法优先写 grep -E，不必使用旧式 egrep。
易错：grep -E '*sh$' 通常会因 * 没有可重复的前置表达式而报错。要匹配以 sh 结尾的文本，可用 '.*sh$'。

### 3.3 cut、awk、sed、sort、wc

```bash
cut -d ':' -f 1 /etc/passwd
awk -F ':' '{print $1, $7}' /etc/passwd
sed -n '1,20p' file.txt
sed 's/old/new/g' file.txt
sort names.txt
wc -l file.txt
cat /etc/passwd | cut -d ':' -f 1
```
注意：/etc/passwd 以冒号分隔；awk -F ':' 指定字段分隔符。

## 4. 文件系统、文件搜索与 find

### 4.1 常见目录

### 4.2 which、locate、find

### 4.3 find 参数速查

```bash
# 找脚本
find . -type f -iname '*.sh'

# 最近 30 分钟修改的文件
find . -type f -mmin -30

# root 拥有的普通文件
find / -type f -user root 2>/dev/null

# SUID 文件
find / -type f -perm -4000 2>/dev/null

# SGID 文件
find / -type f -perm -2000 2>/dev/null

# 其他用户可写的普通文件
find / -type f -perm -o=w 2>/dev/null

# 先列出名为 .svn 的目录，不要直接删除
find . -type d -name '.svn' -print
```
时间提醒：-mtime 2 通常指 48–72 小时前；-mtime -2 指最近 48 小时。不要把 -mtime 2 理解成“最近两天”。
安全提醒：find -exec rm -rf 等命令会真实删除数据。先检查路径与匹配结果；不确定时只使用 -print。

## 5. 用户、组、密码数据库与 sudo

### 5.1 重要文件
/etc/passwd 常见字段：用户名:密码占位符:UID:GID:GECOS:家目录:登录 Shell。x 通常表示密码信息位于 shadow 中。

### 5.2 密码与账号状态

```bash
passwd -S username
sudo passwd -l username
sudo chage -l username
sudo chage -E YYYY-MM-DD username
```
passwd -S 查看密码状态；passwd -l 锁定密码认证；chage -l 查看期限；chage -E 设置账号失效日期。锁定密码不一定阻止密钥认证或已有会话。

### 5.3 sudo 与用户切换

```bash
sudo -l
sudo -i
sudo -s
su - username
```
OSCP 重点：在授权靶机中检查 sudo -l 的允许命令、参数限制和运行身份。发现 sudo 权限并不自动等于提权成功，必须验证实际可执行行为。

## 6. 权限、SUID/SGID 与计划任务

### 6.1 文件权限

```bash
ls -la file
stat file
chmod 755 script.sh
chmod 600 secret.txt
chown user:group file
```

### 6.2 SUID / SGID / capabilities

```bash
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
getcap -r / 2>/dev/null
```
SUID：符合条件时，程序运行时可使用文件所有者的有效 UID。SGID：程序的有效组身份可能受文件 SGID 位影响；目录 SGID 常涉及新文件组继承。Capabilities 是补充枚举项，属于本笔记对 OSCP 的扩展。
枚举到特殊权限程序后，要检查程序用途、版本、调用方式、可控输入和文件权限；不要假设每个条目都可利用。

### 6.3 Cron 计划任务

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
crontab -l
```
检查任务的执行用户、频率、脚本路径、脚本/父目录写权限、PATH 与环境变量。重点是低权限用户能否影响高权限任务的执行结果。

## 7. 系统信息、进程、服务、日志与存储

### 7.1 系统信息

```bash
cat /etc/os-release
uname -a
uname -r
hostname
who
w
```

### 7.2 进程与服务（OSCP 补充）

```bash
ps aux
ps -ef
ss -lntup
systemctl --type=service --state=running
```
查看进程由谁运行、服务监听什么端口、服务执行文件与配置文件在哪里。内核版本仅是线索，不能单独证明存在可利用漏洞。

### 7.3 日志

```bash
ls -lah /var/log/
sudo tail -n 50 /var/log/auth.log
last
dmesg
journalctl -n 50
```
发行版不同，认证日志路径可能不同；部分系统主要通过 systemd journal 查看日志。日志可能包含敏感信息，只在授权范围内查看和处理。

### 7.4 内存与存储

```bash
free -h
df -hT
du -sh /var/log/
lsblk
mount
```
free 查看内存；df 查看文件系统空间；du 查看目录占用；lsblk 查看块设备；mount 查看挂载。

## 8. 网络枚举、DNS、SSH 与文件传输

### 8.1 网络

```bash
ip addr
ip route
ss -lntup
ping -c 2 192.0.2.10
host example.com
dig example.com
dig MX example.com
dig TXT example.com
```

### 8.2 SSH / SCP

```bash
ssh user@192.0.2.10
ssh -p 2222 user@192.0.2.10
scp local.txt user@192.0.2.10:/tmp/
scp user@192.0.2.10:/tmp/remote.txt .
```
常见配置：/etc/ssh/sshd_config；用户 SSH 文件通常在 ~/.ssh/。传输前确认主机、用户、端口和目标路径。

### 8.3 wget / curl

```bash
wget https://example.com/file.txt
curl -o file.txt https://example.com/file.txt
curl -O https://example.com/file.txt
curl -I https://example.com/
```
curl -I 查看响应头；-o 指定本地文件名；-O 通常按远端文件名保存。

### 8.4 FTP
掌握匿名登录的概念、目录查看、上传/下载、ASCII/Binary 模式，以及服务器端权限对操作的影响。FTP 可能明文传输凭据，实际环境优先采用安全传输协议。

## 9. Netcat 与 Socat

### 9.1 Netcat 基础

```bash
# 连接授权实验目标的 TCP 端口
nc -nv 192.0.2.10 80

# 在本地实验环境监听端口
nc -lvnp 4444
```
不同 nc 版本（OpenBSD、传统 netcat、ncat）参数可能不同。先用 nc -h 查看本机版本和支持选项。

### 9.2 Socat 基础

```bash
# 终端标准输入输出连接 TCP 服务
socat - TCP4:192.0.2.10:80

# 本地监听并将接收内容显示到终端
socat TCP4-LISTEN:9000 STDOUT
```
语法模型：socat [选项] 地址1 地址2。TCP4:IP:PORT 是客户端地址；TCP4-LISTEN:PORT 是监听端点；STDOUT 是标准输出；- 常用于标准输入/输出端点。
文件传输提醒：file: 地址的读写方向与打开选项有关；create 仅表示允许创建文件，不代表自动发送文件内容。先在自有实验环境验证方向和文件模式。

## 10. iptables 与连接状态

### 10.1 链与规则

```bash
sudo iptables -L -n -v --line-numbers
sudo iptables -S
sudo iptables -I INPUT 1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -I INPUT 2 -m conntrack --ctstate INVALID -j DROP
sudo iptables -I INPUT 1 -p icmp --icmp-type echo-request -j DROP
```
规则顺序很重要；-I 会改变顺序。修改防火墙可能中断 SSH 或其他连接，先查看现有规则并确保有恢复途径。iptables 规则通常不自动持久化，持久化方式依发行版而异。
禁止 IPv4 ping 示例仅匹配 ICMP echo-request；不等于禁用所有 ICMP，也不影响 IPv6 的 ICMPv6 规则。

## 11. Nano / Vim 快速操作

### 11.1 Nano

### 11.2 Vim
编辑系统配置前，确认目标文件和权限；在 sudo 环境中编辑时，优先使用合适的管理员编辑方式。

## 12. 常见问题与易错点
- grep 搜索文件内容；find -name 搜索文件名；which 主要查找 PATH 中的命令。
- grep 默认是 BRE；grep -E 使用 ERE。表达式 * 不能单独出现在开头作为有效重复符号。
- find -mtime -2 是最近 48 小时；-mtime 2 通常是 48–72 小时前。
- 2>/dev/null 只隐藏标准错误，不会提升权限或修复错误。
- sudo -l 是检查授权命令；sudo -i 和 sudo -s 的环境行为不完全相同。
- SUID/SGID 是权限位，不是漏洞结论；必须验证程序行为和利用条件。
- SSH 服务端口开放不代表认证一定成功；TCP 443 监听也不代表服务自动提供 HTTPS。
- Netcat/Socat 参数依版本而异；文件传输必须确认数据方向、文件模式和网络连通性。
- iptables 的规则按顺序匹配；插入规则可能影响现有管理连接。
- 命令历史可能未写入磁盘、被配置为忽略某些命令，或已被清理；不要假设历史完整。

## 13. OSCP / CTF Linux 枚举流程清单
以下是可按顺序执行的检查清单。它不是自动提权脚本；每一步都要解释结果，并在授权环境中继续验证。

```bash
# 1. 身份与系统
whoami
id
hostname
uname -a
cat /etc/os-release

# 2. 路径、环境与历史
pwd
echo "$PATH"
env
history

# 3. 用户与权限
cat /etc/passwd
cat /etc/group
sudo -l
ls -la ~

# 4. 特殊权限与可写位置
find / -type f -perm -4000 2>/dev/null
find / -type f -perm -2000 2>/dev/null
find / -type f -perm -o=w 2>/dev/null
getcap -r / 2>/dev/null

# 5. 计划任务、进程、服务
cat /etc/crontab
ls -la /etc/cron.d/
ps aux
ss -lntup
systemctl --type=service --state=running

# 6. 网络与存储
ip addr
ip route
df -hT
lsblk

# 7. 日志（仅在具备权限且授权的情况下）
ls -lah /var/log/
journalctl -n 50
```
记录格式建议：命令｜关键输出｜初步假设｜下一步验证｜结论。不要只保存终端截图或复制大量无关输出。

## 14. 命令速查表与学习验收

### 自测：达到什么程度才算会？
- 不看笔记，能解释 whoami、id、pwd、uname、ip、ss 的用途。
- 能使用 find 搜索最近修改文件、root 文件、SUID 文件，并解释筛选条件。
- 能读懂 rwx 权限和 chmod 600/755，理解目录 x 权限。
- 能用 grep -E、cut、awk 提取 /etc/passwd 的指定字段。
- 能检查 sudo -l、cron、进程和监听端口，并记录发现。
- 能解释 NEW、ESTABLISHED、RELATED、INVALID，以及 iptables 规则顺序。
- 在授权靶机上完成一次枚举，能说明每条发现为什么值得进一步调查。


| 命令 | 要回答的问题 | 注意点 |
| --- | --- | --- |
| whoami / id | 当前用户是谁？UID、GID、附加组是什么？ | id 输出比 whoami 更完整。 |
| pwd / hostname | 当前目录和主机名是什么？ | 记录当前工作路径。 |
| uname -a / os-release | 内核、架构、发行版版本是什么？ | 版本号只是线索，不代表一定有可利用漏洞。 |
| ip addr / ip route | 本机有哪些地址？默认路由是什么？ | 现代系统优先使用 ip 命令。 |
| ss -lntup | 哪些 TCP/UDP 端口在监听？对应什么进程？ | 部分进程信息可能需要更高权限。 |
| sudo -l | 当前账号被允许通过 sudo 执行什么？ | 仅在授权系统中检查；不要仅凭输出就认定可提权。 |


| 命令/符号 | 含义 |
| --- | --- |
| bash / zsh / sh | 常见命令解释器；脚本语法和功能可能存在差异。 |
| pwd | 显示当前工作目录。 |
| cd /etc | 切换到绝对路径。 |
| cd .. | 进入上一级目录。 |
| cd ~ 或 cd | 进入当前用户家目录。 |
| cd - | 返回上一次所在目录（常见于 Bash）。 |
| Tab | 路径和命令自动补全。 |
| ~ | 当前用户家目录；不等于根目录 /。 |
| / | 文件系统根目录。 |


| 符号 | 含义 | 示例 |
| --- | --- | --- |
| 0 / stdin | 标准输入 | 命令读取的输入流 |
| 1 / stdout | 标准输出 | 正常结果 |
| 2 / stderr | 标准错误 | 错误与诊断信息 |
| > file | 将 stdout 覆盖写入文件 | echo hello > out.txt |
| >> file | 将 stdout 追加到文件 | echo world >> out.txt |
| 2> file | 将 stderr 写入文件 | ls /missing 2> err.txt |
| 2>/dev/null | 丢弃 stderr | find / -name x 2>/dev/null |
| \| | 将左侧 stdout 传给右侧 stdin | cat file \| grep root |


| 表达式 | 含义 | 提醒 |
| --- | --- | --- |
| ^ | 行首 | ^root |
| $ | 行尾 | bash$ |
| . | 任意单个字符 | r..t |
| * | 前一表达式重复 0 次或多次 | .*sh$ |
| [abc] | 匹配 a、b、c 中一个字符 | [0-9] 匹配数字 |
| \| | ERE 中表示“或” | grep -E 'root\|kali' |
| () | ERE 中分组 | grep -E '^(root\|kali):' |
| + | ERE 中前一表达式重复 1 次或多次 | a+ |
| ? | ERE 中前一表达式出现 0 或 1 次 | colou?r |


| 工具 | 常见用途 |
| --- | --- |
| cut | 按分隔符和字段编号提取列；适合固定分隔结构。 |
| awk | 按字段处理文本，可进行条件筛选、格式化与统计。 |
| sed | 流式筛选、替换、打印指定行。 |
| sort | 排序文本行。 |
| wc -l | 统计行数。 |


| 目录 | 用途/枚举价值 |
| --- | --- |
| /etc | 系统和服务配置。 |
| /home | 普通用户家目录。 |
| /root | root 用户家目录，通常权限受限。 |
| /tmp | 临时文件目录，检查文件权限与临时脚本。 |
| /var/log | 常见日志目录。 |
| /var | 日志、缓存和服务可变数据等。 |
| /proc | 进程与内核信息接口。 |
| /dev | 设备文件。 |
| /usr/bin、/bin | 常见可执行程序；不同发行版可能采用合并目录布局。 |
| /opt | 常见第三方应用安装位置。 |


| 工具 | 用法 | 局限/特点 |
| --- | --- | --- |
| which | which python | 在 PATH 中查找可执行命令；不用于全盘文件搜索。 |
| locate | locate sshd_config | 依赖索引数据库，可能有延迟或过时结果。 |
| find | find / -type f -name '*.conf' 2>/dev/null | 按实时遍历结果和多种条件筛选。 |


| 参数 | 含义/示例 |
| --- | --- |
| -type f / -type d | 普通文件 / 目录。 |
| -name '*.sh' | 文件名匹配，区分大小写。 |
| -iname '*.sh' | 文件名匹配，忽略大小写。 |
| -user root | 文件所有者为 root。 |
| -perm -4000 | 匹配带 SUID 权限位的文件。 |
| -perm -2000 | 匹配带 SGID 权限位的文件。 |
| -perm -o=w | 其他用户写权限位存在。 |
| -mtime -2 | 修改时间在最近 48 小时内（按 24 小时区间理解）。 |
| -mmin -30 | 最近 30 分钟内修改。 |
| -ls | 显示详细信息。 |
| -exec ... {} \; | 对每个匹配项执行命令；执行删除等操作前先用 -print 验证。 |


| 文件/命令 | 用途 |
| --- | --- |
| /etc/passwd | 用户基本信息；每行 7 个冒号分隔字段。 |
| /etc/shadow | 密码哈希/状态与密码期限；通常需要 root 权限读取。 |
| /etc/group | 组信息与组成员列表。 |
| /etc/sudoers | sudo 授权规则；编辑时使用 visudo。 |
| id / groups | 查看当前或指定用户的 UID、GID 与组成员。 |


| 权限数字 | 符号 | 含义 |
| --- | --- | --- |
| 4 | r | 读取 |
| 2 | w | 写入 |
| 1 | x | 执行；目录上的 x 还涉及遍历/进入目录 |
| 7 | rwx | 读取、写入、执行 |
| 5 | r-x | 读取、执行 |
| 0 | --- | 无权限 |


| DNS 记录 | 含义 |
| --- | --- |
| A | 域名对应 IPv4 地址。 |
| AAAA | 域名对应 IPv6 地址。 |
| NS | 权威名称服务器。 |
| MX | 邮件交换服务器。 |
| CNAME | 别名记录。 |
| PTR | 反向解析记录。 |
| TXT | 文本记录，常用于验证或策略信息。 |


| 概念 | 含义 |
| --- | --- |
| INPUT | 发往本机的数据包。 |
| OUTPUT | 由本机发出的数据包。 |
| FORWARD | 经过本机转发的数据包。 |
| ACCEPT | 允许匹配的数据包通过当前链处理。 |
| DROP | 静默丢弃。 |
| REJECT | 拒绝并通常返回错误响应。 |
| -A | 追加规则到链末尾。 |
| -I INPUT 1 | 插入规则到 INPUT 链第 1 位。 |
| -D INPUT 5 | 删除 INPUT 链第 5 条规则。 |


| 状态 | 含义 |
| --- | --- |
| NEW | 新连接或连接跟踪认为处于新建阶段的数据包。 |
| ESTABLISHED | 与已建立连接相关的数据包。 |
| RELATED | 与现有连接相关的流量。 |
| INVALID | 无法归入正常连接状态的数据包。 |
| UNTRACKED | 未被连接跟踪系统跟踪的数据包。 |


| 快捷键 | 作用 |
| --- | --- |
| Ctrl+O | 写入/保存文件。 |
| Ctrl+K | 剪切当前行。 |
| Ctrl+U | 粘贴剪切缓冲区内容。 |
| Ctrl+W | 搜索。 |
| Ctrl+X | 退出。 |
| Ctrl+\ | 替换（多数 Nano 版本）。 |
| Ctrl+_ | 跳转到行号（多数 Nano 版本）。 |


| 命令 | 作用 |
| --- | --- |
| i / a / o | 进入插入模式；在光标前/后或新行插入。 |
| Esc | 回到普通模式。 |
| dd / yy / p | 删除行 / 复制行 / 粘贴。 |
| :wq | 保存并退出。 |
| :q! | 不保存退出。 |
| /pattern | 向前搜索。 |
| :行号 | 跳转到指定行。 |


| 想做什么 | 先查哪条命令 |
| --- | --- |
| 当前用户与组 | whoami；id；groups |
| 系统与内核版本 | cat /etc/os-release；uname -a |
| 查看权限 | ls -la；stat file |
| 搜索文件名 | find / -type f -name 'name' 2>/dev/null |
| 搜索文本 | grep -Rin 'keyword' . |
| 提取 passwd 用户名 | cut -d ':' -f 1 /etc/passwd |
| 检查 sudo 权限 | sudo -l |
| 检查 SUID 文件 | find / -type f -perm -4000 2>/dev/null |
| 查看计划任务 | cat /etc/crontab；ls -la /etc/cron.d/ |
| 查看进程 | ps aux |
| 查看监听端口 | ss -lntup |
| 查看路由 | ip route |
| 查看 DNS | dig example.com；host example.com |
| 查看磁盘 | df -hT；lsblk |
| 查看日志 | ls -lah /var/log/；journalctl -n 50 |
| 查看防火墙 | sudo iptables -L -n -v --line-numbers |
