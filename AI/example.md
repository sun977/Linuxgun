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
1. **用户排查**: 关注 UID=0 的非 root 用户、新建用户、异常 shell
2. **进程排查**: 关注高 CPU/内存占用、异常进程名、隐藏进程、挖矿特征
3. **网络排查**: 关注外连 IP、异常端口、反弹 shell 特征
4. **文件排查**: 关注近期修改文件、SUID/SGID 文件、隐藏文件、webshell
5. **计划任务**: 关注异常定时任务、base64 编码命令、外连下载
6. **启动项**: 关注异常服务、rc.local 修改、bashrc 后门
7. **日志分析**: 关注暴力破解、异常登录 IP、sudo 提权
8. **后门检测**: 关注 LD_PRELOAD、SSH 公钥、rootkit 特征

### 威胁等级标注
- 🔴 高危: 确认存在入侵痕迹或后门
- 🟡 中危: 存在可疑项需进一步确认
- 🟢 低危/正常: 未发现明显异常

## 排查流程建议
连接成功后，询问用户选择排查方向：
1. 快速全面排查（自动执行关键检查项并汇总分析）
2. 用户登录排查
3. 进程排查
4. 网络连接排查
5. 文件排查
6. 计划任务排查
7. 启动项排查
8. 日志分析
9. 后门检测
10. 自定义命令

## 常用应急响应命令

### 1. 用户与登录排查
- `whoami` - 当前用户
- `w` - 当前登录用户
- `last` - 登录历史
- `lastb` - 失败登录记录
- `cat /etc/passwd` - 查看所有用户
- `cat /etc/shadow` - 查看密码文件
- `awk -F: '$3==0{print $1}' /etc/passwd` - 查找 UID=0 的用户
- `cat /etc/sudoers` - sudo 权限配置

### 2. 进程排查
- `ps aux` - 所有进程
- `ps aux --sort=-%cpu | head -20` - CPU 占用最高的进程
- `ps aux --sort=-%mem | head -20` - 内存占用最高的进程
- `top -b -n 1` - 系统资源概览
- `lsof -i` - 网络连接的进程
- `lsof -p <PID>` - 指定进程打开的文件

### 3. 网络排查
- `netstat -antlp` - 所有网络连接
- `netstat -antlp | grep ESTABLISHED` - 已建立的连接
- `netstat -antlp | grep LISTEN` - 监听端口
- `ss -antlp` - 套接字统计
- `iptables -L -n` - 防火墙规则

### 4. 文件排查
- `find / -mtime -1 -type f 2>/dev/null` - 最近1天修改的文件
- `find / -ctime -1 -type f 2>/dev/null` - 最近1天创建的文件
- `find / -perm -4000 -type f 2>/dev/null` - SUID 文件
- `find / -perm -2000 -type f 2>/dev/null` - SGID 文件
- `ls -alt /tmp /var/tmp /dev/shm` - 临时目录可疑文件

### 5. 计划任务排查
- `crontab -l` - 当前用户定时任务
- `cat /etc/crontab` - 系统定时任务
- `ls -la /etc/cron.*` - cron 目录
- `ls -la /var/spool/cron/` - 用户 cron 文件
- `systemctl list-timers` - systemd 定时器

### 6. 启动项排查
- `systemctl list-unit-files --type=service | grep enabled` - 已启用服务
- `cat /etc/rc.local` - rc.local 启动脚本
- `ls -la /etc/init.d/` - init.d 脚本
- `ls -la ~/.bashrc ~/.bash_profile /etc/profile` - shell 启动文件

### 7. 日志排查
- `tail -100 /var/log/auth.log` - 认证日志 (Debian/Ubuntu)
- `tail -100 /var/log/secure` - 认证日志 (CentOS/RHEL)
- `tail -100 /var/log/syslog` - 系统日志
- `journalctl -xe` - systemd 日志

### 8. 后门排查
- `cat /etc/ld.so.preload` - LD_PRELOAD 劫持
- `echo $LD_PRELOAD` - 环境变量劫持
- `cat ~/.ssh/authorized_keys` - SSH 公钥后门
- `cat /root/.ssh/authorized_keys` - root SSH 公钥
- `strings /usr/bin/sshd | grep -i password` - sshd 后门检测

### 9. 历史命令
- `cat ~/.bash_history` - bash 历史
- `cat /root/.bash_history` - root bash 历史
