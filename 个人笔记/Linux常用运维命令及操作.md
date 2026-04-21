# Linux 常用运维命令及操作

> **来源**: 公司内部培训PPT  
> **部门**: 质量控制中心  
> **汇报人**: 周温松  
> **时间**: 2026年3月24日

---

## 目录

1. [系统基础信息管理](#一系统基础信息管理)
2. [文件和目录管理](#二文件和目录管理)
3. [防火墙与安全管理](#三防火墙与安全管理)
4. [软件包管理](#四软件包管理)
5. [进程与服务管理](#五进程与服务管理)
6. [常用运维命令](#六常用运维命令组合)

---

## 一、系统基础信息管理

### 1.1 查看操作系统发行版信息

| 命令                        | 说明             |
| ------------------------- | -------------- |
| `cat /etc/os-release`     | 系统发行版详细信息      |
| `cat /etc/system-release` | 系统发行版详细信息      |
| `cat /etc/issue`          | 操作系统版本信息       |
| `hostnamectl`             | 系统主机名控制(含系统信息) |

### 1.2 查看内核版本和系统架构

| 命令 | 说明 |
|------|------|
| `uname -a` | 显示所有系统信息 |
| `uname -r` | 显示内核版本 |
| `uname -m` | 显示硬件架构 (x86_64/aarch64) |
| `uname -n` | 显示主机名 |

### 1.3 查看 CPU 信息

| 命令                  | 说明                          |
| ------------------- | --------------------------- |
| `lscpu`             | 以友好格式显示CPU架构、核心数、线程数等信息（首选） |
| `cat /proc/cpuinfo` | 查看每个CPU核心的详细参数（底层）          |
| `nproc`             | 显示可用处理器数量                   |

### 1.4 查看内存信息

| 命令                  | 说明            |
| ------------------- | ------------- |
| `free -h`           | 内存使用情况，人类可读格式 |
| `free -m`           | 以 MB 为单位显示    |
| `cat /proc/meminfo` | 详细内存信息        |
|                     |               |

#### vmstat 速查口诀

> **高 `us` 瓶颈在CPU；高 `id` 瓶颈在等待**
> **高 `wa` 是IO等待；`si/so` 持续不为0内存不够用**

| 指标            | 含义                |
| ------------- | ----------------- |
| `us` 高        | CPU瓶颈             |
| `id` 高，空闲少    | CPU等待             |
| `wa` 高        | IO等待高，不可中断睡眠进程    |
| `si/so` 持续不为0 | 内存不够用，交换分区在使用     |
| `cs` 极高       | 线程切换过多、锁争抢、上下文切换多 |

### 1.5 查看磁盘信息

| 命令 | 说明 |
|------|------|
| `df -h` | 查看磁盘分区的使用情况（人类可读，最常用） |
| `df -ih` | 查看 inode 是否满（文件数满但空间够） |
| `df -T` | 显示文件系统类型（ext4/xfs/nfs） |
| `df -a` | 显示所有 |
| `lsblk` | 查看磁盘分区的使用情况（包括磁盘、分区、LVM、只读、挂载点、类型、大小） |

**lsblk 字段说明：**

| 字段 | 说明 |
|------|------|
| NAME | 设备名（disk=硬盘，part=分区） |
| RM | 是否可移动（0=固定硬盘，1=U盘） |
| SIZE | 容量 |
| RO | 是否只读（0=可写，1=只读） |
| TYPE | disk(物理硬盘)、part(分区)、lvm(逻辑卷)、rom(光驱) |

### 1.6 查看网络信息

| 命令 | 说明 |
|------|------|
| `ip addr` | 查看网卡与 IP 地址（lo=本地回环，eth0=真实网卡） |
| `ip route` | 查看路由表 |
| `netstat -tulpn` | 查看监听端口 |
| `ss -tulpn` | 查看监听端口（更现代） |
| `ss -nltp \| grep 8080` | 查找指定端口 |
| `ping/telnet/nc` | 网络连通性测试 |

---

## 二、文件和目录管理

### 2.1 文件与目录操作

| 命令 | 功能 | 常用参数 |
|------|------|----------|
| `ls` | 列出目录内容 | `-l`(详细), `-a`(隐藏), `-h`(可读), `-t`(时间) |
| `cd` | 切换目录 | `cd -`(返回上次), `cd ~`(回家) |
| `pwd` | 显示当前路径 | - |
| `mkdir` | 创建目录 | `-p`(递归创建) |
| `rm` | 删除文件/目录 | `-r`(递归), `-f`(强制) |
| `cp` | 复制文件 | 保留属性 |
| `mv` | 移动/重命名 | - |
| `touch` | 创建空文件 | - |
| `find` | 查找文件 | `-name`, `-mtime`, `-size` |

### 2.2 find 命令详解

`find` 是 Linux 最强磁盘查找命令，支持按文件名、大小、时间、权限、所有者、类型等筛选，找到后可执行操作。

#### 按文件名查找

```bash
# 精确查找（当前目录）
find . -name "test.txt"

# 模糊查找（通配符）
find /etc -name "*.conf"          # /etc下所有.conf文件
find / -name "mysql*"             # 全盘找mysql开头的文件
```

#### 按文件类型查找

| 类型 | 说明 |
|------|------|
| `-type f` | 普通文件 |
| `-type d` | 目录 |
| `-type l` | 软链接 |
| `-type b` | 块设备 |
| `-type c` | 字符设备 |

```bash
find /home -type f               # 只查普通文件
```

#### 按文件大小查找

| 符号 | 含义 |
|------|------|
| `+` | 大于 |
| `-` | 小于 |
| 无符号 | 等于 |

```bash
find . -size +100M               # 大于100M的文件
find . -size -10k                # 小于10k
find . -size +1G -type f          # 大于1G的普通文件
```

#### 按深度查找

```bash
find /etc -maxdepth 2 -name "*.conf"   # 只查2层目录
```

#### 复合条件查找

| 操作符 | 含义 |
|--------|------|
| `-a` | 且（默认，可省略） |
| `-o` | 或 |
| `-not` | 非 |

```bash
find . -mtime -7 -a -size +100M   # 7天内且大于100M
find . -o -name "*.txt" -name "*.log"   # 查找.txt或.log
find . -not -name "*.log"          # 查找不是.log的文件
```

#### 找到文件后后续操作

| 操作 | 说明 |
|------|------|
| `-print` | 打印（默认，可省略） |
| `-delete` | 直接删除（慎用） |
| `-exec` | 执行命令（常用） |

```bash
find . -name "*.txt" -print       # 查找打印
find . -mtime +7 -size -10k -delete   # 删除7天前、小于10k的垃圾文件
find . -name "*.log" -exec ls -l {} \;   # 找到所有.log文件，查看详情
```

### 2.3 find 实战案例

```bash
# 全盘查找nginx配置文件
find / -name "nginx.conf"

# 清理服务器垃圾：删除30天前、小于100M的日志
find /var/log -name "*.log" -mtime +30 -delete

# 查找大于1G的大文件
find / -size +1G -type f

# 查找当前目录所有目录并修改权限
find . -type d -exec chmod 755 {} \;

# 奇怪文件名导致文件不可操作，通过inode查找并删除
ls -li                          # 列出当前目录所有文件，显示inode号
find . -inum 12345 -exec mv {} new_file.txt \;   # 修改文件名
find . -inum 12345 -exec rm -f {} \;              # 删除文件
find . -inum 12345 -delete                           # 或更简洁的写法
```

### 2.4 文件内容查看操作

| 命令 | 功能 |
|------|------|
| `cat file` | 查看文件内容（小文件） |
| `less file` | 分页查看 |
| `head -n 100 file` | 查看前100行 |
| `tail -n 100 file` | 查看后100行 |
| `tail -f file` | 实时跟踪文件变化 |
| `tail -100f file` | 跟踪最后100行并实时 |
| `wc -l file` | 统计行数 |
| `sed 's/old/new/g' file` | 文本替换 |
| `grep "text" file` | 搜索文本 |

### 2.5 grep 命令详解

`grep` 是 Linux/Unix 系统最常用的文本搜索工具，全称 **Global Regular Expression Print**（全局正则表达式打印）。

```bash
grep [选项] "搜索模式" 目标文件/目录
```

| 参数 | 功能说明 | 实用示例 |
|------|----------|----------|
| `-i` | 忽略大小写匹配 | `grep -i "error" log.txt` |
| `-n` | 显示匹配行的行号 | `grep -n "fail" app.log` |
| `-v` | 反向匹配（输出不匹配行） | `grep -v "DEBUG" log.txt` |
| `-c` | 仅统计匹配行数 | `grep -c "ERROR" sys.log` |
| `-r` / `-R` | 递归搜索目录下所有文件 | `grep -r "TODO" ./src/` |
| `-w` | 精确匹配完整单词 | `grep -w "user" config.conf` |
| `-A` / `-B` / `-C` | 显示匹配行后/前/前后行上下文 | `grep -C 5 "panic" app.log` |
| `-E` | 启用扩展正则（等价于 egrep） | `grep -E "error\|warn" log.txt` |

#### grep 实战案例

```bash
# 批量修改文件中的IP地址
grep 192.168.66.202 -rl * | xargs sed -i 's/192.168.66.202/192.168.66.201/g'

# 统计错误出现次数
grep -c "ERROR" /var/log/nginx/access.log

# 实时追踪日志并过滤报错
tail -f access.log | grep --color=auto "error"

# 提取访问日志中的IP地址，并统计出现次数
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" access.log | sort | uniq -c

# 查看指定进程
ps aux | grep nginx | grep -v grep

# 查看端口占用
netstat -tuln | grep 80

# 查看内存
free -h | grep Mem | awk '{print $3}'
```

---

## 三、防火墙与安全管理

### 3.1 Firewalld (动态防火墙)

Firewalld 是 iptables 的前端管理工具，通过 D-Bus 接口与守护进程通信，由守护进程将配置转换为底层的规则。

```bash
# 启动服务
systemctl start firewalld

# 查看区域
firewall-cmd --get-active-zones

# 开放端口
firewall-cmd --add-port=80/tcp --permanent

# 重载规则
firewall-cmd --reload

# 查看所有规则
firewall-cmd --list-all
```

### 3.2 Iptables vs Firewalld 对比

| 对比维度 | Iptables | Firewalld |
|----------|----------|------------|
| **设计理念** | 原生底层，表链规则，精细控制 | 上层抽象，区域服务，动态管理 |
| **生效方式** | 静态重载，重启服务，可能断连 | 动态更新，无需重启，不影响连接 |
| **配置存储** | 单文件 `/etc/sysconfig/iptables` | 多XML文件，`/usr/lib/` + `/etc/firewalld/` |
| **管理方式** | 纯命令行，复杂手动配置 | 命令行 + 图形化，服务/端口一键配置 |
| **默认行为** | 默认允许所有流量，需手动拒绝 | 默认拒绝所有流量，需手动放行 |
| **适用场景** | 老旧系统、复杂NAT、脚本化、高性能 | 现代发行版、动态网络、多接口、快速部署 |
| **守护进程** | 无，直接内核交互 | 有 firewalld.service，常驻后台 |

### 3.3 Linux 安全分层

```
┌─────────────────────────────────────┐
│           Linux 安全体系            │
├─────────────────────────────────────┤
│  网络层：Netfilter/Iptables/Firewalld │
│  （封端口、过滤IP、转发、NAT）        │
├─────────────────────────────────────┤
│  主机层：SELinux（进程权限、文件访问） │
├─────────────────────────────────────┤
│  认证层：SSH安全、sudo PAM           │
├─────────────────────────────────────┤
│  审计层：日志、auditd、监控          │
└─────────────────────────────────────┘
```

### 3.4 主机强制访问控制

#### SSH 安全

- 禁止 root 直接登录
- 修改默认端口
- 禁用密码、只用密钥
- 限制 IP 登录

#### 用户与权限管理

```bash
# 禁用Root远程登录
# 使用普通用户 + sudo 进行管理

# 强密码策略
# 设置复杂度要求并定期更换

# 严格控制sudo
# 最小授权原则，定期审计用户账号
```

#### 端口暴露最小化

```bash
# 只开业务端口：22, 80, 443 等
# 禁止 0.0.0.0 监听不必要端口
ss -tulnp
```

#### 日志与监控审计

- 启用日志审计
- 监控关键日志文件，设置日志轮转
- `/var/log/secure`、`/var/log/messages`
- 定期安全扫描（使用绿盟等工具）

---

## 四、软件包管理

Linux 中 RPM 和 DKG 是最常见的两类软件包管理工具：
- **RPM**: 应用于基于 RPM 软件包的 Linux 发行版（如 CentOS、RHEL）
- **DPKG**: 应用于基于 DEB 软件包的 Linux 发行版（如 Debian、Ubuntu）

### 4.1 RPM 安装/卸载

```bash
rpm [选项]
```

| 参数 | 功能说明 |
|------|----------|
| `-i` | 安装软件包 |
| `-U` | 升级安装软件包 |
| `-v` | 显示指令执行过程，显示附加信息 |
| `-h` | 包安装时列出哈希标记（和-v一起效果更好） |
| `--force` | 忽略软件包及文件的冲突 |
| `--nodeps` | 不验证软件包依赖 |
| `-e` | 删除(卸载)软件包 |

### 4.2 RPM 查询

```bash
# 查询已安装的软件包
rpm -qa                      # 查询系统中已安装的所有软件
rpm -qf 文件名的绝对路径      # 查询文件由哪个软件包提供
rpm -ql 软件名                # 查询该软件包提供的所有文件及安装位置
rpm -qi 软件名                # 查询已安装软件包的信息
rpm -qc 软件名                # 查询该软件提供的配置文件
```

| 参数 | 功能说明 |
|------|----------|
| `-q, --query` | 查询已经安装的软件包 |
| `-a` | 查询所有安装的软件包 |
| `-c, --configfiles` | 显示配置文件列表 |
| `-f` | 查询属于哪个软件包 |
| `-i` | 显示软件包的概要信息 |
| `-l` | 显示软件包中的文件列表 |

### 4.3 YUM 安装

```bash
# yum配置文件示例：/etc/yum.repos.d/local.repo
[base]
name=repository description
gpgcheck=1          # 是否检查RPM包数字签名（1=检测，0=不检测）
baseurl=file://...  # 本地挂载点或网络yum源
gpgkey=...          # 数字签名证书公钥文件
```

```bash
yum list                     # 列出所有可安装/已安装包
yum list installed           # 只看已安装
yum search 关键字            # 搜索软件
yum info 包名                # 查看包详情
yum install -y               # 安装并自动确认
yum localinstall xxx.rpm     # 安装本地rpm并解决依赖
yum remove                   # 卸载
yum clean all                # 清空缓存
yum makecache               # 重建元数据缓存
yum grouplist               # 查看软件组
```

### 4.4 源码安装（Source Code Installation）

源码安装：将软件的原始代码下载到服务器，通过编译、链接等步骤手动构建安装。

#### YUM/RPM vs 源码安装对比

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Yum/RPM（二进制）** | 一键安装、自动解依赖、速度快 | 版本固定（多为旧版）、自定义程度低 | 快速部署、对版本无特殊要求 |
| **源码安装** | 可自定义版本、编译参数、性能更优、功能可裁剪 | 需手动解依赖、步骤多、耗时长 | 需新版软件、需定制功能、性能优化 |

#### Nginx 源码安装步骤

```bash
# 1. 安装核心编译工具（必装）
yum install -y gcc gcc-c++

# 2. 安装Nginx专属依赖（不同软件依赖不同，需提前查文档）
yum install -y pcre-devel zlib-devel openssl-devel

# 3. 下载源码包（官网/镜像站）
cd /usr/local/src
wget https://nginx.org/download/nginx-1.24.0.tar.gz

# 4. 解压
tar -zxvf nginx-1.24.0.tar.gz
cd nginx-1.24.0

# 5. 自定义安装（--prefix指定安装根目录）
./configure \
  --prefix=/usr/local/nginx \
  --with-http_ssl_module \
  --with-http_gzip_static_module

# 6. 多线程编译（-j后接CPU核心数）
make -j4

# 7. 编译成功：无error，生成可执行二进制文件
# 编译失败：检查依赖、源码完整性、编译器版本

# 8. 安装到指定目录
make install

# 9. 验证安装
/usr/local/nginx/sbin/nginx -v

# 10. 启动
/usr/local/nginx/sbin/nginx
```

---

## 五、进程与服务管理

### 5.1 常用命令

```bash
# 找出占用80端口的进程及后台路径
ss -lntup | grep :80
pwdx pid                    # 根据pid查找应用后台路径

# 服务挂了，不知道部署在哪？
redis服务为例：
history | grep redis        # 查找历史启动命令
find / -name redis.conf     # 找不到历史时
find / -name redis-server
/usr/local/redis/bin/redis-server redis.conf

# 磁盘快满了，找超过1G的大文件
find / -type f -size +1G -exec ls -lh {} \;

# 看某个进程的实时资源占用
top -p $(pgrep nginx | head -1)

# 查看内存占用前10的进程
ps aux --sort=-%mem | head -11

# 查看CPU占用前10的进程
ps aux --sort=-%cpu | head -11

# 批量杀死进程
ps aux | grep process_name | grep -v grep | awk '{print $2}' | xargs kill -9

# 查看当前连接数
netstat -an | grep ESTABLISHED | wc -l
```

---

## 六、常用运维命令组合

| 场景 | 命令 |
|------|------|
| 查找占用端口的进程 | `ss -lntup \| grep :80` |
| 查找应用后台路径 | `pwdx pid` |
| 查找历史启动命令 | `history \| grep redis` |
| 查找配置文件 | `find / -name redis.conf` |
| 查找可执行文件 | `find / -name redis-server` |
| 查找大于1G的文件 | `find / -type f -size +1G -exec ls -lh {} \;` |
| 查看进程实时资源占用 | `top -p $(pgrep nginx \| head -1)` |
| 查看内存占用前10进程 | `ps aux --sort=-%mem \| head -11` |
| 查看CPU占用前10进程 | `ps aux --sort=-%cpu \| head -11` |
| 批量杀死进程 | `ps aux \| grep process_name \| grep -v grep \| awk '{print $2}' \| xargs kill -9` |
| 查看当前连接数 | `netstat -an \| grep ESTABLISHED \| wc -l` |

---

## 速查表

### 系统信息
```bash
cat /etc/os-release        # 发行版信息
uname -a                   # 所有系统信息
lscpu                      # CPU信息
free -h                    # 内存信息
df -h                      # 磁盘信息
lsblk                      # 块设备信息
ip addr                    # 网卡信息
```

### 文件操作
```bash
ls -la                     # 列出所有文件
find / -name "*.conf"      # 查找配置文件
grep -r "text" ./          # 递归搜索
tail -f log.txt            # 实时跟踪日志
```

### 软件安装
```bash
yum install -y 包名        # YUM安装
rpm -ivh 包名.rpm          # RPM安装
./configure && make && make install   # 源码安装
```

### 进程与端口
```bash
ss -tulpn                  # 查看端口
ps aux | grep 进程名        # 查看进程
top                        # 实时监控
kill -9 pid                # 杀死进程
```

---

*整理: AI助手 | 来源: 周温松内部培训PPT*
