# Shell
Shell 是操作系统内核和用户之间的 “翻译官”—— 你在终端输入的命令（比如 `ls`、`cd`），需要 Shell 解析后交给内核执行，再把结果返回给你。

`sh` 和 `bash` 都是具体的 Shell 程序，只是诞生背景和功能不同。
## sh
sh是最初标准的shell。
- **全称**：Bourne Shell（以开发者 Stephen Bourne 命名），诞生于 1979 年的 Unix 系统。
- **定位**：Unix 系统的**默认标准 Shell**，语法简单、功能基础，是所有 Shell 的 “鼻祖” 之一。
- **特点**：仅支持最核心的 Shell 语法（比如条件判断 `if`、循环 `for`），没有扩展功能，兼容性极强（所有类 Unix 系统都支持）。
- **存在形式**：在现代系统中，`/bin/sh` 通常不是独立程序，而是**指向其他 Shell 的软链接**（比如指向 `bash`、`dash` 等）。
## bash
bash是sh的增强版（最主流）。
- **全称**：Bourne-Again Shell（“Bourne 重生版”），由 GNU 项目开发，1989 年发布。
- **定位**：`sh` 的**超集**—— 完全兼容 `sh` 的所有语法，同时新增了大量实用功能。
- **核心增强功能**（对比 `sh`）：
    
    - 命令历史记录（按 `↑` 回看之前输入的命令）；
    - 命令补全（按 `Tab` 自动补全命令 / 文件名）；
    - 通配符增强（比如 `{1..5}` 生成 1-5 序列）；
    - 数组、函数、正则表达式支持；
    - 更灵活的变量扩展（比如 `${var:-默认值}`）。
    
- **现状**：Linux 系统（如 Ubuntu、CentOS）的**默认 Shell** 就是 `bash`，macOS 早期也默认用 `bash`（后期切换为 `zsh`，但仍兼容）。
## 系统级目录
这类目录下的配置文件会影响所有登录系统的用户，通常需要 `root` 权限修改。

| 目录          | 作用（与 sh / 环境变量的关联）                                                                                                                                                                                                                                                                                                                                                                    |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/etc`      | 存放系统核心的 Shell 配置文件，是环境变量全局配置的核心目录：<br><br>- `/etc/profile`：全局登录 Shell 配置（所有用户登录时执行，设置全局环境变量如 `PATH`、`USER`）；<br><br>- `/etc/profile.d/`：拆分的配置文件目录（如 `java.sh`、`python.sh`，会被 `/etc/profile` 自动加载，用于给不同软件设置环境变量）；<br><br>- `/etc/bashrc`/`/etc/bash.bashrc`：全局非登录 Shell 配置（如终端交互时的别名、函数，部分系统的 `sh` 也会读取）；<br><br>- `/etc/environment`：纯环境变量配置文件（仅存 `KEY=VALUE` 格式，无 Shell 语法，系统启动时加载）。 |
| `/bin`      | 系统基础命令目录，`PATH` 环境变量默认包含（`sh` 执行的 `ls`、`cd`、`echo` 等核心命令都在这里）。                                                                                                                                                                                                                                                                                                                        |
| `/usr/bin`  | 用户常用命令目录，`PATH` 默认包含（如 `python`、`git` 等非系统核心命令）。                                                                                                                                                                                                                                                                                                                                      |
| `/usr/sbin` | 系统管理命令目录（`root` 用），`PATH` 中 `root` 用户会包含，普通用户可能无权限执行。                                                                                                                                                                                                                                                                                                                                 |
## 用户级目录
这类目录在用户主目录（`~`，等价于 `/home/用户名`）下，配置仅对当前用户生效：

| 目录              | 作用（与 sh / 环境变量的关联）                                                                                                                                                                                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `~`（/home/ 用户名） | 用户主目录，核心配置文件存放地：<br><br>- `~/.profile`：用户登录 Shell 配置（`sh`/`bash` 登录时执行，设置用户专属环境变量）；<br><br>- `~/.bash_profile`：`bash` 专属登录配置（优先级高于 `~/.profile`，若存在则优先执行）；<br><br>- `~/.bashrc`：`bash` 非登录交互配置（如终端别名、自定义函数，`sh` 模式下不读取）；<br><br>- `~/.bash_logout`：用户退出登录时执行的配置（可清理临时环境变量等）。 |
| `~/.local/bin`  | 用户自定义命令目录（新版系统推荐），可将自己的脚本 / 程序放在这里，添加到 `PATH` 后即可全局执行。                                                                                                                                                                                                                       |
## 环境变量加载顺序

1. 系统启动：先加载 `/etc/environment`（纯环境变量）；
2. 用户登录：执行 `/etc/profile` → 加载 `/etc/profile.d/*` → 执行 `~/.bash_profile`（若不存在则执行 `~/.profile`）；
3. 打开终端（非登录）：执行 `/etc/bashrc` → 执行 `~/.bashrc`；
4. `sh` 模式：仅加载 `/etc/profile`、`~/.profile`（禁用 `bash` 扩展，不读 `.bashrc`）。

---

# Linux 防火墙体系

## 整体架构

```
用户态工具（配置层）
  ┌─────────────┐  ┌──────────┐  ┌───────────┐
  │    ufw       │  │ firewalld │  │ iptables   │
  │  (Ubuntu)    │  │ (RHEL系)  │  │  (通用)     │
  └──────┬───────┘  └─────┬─────┘  └─────┬─────┘
         │                │              │
         └────────────────┼──────────────┘
                          ▼
内核态框架（执行层）
  ┌─────────────────────────────────────────┐
  │              Netfilter（内核模块）         │
  │   实际拦截和处理网络数据包的内核基础设施     │
  └─────────────────────────────────────────┘
```

> **核心关系**：Netfilter 是内核里真正干活的，iptables/ufw/firewalld 都是给用户操作 Netfilter 的前端工具。

## 一、Netfilter（内核层）

Linux 内核自带的网络包过滤框架，工作在内核空间。

**5 个钩子点（Hook Point）**，数据包经过网络协议栈时会被拦截：

```
PREROUTING → INPUT → [本机进程] → OUTPUT → POSTROUTING
                  ↘ FORWARD ↗
```

| 钩子点 | 作用 |
|---|---|
| **PREROUTING** | 数据包刚进入网卡，路由判断之前（DNAT 在这里） |
| **INPUT** | 数据包 destined 到本机进程 |
| **FORWARD** | 数据包经过本机转发到其他机器 |
| **OUTPUT** | 本机进程发出的数据包 |
| **POSTROUTING** | 数据包即将离开网卡（SNAT 在这里） |

## 二、iptables（经典工具）

直接操作 Netfilter 的用户态工具，按**表 → 链 → 规则**三级结构组织。

### 四张表（按优先级排列）

| 表 | 用途 | 常用链 |
|---|---|---|
| **raw** | 关闭连接追踪（NOTRACK） | PREROUTING, OUTPUT |
| **mangle** | 修改数据包头部（TTL、TOS 等） | 全部五链 |
| **nat** | 地址转换（DNAT/SNAT） | PREROUTING, OUTPUT, POSTROUTING |
| **filter** | 过滤放行/拒绝（最常用） | INPUT, FORWARD, OUTPUT |

### 五条链

对应 Netfilter 的 5 个钩子点：PREROUTING、INPUT、FORWARD、OUTPUT、POSTROUTING。

### 数据包匹配流程

```
数据包进入
  → raw.PREROUTING
  → mangle.PREROUTING
  → nat.PREROUTING（DNAT）
  → 路由判断：发往本机 or 转发？
      ├→ 本机：mangle.INPUT → filter.INPUT → 本机进程
      └→ 转发：mangle.FORWARD → filter.FORWARD → mangle.POSTROUTING → nat.POSTROUTING（SNAT）

本机进程发包
  → raw.OUTPUT → mangle.OUTPUT → nat.OUTPUT → filter.OUTPUT
  → mangle.POSTROUTING → nat.POSTROUTING（SNAT）
  → 数据包发出
```

### 常用命令

```bash
# 查看规则
iptables -L -n -v
iptables -t nat -L -n

# 放行 SSH（22 端口）
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 放行 HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 默认拒绝所有入站
iptables -P INPUT DROP
iptables -P FORWARD DROP

# 允许已建立的连接
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# DNAT（端口转发）：外部访问 8080 转到内网 192.168.1.100:80
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# SNAT（源地址转换）：内网通过本机上网
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE
```

## 三、nftables（iptables 的继任者）

内核 3.13+ 引入，设计目标就是取代 iptables。

**解决了 iptables 的痛点：**

| 问题 | iptables | nftables |
|---|---|---|
| 规则更新 | 逐条替换，每条都是一个单独命令 | 原子更新，批量操作 |
| 性能 | 线性匹配规则 | 支持集合、字典，O(1) 查找 |
| 语法 | 分散在多张表多条链 | 统一语法，更简洁 |
| 代码重复 | IPv4/IPv6 各一套规则 | 一套规则同时处理 v4/v6 |

```bash
# 查看规则
nft list ruleset

# 示例：放行 SSH
nft add rule ip filter input tcp dport 22 accept
```

> 主流发行版（CentOS 8+、Debian 11+、Ubuntu 22.04+）已默认使用 nftables，但保留了 iptables 兼容层（iptables 命令实际翻译成 nftables 规则）。

## 四、firewalld（RHEL/CentOS 系默认）

RHEL 系发行版的上层管理工具，核心概念是 **zone（区域）**。

```bash
# 查看当前区域
firewall-cmd --get-default-zone

# 查看当前区域的放行规则
firewall-cmd --list-all

# 放行端口
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

# 放行服务
firewall-cmd --add-service=http --permanent

# 端口转发
firewall-cmd --add-forward-port=port=8080:proto=tcp:toport=80:toaddr=192.168.1.100 --permanent
```

**Zone（区域）概念**：不同网络环境用不同规则集

| Zone | 含义 |
|---|---|
| trusted | 完全信任，放行所有 |
| home | 家庭网络，信任度高 |
| public | 公共网络，默认只放行 SSH/DHCP |
| drop | 拒绝所有入站 |

底层实际调用的也是 nftables（CentOS 8+）或 iptables（CentOS 7）。

## 五、ufw（Ubuntu 默认）

Ubuntu 的简化防火墙工具，全称 "Uncomplicated Firewall"。

```bash
# 启用/禁用
ufw enable
ufw disable

# 放行/拒绝端口
ufw allow 22/tcp
ufw allow 80/tcp
ufw deny 3306/tcp

# 放行服务
ufw allow ssh
ufw allow http

# 查看状态
ufw status verbose

# 删除规则
ufw delete allow 80/tcp

# 限定来源 IP
ufw allow from 192.168.1.0/24 to any port 22
```

底层也是操作 iptables/nftables，只是把复杂语法简化了。

## 六、工具选择建议

| 场景 | 推荐 |
|---|---|
| Ubuntu 个人服务器 | **ufw**（最简单） |
| RHEL/CentOS 生产服务器 | **firewalld**（Zone 管理方便） |
| 需要精细控制（NAT、mangle） | **iptables** 或 **nftables** |
| 新项目，长期维护 | **nftables**（未来方向） |
| 容器/云环境 | 用云平台安全组 + 轻量主机防火墙 |

> **Netfilter** 是内核引擎，**iptables/nftables** 是底层配置工具，**ufw/firewalld** 是上层简化封装。选哪个取决于你的发行版和需求复杂度。