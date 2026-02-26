<!-- ---
name: linux-emergency-response
description: Linux 应急响应专用工具，用户只需提供 SSH 连接信息，AI 自动引导进行全面的入侵排查并分析结果。
--- -->

# SKILL.md
id: linux-emergency-response
name: Linux Gun AI Skills
trigger_regex: (?=.*hostname\s*=\s*\S+)(?=.*port\s*=\s*\d+)(?=.*username\s*=\s*\S+)(?=.*password\s*=\s*\S+).+
description: Linux 应急响应专用工具，用户只需提供 SSH 连接信息，AI 自动引导进行全面的入侵排查并分析结果。

## 详细指令
当用户消息同时包含 hostname、port、username、password 四段时：
1. **保存连接信息**：解析并保存用户的 SSH 连接参数。
2. **建立连接**：使用提供的凭据尝试连接目标主机。如果连接失败，请明确提示用户检查网络连通性或凭据正确性。
3. **引导排查**：连接成功后，主动询问用户想要排查的方向，建议从“0. 快速全面排查”开始。
4. **执行与分析**：**每次执行命令后，必须结合命令回显进行深度分析，绝不能只展示输出不给结论。**

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
- [🔴/🟡] [可疑项及判断依据]
  > 📝 证据: [相关日志/进程/文件路径]

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
- 🔴 **高危**: 确认存在入侵痕迹、后门、Webshell 或正在运行的恶意进程
- 🟡 **中危**: 存在可疑配置、未知进程、非标准端口监听或安全基线缺失
- 🟢 **低危/正常**: 未发现明显异常，符合常规系统状态

## 排查流程选择建议
连接成功后，询问用户选择使用哪个排查流程：
0. **快速全面排查**（自动执行关键检查项并汇总分析）
1. 系统信息排查
2. 网络连接排查
3. 进程排查
4. 文件排查
5. 日志排查
6. 后门排查
7. 隧道检测
8. Webshell排查
9. 病毒排查
10. 内存排查
11. 黑客工具排查
12. 内核排查
13. 其他排查
14. Kubernetes排查
15. 系统性能分析
16. 基线检查

## 排查流程对应的详细检查项
- 如果用户选择 `快速全面排查`，则按顺序执行以下所有高优先级的检查项（建议覆盖 1-6, 11, 16）。

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

### 5. 日志排查
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

### 6. 后门排查
- 后门特征检测
    - LD_PRELOAD 环境变量劫持检测
    - PROMPT_COMMAND 后门检测
    - Alias 别名后门检测
    - SSH Wrapper 后门检测
    - inetd.conf/xinetd.conf 后门检测
    - 系统启动脚本后门 (/etc/rc.local, /etc/init.d/)

### 7. 隧道检测
- SSH隧道检测
    - 同一PID的多个sshd连接
    - SSH本地转发特征
    - SSH远程转发特征
    - SSH动态转发(SOCKS代理)特征
    - SSH多级跳板特征
    - SSH隧道网络流量特征
    - SSH隧道持久化特征
- 常见隧道工具检测
    - HTTP隧道 (reGeorg, Neo-reGeorg)
    - DNS隧道 (dnscat2, iodine)
    - ICMP隧道 (icmptunnel, ptunnel)
    - 其他隧道 (frp, nps, ngrok)

### 8. Webshell排查
- WebShell 文件扫描
    - 最近修改的 Web 文件
    - 包含危险函数的 PHP/JSP/ASP 文件
    - 隐藏的 Web 文件
    - 上传目录中的可执行脚本

### 9. 病毒排查
- 异常资源占用检测 (挖矿病毒)
- 常见恶意目录可执行文件检测 (/tmp, /var/tmp, /dev/shm)
- 知名病毒/挖矿进程特征匹配

### 10. 内存排查
- 进程内存映射异常检测 (代码注入)
- 内存段权限异常检测 (rwx 段)

### 11. 黑客工具排查
- 常见黑客工具进程匹配 (nmap, sqlmap, hydra, metasploit, cobalt strike)
- 常见黑客工具文件残留

### 12. 内核排查
- 内核模块异常检测 (Rootkit)
- 隐藏内核模块检测
- 系统调用劫持迹象

### 13. 其他排查
- 可疑脚本文件排查
- 系统文件完整性校验(MD5)
- 安装软件排查

### 14. Kubernetes排查
- 集群节点状态排查
- 异常 Pod 排查 (特权容器, 挂载宿主机目录)
- 敏感权限绑定 (ClusterRoleBinding)
- 敏感 ConfigMap/Secret 扫描

### 15. 系统性能分析
- 磁盘使用情况
- CPU使用情况
- 内存使用情况
- 系统负载情况
- 网络流量情况

### 16. 基线检查
- 账号安全基线
- 密码策略基线
- 访问控制基线
- 日志审计基线

## 具体检查项对应的应急响应命令

### 1. 系统信息排查
#### IP 地址
- `ip -br a` - 简要查看网卡/IPv4/IPv6
- `ip route` - 查看默认网关与路由

#### 系统版本信息
- `uname -a` - 内核版本、架构与编译信息
- `hostnamectl` - 主机名、内核、虚拟化等汇总

#### 系统发行版本
- `cat /etc/os-release` - 发行版与版本号
- `cat /etc/issue` - 登录前 banner

#### 虚拟化环境检测
- `systemd-detect-virt` - 识别虚拟化类型
- `lscpu | grep -i hypervisor` - 识别是否存在 Hypervisor 提示

#### 正在登录用户
- `w` - 当前登录会话与正在执行的命令
- `who` - 当前登录用户/来源

#### 系统最后登录用户
- `last -n 20` - 最近登录历史
- `lastb -n 20` - 最近失败登录

#### 用户信息 passwd 文件分析
- `cat /etc/passwd` - 查看全部用户
- `awk -F: '$7 !~ /(nologin|false)$/ {print $1"\t"$3"\t"$7}' /etc/passwd` - 可能可登录用户与 shell

#### 检查超级用户（除 root 外）
- `awk -F: '$3==0{print $1}' /etc/passwd` - UID=0 用户

#### 检查克隆用户（重复 UID）
- `cut -d: -f3 /etc/passwd | sort | uniq -d` - 输出重复 UID

#### 检查非系统用户
- `awk -F: '$3>=1000 && $3<65534 {print $1"\t"$3"\t"$7}' /etc/passwd`

#### 检查空口令用户（需要 root）
- `awk -F: '($2==""){print $1}' /etc/shadow`

#### 检查口令未加密用户
- `awk -F: '($2!="x"){print $1"\t"$2}' /etc/passwd`

#### 用户组信息 group 文件分析
- `cat /etc/group`
- `egrep '^(sudo|wheel):' /etc/group`

#### 相同 GID 用户组
- `cut -d: -f3 /etc/group | sort | uniq -d`

#### 系统计划任务
- `cat /etc/crontab`
- `ls -la /etc/cron.*`

#### 用户计划任务
- `crontab -l`
- `ls -la /var/spool/cron/ 2>/dev/null`

#### 输出当前 shell 历史命令
- `history | tail -200`

#### 输出用户历史命令
- `cat ~/.bash_history 2>/dev/null | tail -200`
- `cat /root/.bash_history 2>/dev/null | tail -200`

#### 是否下载过脚本文件
- `grep -Ein '(^|[[:space:]])(wget|curl)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`
- `find /tmp /var/tmp /dev/shm -maxdepth 2 -type f -mmin -1440 2>/dev/null`

#### 是否通过主机下载/传输过文件
- `grep -Ein '(^|[[:space:]])(scp|sftp|ftp|tftp|rsync)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`

#### 是否增加/删除过账号
- `grep -Ein '(^|[[:space:]])(useradd|userdel|usermod|groupadd|groupdel|passwd|chage)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`

#### 是否执行过黑客命令/工具
- `grep -Ein '(^|[[:space:]])(nc|ncat|netcat|socat|proxychains|frp|ngrok|msfconsole|nmap)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`

#### 其他敏感命令
- `grep -Ein '(^|[[:space:]])(chmod|chattr|iptables|nft|firewall-cmd|setenforce|getenforce|crontab|systemctl)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`

#### 检查系统中所有可能的历史文件路径
- `find / -maxdepth 4 -type f -name '.*history' 2>/dev/null`

#### 输出系统中所有用户的 bash 历史文件
- `for f in /home/*/.bash_history /root/.bash_history; do [ -e "$f" ] && echo "===== $f =====" && tail -200 "$f"; done 2>/dev/null`

#### 输出数据库操作历史命令
- `grep -Ein '(^|[[:space:]])(mysql|mysqldump|psql|redis-cli|mongo|mongosh)[[:space:]]' ~/.bash_history /root/.bash_history 2>/dev/null`

### 2. 网络连接排查
#### ARP 攻击分析
- `ip neigh`
- `arp -an 2>/dev/null`

#### 网络连接分析
- `ss -antup`
- `netstat -antup 2>/dev/null`

#### TCP 端口检测
- `ss -lntp`
- `netstat -lntp 2>/dev/null`

#### TCP 高危端口
- `ss -lntp | awk 'NR>1{print $4}' | sed 's/.*://g' | sort -n | uniq`
- *AI 需自行判断常用高危端口，如 445, 3389, 6379, 2375 等*

#### UDP 端口检测
- `ss -lnup`
- `netstat -lnup 2>/dev/null`

#### DNS 信息排查
- `cat /etc/resolv.conf`
- `grep -v '^[[:space:]]*#' /etc/hosts`

#### 网卡工作模式
- `ip link | grep -E '^[0-9]+:|PROMISC'`

#### 网络路由信息排查
- `ip route show`
- `ip rule show`

#### 路由转发排查
- `sysctl net.ipv4.ip_forward`
- `sysctl net.ipv6.conf.all.forwarding`

#### 防火墙策略排查
- `iptables -S 2>/dev/null`
- `nft list ruleset 2>/dev/null`
- `firewall-cmd --list-all 2>/dev/null`

### 3. 进程排查
#### ps 进程分析
- `ps aux`
- `ps -ef`
- `ps aux --sort=-%cpu | head -20`
- `ps aux --sort=-%mem | head -20`

#### top 进程分析
- `top -b -n 1 | head -80`

#### 规则匹配敏感进程
- `ps -ef | egrep -i '(nc|ncat|netcat|socat|frp|ngrok|proxychains|ssh -[NRD])'`

#### 异常进程检测
- `ps -eo pid,ppid,user,tty,stat,lstart,cmd --sort=ppid | head -200`

#### 孤儿进程检测
- `ps -eo pid,ppid,cmd | awk '$2==1{print $0}' | head -50`

#### 网络连接和进程映射
- `ss -antup`
- `lsof -i -n -P 2>/dev/null | head -200`

#### 进程可疑内存映射
- `pmap -x <PID> 2>/dev/null | head -80` (针对可疑 PID)
- `cat /proc/<PID>/maps 2>/dev/null | head -80`

#### 文件描述符异常进程
- `ls -l /proc/<PID>/fd 2>/dev/null | head -50`

#### 进程启动时间异常检测
- `ps -eo pid,lstart,cmd --sort=lstart | head -50`
- `ps -eo pid,lstart,cmd --sort=-lstart | head -50`

#### 进程环境变量异常检测
- `tr '\0' '\n' < /proc/<PID>/environ 2>/dev/null | head -80`

### 4. 文件排查
#### 系统服务收集
- `systemctl list-unit-files --type=service 2>/dev/null | head -200`

#### 系统自启动服务分析
- `systemctl list-unit-files --type=service 2>/dev/null | grep enabled`
- `cat /etc/rc.local 2>/dev/null`

#### 系统正在运行的服务分析
- `systemctl --type=service --state=running 2>/dev/null | head -200`

#### 用户服务分析
- `systemctl --user list-unit-files --type=service 2>/dev/null | head -200`

#### /tmp 目录
- `ls -alh /tmp /var/tmp /dev/shm 2>/dev/null | head -200`

#### /root 目录隐藏文件
- `ls -al /root 2>/dev/null | grep '^\.' || true`

#### .ssh 目录排查
- `ls -al ~/.ssh 2>/dev/null`
- `ls -al /root/.ssh 2>/dev/null`

#### 公钥私钥排查
- `ls -al ~/.ssh/id_* /root/.ssh/id_* 2>/dev/null`
- `ssh-keygen -lf ~/.ssh/id_rsa.pub 2>/dev/null`

#### authorized_keys 文件排查
- `cat ~/.ssh/authorized_keys 2>/dev/null`
- `cat /root/.ssh/authorized_keys 2>/dev/null`

#### known_hosts 文件排查
- `cat ~/.ssh/known_hosts 2>/dev/null | tail -50`

#### sshd_config 文件分析
- `grep -v '^[[:space:]]*#' /etc/ssh/sshd_config 2>/dev/null`

#### 环境变量文件分析
- `cat /etc/profile 2>/dev/null`
- `ls -la ~/.bashrc ~/.bash_profile 2>/dev/null`

#### env 命令分析
- `env | sort | head -200`

#### hosts 文件排查
- `cat /etc/hosts`

#### shadow/gshadow 文件权限与属性
- `ls -l /etc/shadow /etc/gshadow 2>/dev/null`
- `stat /etc/shadow /etc/gshadow 2>/dev/null`
- `lsattr /etc/shadow /etc/gshadow 2>/dev/null`

#### 24 小时变动文件排查
- `find / -mtime -1 -type f 2>/dev/null | head -200`

#### SUID/SGID 文件排查
- `find / -perm -4000 -type f 2>/dev/null | head -200`
- `find / -perm -2000 -type f 2>/dev/null | head -200`

### 5. 日志排查
#### message 日志分析
- `grep -Ein 'rz|sz|ZMODEM' /var/log/messages* 2>/dev/null | head -50`
- `grep -Ein 'named|dnsmasq|resolv|DNS' /var/log/messages* 2>/dev/null | head -50`

#### secure/auth 日志分析
- `grep -Ein 'Accepted |Failed password|Invalid user' /var/log/secure* /var/log/auth.log* 2>/dev/null | head -80`
- `grep -Ein 'useradd|userdel|usermod|groupadd|groupdel' /var/log/secure* /var/log/auth.log* 2>/dev/null | head -80`

#### 计划任务日志分析 (cron)
- `grep -Ein 'CRON|cron' /var/log/cron* /var/log/messages* /var/log/syslog* 2>/dev/null | head -120`

#### yum/dnf 日志分析
- `grep -Ein 'Installed:|Updated:|Erased:' /var/log/yum.log* /var/log/dnf.log* 2>/dev/null | tail -200`

#### dmesg 日志分析
- `dmesg | tail -200`

#### btmp / lastlog / wtmp 日志分析
- `lastb -n 50 2>/dev/null`
- `lastlog | head -200`
- `last -n 50`

#### journalctl 工具日志分析
- `journalctl --since "24 hours ago" --no-pager | tail -200`

#### auditd 服务状态
- `systemctl status auditd 2>/dev/null | head -120`

#### rsyslog 配置文件
- `cat /etc/rsyslog.conf 2>/dev/null`
- `ls -la /etc/rsyslog.d 2>/dev/null`

### 6. 后门排查
#### LD_PRELOAD 环境变量劫持检测
- `echo $LD_PRELOAD`
- `cat /etc/ld.so.preload 2>/dev/null`

#### PROMPT_COMMAND 后门检测
- `echo $PROMPT_COMMAND`

#### Alias 别名后门检测
- `alias` (当前 shell)
- `cat ~/.bashrc /etc/bashrc /etc/profile | grep alias`

#### SSH Wrapper 后门检测
- `file /usr/sbin/sshd`
- `stat /usr/sbin/sshd`
- *解读：检查 sshd 是否为脚本文件或修改时间异常*

#### 系统启动脚本后门
- `cat /etc/rc.local 2>/dev/null`
- `ls -alt /etc/init.d/ | head -20`

### 7. 隧道检测
#### SSH 隧道检测
- `ss -antp | grep sshd`
- `ps -ef | egrep -i 'ssh .*(-L|-R|-D)[[:space:]]'`
- `ps -ef | egrep -i '(autossh|ssh .*ServerAliveInterval|ssh .*ControlMaster)'`

#### 常见隧道工具检测 (HTTP/DNS/ICMP/Other)
- `ps -ef | egrep -i '(reGeorg|neo-regeorg|dnscat|iodine|icmptunnel|ptunnel|frp|nps|ngrok|chisel|gost|ncat|socat)'`
- *解读：命中任何关键词需立即检查该进程的二进制文件位置与网络连接*

### 8. Webshell 排查
#### 最近修改的 Web 文件
- `find /var/www /home/wwwroot -type f -mtime -3 2>/dev/null | head -100` (路径需根据实际 web 根目录调整)

#### 包含危险函数的 PHP 文件
- `grep -rE "eval\(|base64_decode\(|gzinflate\(|assert\(|system\(|passthru\(|shell_exec\(|popen\(|proc_open\(" /var/www /home/wwwroot 2>/dev/null | head -100`

#### 包含危险函数的 JSP 文件
- `grep -rE "getRuntime\(|exec\(|ProcessBuilder" /var/www /home/wwwroot 2>/dev/null | grep ".jsp" | head -100`

#### 包含危险函数的 ASP/ASPX 文件
- `grep -rE "eval\(|execute\(|response.write" /var/www /home/wwwroot 2>/dev/null | grep -E ".asp|.aspx" | head -100`

#### 隐藏的 Web 文件
- `find /var/www /home/wwwroot -name ".*" 2>/dev/null`

### 9. 病毒排查
#### 异常资源占用检测 (挖矿)
- `top -b -n 1 | awk '$9 > 50 {print $0}'` (CPU 占用 > 50% 的进程)

#### 常见恶意目录可执行文件检测
- `find /tmp /var/tmp /dev/shm -type f -executable 2>/dev/null`

#### 知名病毒/挖矿进程特征匹配
- `ps -ef | egrep -i '(xmrig|minerd|cpuminer|suppoie|kworkerds|kinsing|kdevtmpfsi)'`

### 10. 内存排查
#### 进程内存映射异常检测
- `cat /proc/self/maps` (示例，需针对可疑 PID 执行 `cat /proc/<PID>/maps`)
- *解读：关注 `/tmp`, `/dev/shm` 路径的 `r-xp` 或 `rwxp` 映射*

#### 内存段权限异常检测
- `grep "rwx" /proc/*/maps 2>/dev/null | head -50`
- *解读：`rwx` (可读可写可执行) 内存段在现代程序中非常少见，通常是 shellcode 注入特征*

### 11. 黑客工具排查
#### 常见黑客工具进程匹配
- `ps -ef | egrep -i '(nmap|sqlmap|hydra|msfconsole|cobaltstrike|cs|teamserver|beacon|mimikatz|procdump)'`

#### 常见黑客工具文件残留
- `find / -name "sqlmap.py" -o -name "nmap" -o -name "hydra" -o -name "mimikatz.exe" 2>/dev/null | head -20`

### 12. 内核排查
#### 内核模块异常检测
- `lsmod | head -50`
- `cat /proc/modules | head -50`

#### 隐藏内核模块检测
- *比较 `lsmod` 输出与 `/sys/module` 目录内容（需 AI 逻辑判断，较难通过单条命令完成，可建议用户关注差异）*

### 13. 其他排查
#### 可疑脚本文件排查
- `find /tmp /var/tmp /dev/shm -type f \( -name '*.sh' -o -name '*.py' -o -name '*.pl' -o -name '*.elf' \) 2>/dev/null`

#### 系统文件完整性校验
- `rpm -Va` (RPM 系，检查所有包文件更改)
- `dpkg --verify` (Debian 系)

### 14. Kubernetes 排查
#### 集群节点状态
- `kubectl get nodes -o wide`

#### 异常 Pod 排查
- `kubectl get pods --all-namespaces -o wide | grep -v 'Running\|Completed'`
- `kubectl get pods --all-namespaces -o jsonpath='{range .items[?(@.spec.hostNetwork==true)]}{.metadata.namespace} {.metadata.name} {.spec.containers[*].image}{"\n"}{end}'` (检查开启 hostNetwork 的 Pod)

#### 敏感权限绑定
- `kubectl get clusterrolebinding | grep 'admin\|system:master'`

### 15. 系统性能分析
- `df -h`
- `top -b -n 1 | head -20`
- `free -h`
- `uptime`
- `ip -s link`

### 16. 基线检查
- `cat /etc/login.defs | grep PASS`
- `cat /etc/pam.d/system-auth`
- `awk -F: '($2=="" || $2=="!" || $2=="*"){print $1}' /etc/shadow` (锁定或空密码账户)
- `ls -l /etc/passwd /etc/shadow /etc/group` (检查关键文件权限)
