# SKILL.md
id: linux-emergency-response
name: Linux Gun AI Skills
trigger_regex: (?=.*hostname\s*=\s*\S+)(?=.*port\s*=\s*\d+)(?=.*username\s*=\s*\S+)(?=.*password\s*=\s*\S+).+
description: Linux 应急响应专用工具，用户只需提供 SSH 连接信息，AI 自动引导进行全面的入侵排查并分析结果。

## 详细指令
当用户消息同时包含 hostname、port、username、password 四段时：
1. 保存连接信息用于后续 SSH 命令执行
2. 使用 expect 通过 SSH 执行应急响应命令
3. 主动询问用户想要排查的方向，或建议从基础信息开始
4. **每次执行命令后必须分析输出结果，给出专业结论**

## 验证规则
- port 必须是 1-65535 整数，否则立即返回"port 范围错误"
- 四段任一缺失，返回"参数不完整，请提供 hostname port username password"

## 输出分析要求（重要）
每次执行命令后，必须对输出进行分析并给出结论，格式如下：

### 分析报告格式
```
📋 命令: [执行的命令]
📊 分析结果:
- [发现的关键信息点1]
- [发现的关键信息点2]
...

⚠️ 可疑项:
- [可疑项1及原因]
- [可疑项2及原因]
...

✅ 正常项:
- [正常项说明]

💡 建议:
- [下一步排查建议]
- [处置建议（如有必要）]
```

### 分析重点
1. **用户与历史**: 关注 UID=0 非 root 用户、异常登录、History 敏感指令(下载/执行/删除)、空口令/弱口令
2. **进程与隐藏**: 关注高 CPU/内存占用、隐藏进程(孤儿/内存映射异常)、敏感进程名、挖矿/反弹特征
3. **网络与隧道**: 关注外连 IP、高危端口、ARP 异常、网卡混杂模式、SSH 隧道(本地/远程/动态转发)
4. **文件与权限**: 关注 /tmp 及 /root 隐藏文件、SUID/SGID、SSH 公钥/配置、24h 内变动文件、文件属性锁定
5. **服务与启动**: 关注异常系统服务、rc.local/bashrc 修改、计划任务(Crontab/Anacron)、恶意启动脚本
6. **日志与审计**: 关注 SSH 爆破(secure/btmp)、日志清除痕迹、关键服务日志(message/yum/auditd)、内核日志
7. **恶意特征**: 关注 Webshell、Rootkit(内核模块/LD_PRELOAD)、黑客工具痕迹、已知病毒/恶意软件
8. **容器与基线**: 关注 K8s 集群异常、Docker 容器逃逸风险、系统安全基线(账户/密码/权限)配置

### 威胁等级标注
- 🔴 高危: 确认存在入侵痕迹或后门
- 🟡 中危: 存在可疑项需进一步确认
- 🟢 低危/正常: 未发现明显异常

## 排查流程选择建议
连接成功后，询问用户选择使用哪个排查流程：
0. 快速全面排查（自动执行关键检查项并汇总分析）
1. 系统信息排查
2. 网络连接排查
3. 进程排查
4. 文件排查
5. 后门排查
6. 隧道检测
7. webshell排查
8. 病毒排查
9. 内存排查
10. 黑客工具排查
11. 内核排查
12. 其他排查
13. Kubernetes排查
14. 系统性能分析
15. 基线检查


## 排查流程对应的详细检查项
- 如果用户选择 `快速全面排查（自动执行关键检查项并汇总分析）`,则执行所有检查项
### 1. 系统信息排查
- IP地址
- 系统基础信息
    - 系统版本信息
    - 系统发行版本
    - 虚拟化环境检测
- 用户信息分析
    - 正在登录用户
    - 系统最后登录用户
    - 用户信息passwd文件分析
    - 检查可登录用户
    - 检查超级用户(除root外)
    - 检查克隆用户
    - 检查非系统用户
    - 检查空口令用户
    - 检查空口令且可登录用户
    - 检查口令未加密用户
    - 用户组信息group文件分析
    - 检查特权用户组(除root组外)
    - 相同GID用户组
    - 相同用户组名
- 计划任务分析
    - 系统计划任务
    - 用户计划任务
- 历史命令分析
    - 输出当前shell系统历史命令[history]
    - 输出用户历史命令[.bash_history]
    - 是否下载过脚本文件
    - 是否通过主机下载,传输过文件
    - 是否增加,删除过账号
    - 是否执行过黑客命令
    - 其他敏感命令
    - 检查系统中所有可能的历史文件路径[补充]
    - 输出系统中所有用户的历史文件[补充]
    - 输出数据库操作历史命令

### 2. 网络连接排查
- ARP 攻击分析
- 网络连接分析
- 端口信息排查
    - TCP 端口检测
    - TCP 高危端口(自定义高危端口组)
    - UDP 端口检测
    - UDP 高危端口(自定义高危端口组)
- DNS 信息排查
- 网卡工作模式
- 网络路由信息排查
- 路由转发排查
- 防火墙策略排查

### 3. 进程排查
- ps进程分析
- top进程分析
- 规则匹配敏感进程(自定义进程组)
- 异常进程检测
- 高级进程隐藏检测
    - 孤儿进程检测
    - 网络连接和进程映射
    - 进程可疑内存映射
    - 文件描述符异常进程
    - 系统调用表完整性检测
    - 进程启动时间异常检测
    - 进程环境变量异常检测

### 4. 文件排查
- 系统服务排查
    - 系统服务收集
    - 系统服务分析
        - 系统自启动服务分析
        - 系统正在运行的服务分析
    - 用户服务分析
- 敏感目录排查
    - /tmp目录
    - /root目录(隐藏文件)【隐藏文件分析】
- 特殊文件排查
    - ssh相关文件排查
        - .ssh目录排查
        - 公钥私钥排查
        - authorized_keys文件排查
        - known_hosts文件排查
        - sshd_config文件分析
            - 所有开启的配置(不带#号)
            - 检测是否允许空口令登录
            - 检测是否允许root远程登录
            - 检测ssh协议版本
            - 检测ssh版本
    - 环境变量排查
        - 环境变量文件分析
        - env命令分析
    - hosts文件排查
    - shadow文件排查
        - shadow文件权限
        - shadow文件属性
        - gshadow文件权限
        - gshadow文件属性
    - 24小时变动文件排查
    - SUID/SGID文件排查
- 日志文件分析
    - message日志分析
        - ZMODEM传输文件
        - 历史使用DNS情况
    - secure日志分析
        - 登录成功记录分析
        - 登录失败记录分析(SSH爆破)
        - SSH登录成功记录分析
        - 新增用户分析
        - 新增用户组分析
    - 计划任务日志分析(cron)
        - 定时下载文件
        - 定时执行脚本
    - yum日志分析
        - yum下载记录
        - yum卸载记录
        - yum安装可疑工具
    - dmesg日志分析[内核自检日志]
    - btmp日志分析[错误登录日志]
    - lastlog日志分析[所有用户最后一次登录日志]
    - wtmp日志分析[所有用户登录日志]
    - journalctl工具日志分析
        - 最近24小时日志
    - auditd 服务状态
    - rsyslog 配置文件

### 5. 后门排查
- 后门特征检测(SUID/SGID/启动项/异常进程)[待完成]

### 6. 隧道检测
- SSH隧道检测
    - 同一PID的多个sshd连接
    - SSH本地转发特征
    - SSH远程转发特征
    - SSH动态转发(SOCKS代理)特征
    - SSH多级跳板特征
    - SSH隧道网络流量特征
    - SSH隧道持久化特征
- HTTP隧道检测[待完成]
- DNS隧道检测[待完成]
- ICMP隧道检测[待完成]
- 其他隧道工具检测[待完成]

### 7. webshell排查
- WebShell 排查(关键词匹配/文件特征)[待完成]

### 8. 病毒排查
- 病毒信息排查(已安装可疑软件/RPM检测)[待完成]

### 9. 内存排查
- 内存信息排查(内存占用/异常内容)[待完成]

### 10. 黑客工具排查
- 黑客工具匹配(规则自定义)
- 常见黑客痕迹排查(待完成)

### 11. 内核排查
- 内核驱动排查
- 可疑驱动排查(自定义可疑驱动列表)
- 内核模块检测

### 12. 其他排查
- 可疑脚本文件排查
- 系统文件完整性校验(MD5)
- 安装软件排查

### 13. Kubernetes排查
- 集群信息排查
- 集群凭据排查
- 集群敏感文件扫描
- 集群基线检查

### 14. 系统性能分析
- 磁盘使用情况
- CPU使用情况
- 内存使用情况
- 系统负载情况
- 网络流量情况

### 15. 基线检查


## 具体检查项对应的应急响应命令

### 1. 系统信息排查
#### IP 地址
- `ip -br a` - 简要查看网卡/IPv4/IPv6
- `ip route` - 查看默认网关与路由
- 输出格式：`<iface> <state> <ip/cidr> ...`、`default via <gw> dev <iface>`
- 解读要点：关注未知网卡、异常公网 IP、非预期默认路由

#### 系统版本信息
- `uname -a` - 内核版本、架构与编译信息
- `hostnamectl` - 主机名、内核、虚拟化等汇总（若 systemd 存在）
- 输出格式：`Linux <host> <kernel> ...`、`Operating System: ...`
- 解读要点：用于判断已知漏洞面、内核版本异常（与基线不符）

#### 系统发行版本
- `cat /etc/os-release` - 发行版与版本号（通用）
- `cat /etc/issue` - 登录前 banner（可能被篡改）
- 输出格式：`ID=<distro>`、`VERSION_ID=<ver>`
- 解读要点：结合包管理器日志（yum/dnf/apt）定位安装轨迹

#### 虚拟化环境检测
- `systemd-detect-virt` - 识别虚拟化类型（若 systemd 存在）
- `lscpu | grep -i hypervisor` - 识别是否存在 Hypervisor 提示
- 输出格式：`kvm|vmware|docker|none`
- 解读要点：用于判断是否容器/云主机，避免误判“看不到硬件信息”等现象

#### 正在登录用户
- `w` - 当前登录会话与正在执行的命令
- `who` - 当前登录用户/来源
- 输出格式：`USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT`
- 解读要点：重点关注 FROM 字段（陌生 IP/内网横向）、异常登录时间、异常终端

#### 系统最后登录用户
- `last -n 20` - 最近登录历史（wtmp）
- `lastb -n 20` - 最近失败登录（btmp，可能需要 root/文件存在）
- 输出格式：`<user> <tty> <from> <time> ...`
- 解读要点：结合 secure/auth 日志确认是否存在爆破、异地登录

#### 用户信息 passwd 文件分析
- `cat /etc/passwd` - 查看全部用户
- `awk -F: '$7 !~ /(nologin|false)$/ {print $1"\t"$3"\t"$7}' /etc/passwd` - 可能可登录用户与 shell
- 输出格式：`用户名:口令占位:UID:GID:注释:家目录:Shell`
- 解读要点：关注异常 UID/GID、异常 Shell（如 /bin/bash）、异常家目录路径

#### 检查超级用户（除 root 外）
- `awk -F: '$3==0{print $1}' /etc/passwd` - UID=0 用户
- 输出格式：`root` 或多个用户名
- 解读要点：除 root 外出现 UID=0 通常高风险（提权/后门账号）

#### 检查克隆用户（重复 UID）
- `cut -d: -f3 /etc/passwd | sort | uniq -d` - 输出重复 UID
- 输出格式：`<uid>`
- 解读要点：重复 UID 可能是克隆账号或人为伪装，需回溯创建时间与登录日志

#### 检查非系统用户（常用经验规则）
- `awk -F: '$3>=1000 && $3<65534 {print $1"\t"$3"\t"$7}' /etc/passwd` - 普通用户候选
- 输出格式：`<user> <uid> <shell>`
- 解读要点：根据发行版调整阈值（部分系统普通用户从 500 起）

#### 检查空口令用户（需要 root）
- `awk -F: '($2==""){print $1}' /etc/shadow` - shadow 中密码字段为空
- 输出格式：`<user>`
- 解读要点：空口令通常直接可被登录利用，需核验 PAM/SSH 配置

#### 检查口令未加密用户（旧系统风险）
- `awk -F: '($2!="x"){print $1"\t"$2}' /etc/passwd` - passwd 中出现明文/哈希
- 输出格式：`<user> <hash-or-plain>`
- 解读要点：现代系统应为 `x` 并使用 /etc/shadow 存储口令哈希

#### 用户组信息 group 文件分析
- `cat /etc/group` - 查看全部用户组
- `egrep '^(sudo|wheel):' /etc/group` - 常见特权组（不同发行版不同）
- 输出格式：`组名:口令占位:GID:成员列表`
- 解读要点：关注特权组成员是否异常增加

#### 相同 GID 用户组
- `cut -d: -f3 /etc/group | sort | uniq -d` - 输出重复 GID
- 输出格式：`<gid>`
- 解读要点：重复 GID 不一定是问题，但结合组名/成员变化可定位异常

#### 系统计划任务
- `cat /etc/crontab` - 系统 cron 配置
- `ls -la /etc/cron.*` - cron 目录（daily/hourly 等）
- 输出格式：`分 时 日 月 周 用户 命令`
- 解读要点：关注可疑下载/执行（wget/curl/bash|sh/python）、隐藏脚本路径

#### 用户计划任务
- `crontab -l` - 当前用户 cron
- `ls -la /var/spool/cron/ 2>/dev/null` - 所有用户 cron 文件（路径随发行版变化）
- 输出格式：`分 时 日 月 周 命令`
- 解读要点：关注异常用户的定时任务、执行权限与脚本落地路径

#### 输出当前 shell 历史命令
- `history | tail -200` - 当前会话历史（数量可调）
- 输出格式：`<序号> <命令>`
- 解读要点：用于快速锁定近期操作；注意 history 可能被清理/禁用

#### 输出用户历史命令（bash）
- `cat ~/.bash_history 2>/dev/null | tail -200` - 当前用户 bash 历史
- `cat /root/.bash_history 2>/dev/null | tail -200` - root bash 历史
- 输出格式：逐行命令
- 解读要点：关注下载执行、提权、横向、清理痕迹相关命令

#### 是否下载过脚本文件（基于历史与落地文件两条线）
- `grep -Ein '(^|[[:space:]])(wget|curl)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null` - 历史中的下载行为
- `find /tmp /var/tmp /dev/shm -maxdepth 2 -type f -mmin -1440 2>/dev/null` - 常见临时目录近 24 小时落地文件
- 输出格式：命令匹配行、文件路径列表
- 解读要点：下载行为要结合落地路径与执行痕迹（chmod、bash/sh 执行）

#### 是否通过主机下载/传输过文件
- `grep -Ein '(^|[[:space:]])(scp|sftp|ftp|tftp|rsync)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null` - 历史中的传输行为
- 输出格式：命令匹配行
- 解读要点：结合网络连接、日志（secure/auth）定位对端 IP 与时间

#### 是否增加/删除过账号
- `grep -Ein '(^|[[:space:]])(useradd|userdel|usermod|groupadd|groupdel|passwd|chage)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`
- 输出格式：命令匹配行
- 解读要点：再与 `/etc/passwd`、`/etc/group` 的修改时间、secure/auth 日志交叉验证

#### 是否执行过黑客命令/工具
- `grep -Ein '(^|[[:space:]])(nc|ncat|netcat|socat|proxychains|frp|ngrok|msfconsole|nmap)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`
- 输出格式：命令匹配行
- 解读要点：命中后优先检查对应进程、监听端口、落地二进制位置

#### 其他敏感命令
- `grep -Ein '(^|[[:space:]])(chmod|chattr|iptables|nft|firewall-cmd|setenforce|getenforce|crontab|systemctl)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`
- 输出格式：命令匹配行
- 解读要点：用于定位“持久化”“清痕”“关闭防护”等行为

#### 检查系统中所有可能的历史文件路径
- `find / -maxdepth 4 -type f -name '.*history' 2>/dev/null` - 查找历史文件（可能较慢）
- 输出格式：文件路径列表
- 解读要点：某些 shell/工具会生成不同历史文件（如 zsh、mysql 等）

#### 输出系统中所有用户的 bash 历史文件
- `for f in /home/*/.bash_history /root/.bash_history; do [ -e "$f" ] && echo "===== $f =====" && tail -200 "$f"; done 2>/dev/null`
- 输出格式：按文件分段输出
- 解读要点：用“分段输出”避免混淆不同用户行为

#### 输出数据库操作历史命令（基于历史关键字）
- `grep -Ein '(^|[[:space:]])(mysql|mysqldump|psql|redis-cli|mongo|mongosh)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`
- 输出格式：命令匹配行
- 解读要点：结合数据库审计/慢日志（若开启）进一步确认数据访问与导出

### 2. 网络连接排查
#### ARP 攻击分析
- `ip neigh` - 查看邻居表（替代 arp 的通用方式）
- `arp -an 2>/dev/null` - 查看 ARP 缓存（旧工具，存在则可用）
- 输出格式：`<ip> dev <iface> lladdr <mac> REACHABLE|STALE`
- 解读要点：关注同一 IP 对应 MAC 频繁变化、网关 IP 的 MAC 异常

#### 网络连接分析
- `ss -antup` - TCP/UDP 连接与进程映射（推荐）
- `netstat -antup 2>/dev/null` - 连接与监听（旧工具）
- 输出格式：`ESTAB|LISTEN ... users:(("proc",pid=...))`
- 解读要点：关注外联到陌生公网、长连接、异常高频连接与可疑进程名

#### TCP 端口检测
- `ss -lntp` - TCP 监听端口与进程
- `netstat -lntp 2>/dev/null` - TCP 监听端口与进程（旧工具）
- 输出格式：`LISTEN <addr>:<port> users:(("proc",pid=...))`
- 解读要点：关注 0.0.0.0/:: 上的异常端口、非预期服务对外监听

#### TCP 高危端口（自定义高危端口组）
- `awk '{print $1}' checkrules/dangerstcpports.txt 2>/dev/null` - 查看高危端口规则（仓库内）
- `ss -lntp | awk 'NR>1{print $4}' | sed 's/.*://g' | sort -n | uniq` - 提取监听端口清单
- 输出格式：规则端口列表、监听端口列表
- 解读要点：将监听端口与规则端口交叉比对，优先排查命中项对应进程

#### UDP 端口检测
- `ss -lnup` - UDP 监听端口与进程
- `netstat -lnup 2>/dev/null` - UDP 监听端口与进程（旧工具）
- 输出格式：`UNCONN <addr>:<port> users:(...)`
- 解读要点：关注异常 UDP 监听与可疑进程（隧道/木马常见）

#### UDP 高危端口（自定义高危端口组）
- `awk '{print $1}' checkrules/dangersudpports.txt 2>/dev/null` - 查看高危端口规则（仓库内）
- `ss -lnup | awk 'NR>1{print $5}' | sed 's/.*://g' | sort -n | uniq` - 提取 UDP 监听端口
- 输出格式：规则端口列表、监听端口列表
- 解读要点：优先核验命中端口的服务是否属于业务预期

#### DNS 信息排查
- `cat /etc/resolv.conf` - DNS 服务器配置
- `grep -v '^[[:space:]]*#' /etc/hosts` - hosts 静态解析（可能被投毒）
- 输出格式：`nameserver <ip>`
- 解读要点：关注 nameserver 指向陌生公网、hosts 对常见域名做了异常指向

#### 网卡工作模式（是否混杂模式）
- `ip link | grep -E '^[0-9]+:|PROMISC'` - 查看是否开启 PROMISC
- 输出格式：网卡 flags 中包含 `PROMISC`
- 解读要点：混杂模式可能用于嗅探；也可能是正常抓包/虚拟化导致，需结合进程与时间判断

#### 网络路由信息排查
- `ip route show` - 路由表
- `ip rule show` - 策略路由（存在则需关注）
- 输出格式：`default via ...`、`from ... lookup ...`
- 解读要点：关注异常静态路由、策略路由将流量导向非预期网关

#### 路由转发排查
- `sysctl net.ipv4.ip_forward` - IPv4 转发是否开启
- `sysctl net.ipv6.conf.all.forwarding` - IPv6 转发是否开启
- 输出格式：`net.ipv4.ip_forward = 0|1`
- 解读要点：转发开启不一定恶意，但常见于代理/隧道/横向中继

#### 防火墙策略排查
- `iptables -S 2>/dev/null` - iptables 规则（存在则优先）
- `nft list ruleset 2>/dev/null` - nftables 规则（新系统）
- `firewall-cmd --list-all 2>/dev/null` - firewalld 概览（存在则可用）
- 输出格式：规则链/表输出
- 解读要点：关注放行可疑端口、对特定 IP 白名单、出站流量异常放行

### 3. 进程排查
#### ps 进程分析
- `ps aux` - 所有进程（BSD 风格）
- `ps -ef` - 所有进程（SYSV 风格）
- `ps aux --sort=-%cpu | head -20` - CPU 占用 Top
- `ps aux --sort=-%mem | head -20` - 内存占用 Top
- 输出格式：`USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND`
- 解读要点：关注可疑命令行（落地路径、可疑参数）、异常用户、伪装进程名

#### top 进程分析
- `top -b -n 1 | head -80` - 批处理模式抓取一屏概览
- 输出格式：负载、CPU、内存、Top 进程列表
- 解读要点：关注持续高占用且命令行可疑的进程，必要时进一步取样多次对比

#### 规则匹配敏感进程（自定义进程组）
- `ps -ef | egrep -i '(nc|ncat|netcat|socat|frp|ngrok|proxychains|ssh -[NRD])'` - 常见隧道/代理/横向特征
- 输出格式：`UID PID PPID ... CMD`
- 解读要点：命中后优先做“网络连接映射”“可执行文件路径与哈希”确认

#### 异常进程检测（经验型）
- `ps -eo pid,ppid,user,tty,stat,lstart,cmd --sort=ppid | head -200` - 观察进程树与启动时间
- 输出格式：`PID PPID USER ... LSTART CMD`
- 解读要点：关注 PPID 异常、TTY 缺失但长期驻留、启动时间落在攻击窗口

#### 孤儿进程检测
- `ps -eo pid,ppid,cmd | awk '$2==1{print $0}' | head -50` - PPID=1 的孤儿进程（截断展示）
- 输出格式：`PID PPID CMD`
- 解读要点：孤儿进程不一定恶意，但可作为“守护化/持久化”线索

#### 网络连接和进程映射
- `ss -antup` - 连接与进程
- `lsof -i -n -P 2>/dev/null | head -200` - 端口与打开的 socket
- 输出格式：`users:(("proc",pid=...))`、`COMMAND PID USER FD TYPE ... NAME`
- 解读要点：关注外联进程的二进制路径、父进程、是否为业务预期

#### 进程可疑内存映射
- `pmap -x <PID> 2>/dev/null | head -80` - 查看内存映射概览（若 pmap 存在）
- `cat /proc/<PID>/maps 2>/dev/null | head -80` - 直接查看映射（通用）
- 输出格式：`address perms offset dev inode pathname`
- 解读要点：关注映射到 `/tmp`、`/dev/shm`、匿名可执行段（r-xp 且无路径）等

#### 文件描述符异常进程
- `ls -l /proc/<PID>/fd 2>/dev/null | head -50` - 查看 FD 指向
- `lsof -p <PID> 2>/dev/null | head -80` - 进程打开文件/网络（若 lsof 存在）
- 输出格式：FD 列表、lsof 表格
- 解读要点：关注指向已删除文件（`(deleted)`）、异常大量 FD、可疑 socket

#### 系统调用表完整性检测（需要 root，rootkit 线索）
- `grep -n "sys_call_table" /proc/kallsyms 2>/dev/null` - 查找系统调用表符号
- 输出格式：`<addr> <type> sys_call_table`
- 解读要点：符号消失/不可读可能是权限/内核配置，也可能是隐藏行为；需结合内核模块与完整性校验进一步判断

#### 进程启动时间异常检测
- `ps -eo pid,lstart,cmd --sort=lstart | head -50` - 最早启动的一批进程
- `ps -eo pid,lstart,cmd --sort=-lstart | head -50` - 最新启动的一批进程
- 输出格式：`PID LSTART CMD`
- 解读要点：将启动时间与告警时间/日志时间对齐，定位攻击落点进程

#### 进程环境变量异常检测
- `tr '\0' '\n' < /proc/<PID>/environ 2>/dev/null | head -80` - 查看环境变量
- 输出格式：`KEY=VALUE`
- 解读要点：关注 LD_PRELOAD、PATH 异常、代理变量（http_proxy/https_proxy）等劫持线索

### 4. 文件排查
#### 系统服务收集
- `systemctl list-unit-files --type=service 2>/dev/null | head -200` - 列出服务清单（若 systemd 存在）
- 输出格式：`UNIT FILE STATE`
- 解读要点：用于快速定位“新增服务/异常 service 文件”

#### 系统自启动服务分析
- `systemctl list-unit-files --type=service 2>/dev/null | grep enabled` - 已启用服务
- `cat /etc/rc.local 2>/dev/null` - rc.local 自启动（存在则关注）
- 输出格式：enabled 列表、rc.local 内容
- 解读要点：关注可疑服务名、可疑 ExecStart 路径、执行脚本落地位置

#### 系统正在运行的服务分析
- `systemctl --type=service --state=running 2>/dev/null | head -200` - 正在运行服务
- 输出格式：`UNIT LOAD ACTIVE SUB DESCRIPTION`
- 解读要点：关注业务外服务、名称伪装（与常见系统服务相近）

#### 用户服务分析（若 systemd user 存在）
- `systemctl --user list-unit-files --type=service 2>/dev/null | head -200` - 用户级服务
- 输出格式：`UNIT FILE STATE`
- 解读要点：用户级持久化常见于无 root 落地场景

#### /tmp 目录
- `ls -alh /tmp /var/tmp /dev/shm 2>/dev/null | head -200` - 临时目录文件
- 输出格式：`权限 用户 组 大小 时间 文件名`
- 解读要点：关注可执行文件、最近修改文件、随机命名脚本与 ELF

#### /root 目录隐藏文件（隐藏文件分析）
- `ls -al /root 2>/dev/null | grep '^\.' || true` - root 家目录隐藏项
- 输出格式：以 `.` 开头的文件/目录
- 解读要点：关注异常 `.ssh`、隐藏脚本、隐藏配置与可执行文件

#### .ssh 目录排查
- `ls -al ~/.ssh 2>/dev/null` - 当前用户 ssh 目录
- `ls -al /root/.ssh 2>/dev/null` - root ssh 目录
- 输出格式：文件列表与权限
- 解读要点：关注目录权限是否过宽（如 777/775），以及异常新增文件

#### 公钥私钥排查
- `ls -al ~/.ssh/id_* /root/.ssh/id_* 2>/dev/null` - 常见 key 文件
- `ssh-keygen -lf ~/.ssh/id_rsa.pub 2>/dev/null` - 打印公钥指纹（按实际文件调整）
- 输出格式：文件列表、`<bits> <fingerprint> <comment>`
- 解读要点：对比指纹与基线，确认是否为授权运维密钥

#### authorized_keys 文件排查
- `cat ~/.ssh/authorized_keys 2>/dev/null` - 当前用户授权 key
- `cat /root/.ssh/authorized_keys 2>/dev/null` - root 授权 key
- 输出格式：逐行公钥
- 解读要点：新增 key 是最常见的持久化方式之一；需确认来源与变更窗口

#### known_hosts 文件排查
- `cat ~/.ssh/known_hosts 2>/dev/null | tail -50` - 最近记录（数量可调）
- 输出格式：`hostnames keytype base64key`
- 解读要点：用于辅助还原曾经连接的主机；并非直接恶意证据

#### sshd_config 文件分析
- `grep -v '^[[:space:]]*#' /etc/ssh/sshd_config 2>/dev/null` - 生效配置（去掉注释）
- `sshd -T 2>/dev/null | head -200` - 解析后的最终配置（若支持）
- 输出格式：`Key Value` 或 `key value`
- 解读要点：重点检查是否允许空口令、是否允许 root 登录、允许的认证方式与端口

#### 检测是否允许空口令登录（sshd）
- `sshd -T 2>/dev/null | grep -i '^permitemptypasswords'` - 最终配置（若支持）
- `grep -i '^[[:space:]]*PermitEmptyPasswords' /etc/ssh/sshd_config 2>/dev/null` - 原始配置
- 输出格式：`permitemptypasswords yes|no`
- 解读要点：`yes` 通常高风险；需同步检查 PAM 与账号口令状况

#### 检测是否允许 root 远程登录（sshd）
- `sshd -T 2>/dev/null | grep -i '^permitrootlogin'` - 最终配置（若支持）
- `grep -i '^[[:space:]]*PermitRootLogin' /etc/ssh/sshd_config 2>/dev/null` - 原始配置
- 输出格式：`permitrootlogin yes|no|prohibit-password`
- 解读要点：业务不需要时建议禁用；若必须开启需配合 MFA/白名单等措施

#### 检测 ssh 协议版本 / ssh 版本
- `ssh -V` - SSH 客户端版本（输出到 stderr）
- `sshd -V 2>&1 | head -1` - SSH 服务端版本（部分发行版可用）
- 输出格式：`OpenSSH_X.YpZ ...`
- 解读要点：用于判断是否存在已知漏洞，以及是否被替换为异常版本

#### 环境变量文件分析
- `cat /etc/profile 2>/dev/null` - 全局 profile
- `ls -la ~/.bashrc ~/.bash_profile 2>/dev/null` - 用户启动文件
- 输出格式：文件内容/权限
- 解读要点：关注 PATH/LD_PRELOAD/alias 注入、可疑远程下载执行片段

#### env 命令分析
- `env | sort | head -200` - 当前环境变量
- 输出格式：`KEY=VALUE`
- 解读要点：关注代理变量、LD_* 相关变量、PATH 异常顺序

#### hosts 文件排查
- `cat /etc/hosts` - hosts 静态解析
- 输出格式：`<ip> <hostname> ...`
- 解读要点：关注对常见域名/更新源做了异常指向（投毒/劫持）

#### shadow/gshadow 文件权限与属性
- `ls -l /etc/shadow /etc/gshadow 2>/dev/null` - 权限查看
- `stat /etc/shadow /etc/gshadow 2>/dev/null` - 详细属性
- `lsattr /etc/shadow /etc/gshadow 2>/dev/null` - 扩展属性（若支持）
- 输出格式：权限字符串、stat 表格、`----i--------` 等
- 解读要点：权限异常（可被普通用户读取）或被加 immutable（i）都需要重点核查

#### 24 小时变动文件排查
- `find / -mtime -1 -type f 2>/dev/null | head -200` - 最近修改文件（可能较慢）
- 输出格式：文件路径列表
- 解读要点：优先结合 /tmp、/dev/shm、web 目录、启动项目录做定向排查

#### SUID/SGID 文件排查
- `find / -perm -4000 -type f 2>/dev/null | head -200` - SUID
- `find / -perm -2000 -type f 2>/dev/null | head -200` - SGID
- 输出格式：文件路径列表
- 解读要点：新增/异常 SUID 是提权常见入口，需与系统基线比对

### 5. 日志排查
#### message 日志分析（CentOS/RHEL 常见）
- `grep -Ein 'rz|sz|ZMODEM' /var/log/messages* 2>/dev/null | head -50` - ZMODEM 传输线索
- `grep -Ein 'named|dnsmasq|resolv|DNS' /var/log/messages* 2>/dev/null | head -50` - DNS 相关线索（粗粒度）
- 输出格式：`文件:行号:内容`
- 解读要点：用于快速发现“交互式传输/可疑解析行为”，命中后需结合时间轴深挖

#### secure/auth 日志分析（登录/认证）
- `grep -Ein 'Accepted |Failed password|Invalid user' /var/log/secure* /var/log/auth.log* 2>/dev/null | head -80` - 登录成功/失败/无效用户
- `grep -Ein 'useradd|userdel|usermod|groupadd|groupdel' /var/log/secure* /var/log/auth.log* 2>/dev/null | head -80` - 新增用户/组线索
- 输出格式：`时间 主机 进程[PID]: 消息`
- 解读要点：用于爆破/撞库判断、定位新增账号时间点与来源 IP

#### 计划任务日志分析（cron）
- `grep -Ein 'CRON|cron' /var/log/cron* /var/log/messages* /var/log/syslog* 2>/dev/null | head -120` - cron 运行轨迹（不同发行版日志位置不同）
- 输出格式：`时间 主机 CRON[PID]: (user) CMD (command)`
- 解读要点：用于发现“定时下载/定时执行脚本”等持久化行为

#### yum/dnf 日志分析（RPM 系）
- `grep -Ein 'Installed:|Updated:|Erased:' /var/log/yum.log* /var/log/dnf.log* 2>/dev/null | tail -200` - 安装/更新/卸载记录
- 输出格式：`时间 操作: 包名-版本`
- 解读要点：用于定位可疑工具安装（如 nmap/nc/socat），以及异常卸载（清痕）

#### dmesg 日志分析（内核自检/异常）
- `dmesg | tail -200` - 最近内核消息（数量可调）
- 输出格式：`[timestamp] message`
- 解读要点：用于发现内核模块加载、硬件/驱动异常、OOM 等关键事件

#### btmp / lastlog / wtmp 日志分析
- `lastb -n 50 2>/dev/null` - 失败登录（btmp）
- `lastlog | head -200` - 所有用户最后登录（lastlog）
- `last -n 50` - 登录历史（wtmp）
- 输出格式：表格/记录行
- 解读要点：与 secure/auth 日志互证，构建攻击时间线

#### journalctl 工具日志分析（systemd）
- `journalctl --since "24 hours ago" --no-pager | tail -200` - 最近 24 小时
- 输出格式：`时间 主机 单元[PID]: 消息`
- 解读要点：systemd 系发行版的统一日志入口，可替代部分 /var/log 文件

#### auditd 服务状态
- `systemctl status auditd 2>/dev/null | head -120` - auditd 状态（若存在）
- 输出格式：Active/Loaded/Recent logs
- 解读要点：审计开启与否影响“是否能还原关键操作”，异常停用也需关注

#### rsyslog 配置文件
- `cat /etc/rsyslog.conf 2>/dev/null` - 主配置
- `ls -la /etc/rsyslog.d 2>/dev/null` - 分段配置目录
- 输出格式：配置内容/文件列表
- 解读要点：关注是否被改写转发到陌生日志服务器、是否屏蔽关键设施日志

### 6. 后门排查
#### 后门特征检测（脚本未覆盖，按需手工）
- `cat /etc/ld.so.preload 2>/dev/null` - LD_PRELOAD 劫持（常见后门点）
- `echo "$LD_PRELOAD"` - 环境变量劫持
- 输出格式：路径或空
- 解读要点：命中后需核验对应 so 文件来源、时间线与引用进程

### 7. 隧道检测
#### SSH 隧道检测（脚本覆盖：同 PID 多连接为主要判据）
- `netstat -anpot 2>/dev/null | grep sshd` - 查看 sshd 连接（旧工具）
- `ss -antp | grep sshd` - 查看 sshd 连接（推荐）
- 输出格式：`ESTAB ... <local>:<port> <remote>:<port> users:(("sshd",pid=...))`
- 解读要点：同一 pid 出现多个外联连接、来源/目的异常时，需要进一步检查对应进程树与启动参数

#### SSH 本地/远程/动态转发特征（经验型）
- `ps -ef | egrep -i 'ssh .*(-L|-R|-D)[[:space:]]'` - 查找 ssh 转发参数
- 输出格式：命令行中包含 `-L`/`-R`/`-D`
- 解读要点：命中后结合 `ss -antp` 确认是否形成实际转发通道

#### SSH 隧道持久化特征（经验型）
- `ps -ef | egrep -i '(autossh|ssh .*ServerAliveInterval|ssh .*ControlMaster)'` - 常见保活/复用
- 输出格式：命令行命中
- 解读要点：持久化通常伴随计划任务/服务自启动，需与“启动项/cron”联动排查

#### HTTP/DNS/ICMP/其他隧道工具检测
- （脚本未覆盖，按需手工）`ps -ef | egrep -i '(iodine|dnscat|chisel|frp|ngrok)'` - 常见隧道工具
- 输出格式：命令行命中
- 解读要点：命中后优先做二进制来源、配置文件、网络端口与落地路径核验

### 8. webshell 排查
#### WebShell 排查（脚本未覆盖，按需手工）
- `find /var/www /srv -type f -name '*.php' -o -name '*.jsp' -o -name '*.aspx' 2>/dev/null | head -200` - 定位常见 web 目录脚本
- `grep -RInE '(eval\(|base64_decode\(|gzinflate\(|assert\(|system\(|passthru\()' /var/www 2>/dev/null | head -200` - 常见危险函数
- 输出格式：文件列表、`文件:行号:内容`
- 解读要点：命中后需结合访问日志与文件时间线判断是否为真实后门

### 9. 病毒排查
#### 病毒信息排查（脚本未覆盖，按需手工）
- `rpm -qa 2>/dev/null | head -200` - RPM 已安装包（RPM 系）
- `dpkg -l 2>/dev/null | head -200` - DEB 已安装包（DEB 系）
- 输出格式：软件包列表
- 解读要点：结合 yum/dnf/apt 日志与业务白名单，定位可疑工具/组件

### 10. 内存排查
#### 内存信息排查（脚本未覆盖，按需手工）
- `free -h` - 内存概览
- `ps aux --sort=-%mem | head -20` - 内存占用 Top
- 输出格式：`Mem: total used free ...`、进程表
- 解读要点：用于定位异常占用进程；进阶内存取证需结合专用工具

### 11. 黑客工具排查
#### 黑客工具匹配（规则自定义）
- `cat checkrules/hackertoolslist.txt 2>/dev/null | head -200` - 工具关键字规则（仓库内）
- `ps -ef | egrep -i "$(tr '\n' '|' < checkrules/hackertoolslist.txt 2>/dev/null | sed 's/|$//')" 2>/dev/null | head -200` - 按规则匹配进程（命中率依规则）
- 输出格式：规则列表、进程列表
- 解读要点：规则需要结合环境裁剪；命中后用“二进制路径+哈希+连接”三要素确认

### 12. 内核排查
#### 内核驱动/模块排查
- `lsmod | head -200` - 已加载内核模块
- `modinfo <module> 2>/dev/null | head -80` - 模块信息（按需）
- 输出格式：`Module Size Used by`
- 解读要点：关注异常模块名、加载时间线（结合 dmesg），以及是否来源于非标准路径

#### 可疑驱动排查（自定义列表）
- （按需手工）`grep -Ein '^(<module1>|<module2>)$' <suspect_list> 2>/dev/null` - 自定义列表匹配
- 输出格式：命中模块名
- 解读要点：将“已加载模块”与“可疑列表”比对，优先核验命中项来源与符号信息

### 13. 其他排查
#### 可疑脚本文件排查
- `find /tmp /var/tmp /dev/shm -type f \( -name '*.sh' -o -name '*.py' -o -name '*.pl' \) 2>/dev/null | head -200` - 临时目录脚本
- `find /etc /usr/local -type f -name '*.sh' 2>/dev/null | head -200` - 常见持久化脚本目录（按需扩展）
- 输出格式：文件路径列表
- 解读要点：结合修改时间、属主、执行位、脚本内容关键字（curl/wget/bash）判断风险

#### 系统文件完整性校验（MD5，脚本覆盖）
- `md5sum <file>` - 计算单个文件 MD5
- `md5sum -c <md5_list_file>` - 按基线清单做校验（脚本会生成/校验 sysfile_md5.txt）
- 输出格式：`<md5> <path>`、`<path>: OK|FAILED`
- 解读要点：应与“可信基线”比对；单机自生成 MD5 只能用于后续对比，不能证明当前未被篡改

#### 安装软件排查
- `which <cmd> 2>/dev/null` - 定位命令来源路径
- `rpm -qf <path> 2>/dev/null` - 查路径属于哪个 RPM 包（RPM 系）
- `dpkg -S <path> 2>/dev/null` - 查路径属于哪个 DEB 包（DEB 系）
- 输出格式：包名/无匹配
- 解读要点：用于判断二进制是否来自系统包，或为攻击者落地的“自带工具”

### 14. Kubernetes 排查
#### 集群信息/凭据/敏感文件/基线（脚本未覆盖，按需手工）
- `kubectl version --short 2>/dev/null` - 版本信息（若已安装/配置）
- `find /etc /var/lib -maxdepth 4 -type f -name '*.kubeconfig' -o -name 'admin.conf' 2>/dev/null | head -200` - 常见 kubeconfig/凭据
- 输出格式：版本输出、文件路径列表
- 解读要点：凭据文件命中需立即做权限核查与旋转，避免被用于集群横向扩散

### 15. 系统性能分析
#### 磁盘使用情况
- `df -h` - 文件系统使用率
- 输出格式：`Filesystem Size Used Avail Use% Mounted on`
- 解读要点：磁盘满可能导致日志丢失/服务异常，也可能是攻击者落地文件堆积

#### CPU 使用情况
- `top -b -n 1 | head -40` - CPU 概览与 Top 进程
- 输出格式：`%Cpu(s): ...` + 进程列表
- 解读要点：关注持续异常占用进程，结合“进程排查”定位来源

#### 内存使用情况
- `free -h` - 内存概览
- 输出格式：`Mem: total used free ...`
- 解读要点：关注缓存/可用内存变化与异常 swap 使用

#### 系统负载情况
- `uptime` - 负载与在线时间
- 输出格式：`load average: 1m, 5m, 15m`
- 解读要点：负载异常需结合 CPU/IO/内存综合判断，不等同于 CPU 100%

#### 网络流量情况
- `ip -s link` - 网卡收发统计
- 输出格式：`RX: bytes packets errors dropped ...`
- 解读要点：流量异常可提示数据外传/隧道活动，需结合连接表与进程映射确认

### 16. 基线检查
#### 账号审查（复用“用户信息分析”）
- `cat /etc/passwd` - 用户清单
- `awk -F: '$3==0{print $1}' /etc/passwd` - UID=0 用户
- 输出格式：用户列表/用户名列表
- 解读要点：基线的核心是“是否出现新增/异常账号”，需结合日志与时间线

#### 密码有效期策略（login.defs，脚本覆盖）
- `grep -v '^[[:space:]]*#' /etc/login.defs | grep -E '^PASS_'` - PASS_* 策略项
- 输出格式：`PASS_MAX_DAYS <n>`、`PASS_MIN_DAYS <n>`、`PASS_MIN_LEN <n>`、`PASS_WARN_AGE <n>`
- 解读要点：用于基线合规检查；同时为“账号安全事件”提供判断依据（弱策略更易爆破）

#### 密码复杂度策略（PAM，脚本覆盖）
- `grep -v '^[[:space:]]*#' /etc/pam.d/system-auth 2>/dev/null` - PAM 策略（CentOS/RHEL 常见路径）
- `grep -v '^[[:space:]]*#' /etc/pam.d/common-password 2>/dev/null` - PAM 策略（Debian/Ubuntu 常见路径）
- 输出格式：PAM 配置行
- 解读要点：关注 pwquality/cracklib、重试次数、最小长度/复杂度等配置是否被弱化

#### 密码已过期用户（基于 /etc/shadow，脚本覆盖）
- `awk -F: -v today="$(($(date +%s)/86400))" '($5!="" && today>$3+$5){print $1}' /etc/shadow 2>/dev/null` - 输出密码已过期用户
- 输出格式：用户名列表
- 解读要点：过期不一定是入侵，但可提示“长期无人维护账号/策略失效”，需结合登录日志判断

#### 账号超时锁定策略（TMOUT，脚本覆盖）
- `grep -n 'TMOUT' /etc/profile 2>/dev/null` - 检查是否设置会话超时
- 输出格式：`TMOUT=<seconds>` 或无输出
- 解读要点：未设置/设置过大可能导致共享终端风险增大；变更时间可作为异常线索

#### grub2 密码策略（脚本覆盖）
- `grep -nE '^[[:space:]]*password_pbkdf2' /boot/grub2/grub.cfg 2>/dev/null` - 检查是否启用 grub2 密码（PBKDF2）
- 输出格式：命中行或无输出
- 解读要点：引导层保护属于“物理/控制台安全基线”，云环境也可用于防止低层篡改

#### 远程登录限制（TCP Wrappers，脚本覆盖）
- `grep -v '^[[:space:]]*#' /etc/hosts.allow 2>/dev/null` - 允许列表
- `grep -v '^[[:space:]]*#' /etc/hosts.deny 2>/dev/null` - 拒绝列表
- 输出格式：规则行或空
- 解读要点：关注是否被加入了异常放行 IP/网段，或策略被清空导致放大暴露面

#### 防火墙策略检查（脚本覆盖）
- `firewall-cmd --get-active-zones 2>/dev/null` - firewalld 活跃区域
- `firewall-cmd --list-all 2>/dev/null` - 区域规则概览（若 firewalld 存在）
- `iptables -L -n -v 2>/dev/null` - iptables 规则（若存在）
- 输出格式：区域/规则表
- 解读要点：关注异常放行端口、对特定 IP 的放行/拒绝，以及出站策略异常

#### SELinux 策略检查（脚本覆盖）
- `getenforce 2>/dev/null` - 当前模式
- `sestatus 2>/dev/null | head -50` - 状态摘要
- `grep '^SELINUX=' /etc/selinux/config 2>/dev/null` - 默认启动模式
- 输出格式：`Enforcing|Permissive|Disabled`、状态表
- 解读要点：SELinux 从 Enforcing 被改为 Permissive/Disabled 时，需要结合变更窗口重点核查

#### 关键文件权限与 core dump（脚本覆盖）
- `stat -c '%A %U %G %n' /etc/passwd /etc/group /etc/securetty /etc/services 2>/dev/null` - 登录相关关键文件权限
- `grep -nE '^\\*\\s+(soft|hard)\\s+core\\s+0' /etc/security/limits.conf 2>/dev/null` - core dump 禁用策略
- 输出格式：`<perm> <owner> <group> <path>`、匹配行
- 解读要点：关键文件权限异常可能导致提权/信息泄露；core dump 未禁用可能泄露敏感内存信息

---------
