# Linux 命令

## 系统监控

```bash
# 查看进程
ps -aux 
# 查看指定进程中的线程数
ps -Lf <pid>
# 查看父进程，进程组
ps -ef
# 查看内存使用
free -h 
# 实时查看内存使用，以下是交互命令
# c：切换显示命令名称和完整命令行；
# M：根据驻留内存大小进行排序；
# P：根据CPU使用百分比大小进行排序；
top 
# 查看虚拟内存，2 指间隔多少秒输出，（-SM）-S 指定单位 M
# 输出格式：第一行永远输出开机后平均值，以后的行输出当前值
vmstat 2 -SM
# 查看磁盘使用
df -h 
# 显示当前目录下所有文件和其大小
du -h 
# 查看开机时间, 登录用户数, 负载(最近5min, 10min, 15min)和当前登录用户
w 
# 查看系统各种设置, 如页大小, 最大打开文件数
ulimit -a 
# 查看内核版本
uname -r 
# 打印环境变量
printenv 
# 查看日历
cal 
# 查看时间
date 
# 查看当前时区
timedatectl
# 查看当前编码
locale
# 查看最近10条登录记录
last -10 
# 查看关机记录
last -x | grep shutdown 
# 查看最近10条失败的登录记录
lastb -10
# 查看 CPU 温度，配合 `watch` 可实时监控
sensors
# 实时监测 CPU 的频率
watch grep \"cpu MHz\" /proc/cpuinfo
# 查看 CPU 设置的频率信息
cpupower frequency-info
```

## 查看文件

```bash
# 查看文件详细信息
stat <file> 
# 查看文件类型
file <file> 
# 查看文件16进制, 比od更好用
xxd <file> 
# 查看文本文件信息, 分别对应行数, 字数, 字节数
wc <文本文件> 
# 查看动态链接库
ldd <程序名> 
```

## 网络相关

### ip 命令

```bash
# 查看所有IP
ip addr 
# 查看所有IPV6
ip -6 addr
# 查看指定网卡ip
ip addr show <网卡名> 
# 查看路由
ip route show 
# 查看已激活网络接口的流量信息
ip -s link show up 
# 查看接入你所在的局域网的设备的MAC地址
ip neighbour 
```

### ss 命令

```bash
# 语法
ss <选项>

# 选项
-h # 显示帮助信息
-V # 显示指令版本信息
-n # 不解析服务名称, 以数字方式显示
-a # 显示所有的套接字
-l # 显示处于监听状态的套接字
-o # 显示计时器信息
-m # 显示套接字的内存使用情况
-p # 显示使用套接字的进程信息
-i # 显示内部的TCP信息
-4 # 只显示ipv4的套接字
-6 # 只显示ipv6的套接字
-t # 只显示tcp套接字
-u # 只显示udp套接字
-d # 只显示DCCP套接字
-w # 仅显示RAW套接字
-x # 仅显示UNIX域套接字

# 实例
ss -lntp # 查看监听的端口
ss -antp # 查看所有建立的tcp连接
```

### 其他命令

```bash
# 查看网卡信息
ifconfig 
# 查看无线网卡信息
iwconfig 
# 查看arp缓存
arp -a 
# 查看DNS服务器地址
cat /etc/resolv.conf
# 查询主机IP地址, 也可以反向查询
nslookup <URL|IP>
# 测试网络, -c指定ping次数
ping -c4 <URL> 
# 路由追踪
tracepath 
# 查看出站入站流量
nload 
# 查看进程联网情况
nethogs
# 请求一个网页, 不会处理重定向
curl <URL>
# 下载内容
wget <URL>
# 取得域名 dns 信息
dig <domain>
# 拷贝远程主机文件到本机
# 也可以拷贝本机文件到远程主机
scp <远程用户名>@<IP>:<目录> <本地路径>
```

## 进程管理

```bash
# Ctrl+z会使当前进程挂起
# 此时可用`jobs`命令查看
# 可以用`fg [jobs号]`恢复到前台
# 也可用`bg [jobs号]`使进程转入后台执行

# 让程序在后台执行, 并重定位输出, 不会随用户注销而结束
nohup python3 fuck.py > /dev/null &
# 发送信号给指定pid进程, 默认15号信号
kill [-signum] <pid> 
```

## 服务管理

```bash
# 查看所有服务单元
systemctl list-unit-files --type service
# 查看所有正在运行的服务单元
systemctl list-units --type=service
# 启动, 重启和结束服务
systemctl <start|restart|stop> <服务名> 
# 开机启动开关
systemctl <enable|disable> <服务名> 
# 查看服务状态
systemctl status <服务名>
# 从结尾处(-f)开始查看指定服务日志
journalctl -f <服务名> 
```

## 用户管理

```bash
# 切换用户, 加上-会使shell环境也切换
su - <用户名>
# 指定用户(-u)执行命令, nologin用户可以执行
sudo -u <用户名> <命令>
# 指定用户执行命令, nologin用户无法执行
su - <用户名> -c <命令>
# 创建用户, -m自动创建家目录
useradd -m <用户名>
# 创建用户, 详细
# -s指定shell
# -M不创建家目录
# -g指定组名
useradd -s /sbin/nologin -M -g mysql mysql
# 删除用户, -r自动删除家目录
userdel -r <用户名> 
# 修改用户密码
passwd <用户名> 
# 创建组
groupadd <组名> 
# 删除组
groudel <组名> 
# 查看用户所属组
groups <用户名> 
# 将用户添加到该组
usermod -a -G <组名> <用户名> 
# 将用户从组中删去
gpasswd -d <用户名> <组名> 
# 修改文件所有者, -R递归
chown <用户名> <文件名> 
# 修改文件所属组, -R递归
chgrp <组名> <文件名> 
# 修改文件权限, -R递归
chmod 644 <文件名> 
```

## 查找

```bash
# 根据内容查找文件(r), 忽略大小写(i), 输出行号(n)
grep -rin "查找内容" <目录>
# 精确匹配内容
grep -w "查找内容" <文件名> 
# 根据文件名查找
find <目录> -name <文件名> 
# 根据大小查找, +大于, -小于
find <目录> -size +10k 
# 根据类型查找
find <目录> -tye <类型> 

# linux文件类型
	# 普通文件: -
	# 目录: d
	# 链接符号: l
	# 块设备: b
	# 字符设备: c
	# socket文件: s
	# 管道: p
```

## 压缩解压缩

- tar.gz 格式

```bash
# 压缩
tar -zcvf <打包后文件名.tar.gz> <打包文件夹名>
# 解压缩
tar -zxvf <文件名.tar.gz>
```

- tar.bz2 格式

```bash
# 压缩
tar -jcvf <打包后文件名.tar.bz2> <打包文件夹名>
# 解压缩
tar -jxvf <文件名.tar.bz2>
```

- zip 格式

```bash
# 压缩
zip  <打包后文件名.zip> <打包文件夹名>
# 解压缩, 需安装 unzip
unzip <文件名.zip>
```

- rar 格式

```bash
# 压缩
rar a <打包后文件名.rar> <打包文件夹名> # a 指压缩
# 解压缩
rar x <文件名.rar> # x 指解压
```

## 定时任务

首先必须启动守护进程（进程名叫 crond）：

```bash
systemctl start cronie 
```

几个命令：

```bash
# 创建定时任务，相当于修改"/var/spool/cron/<用户名>"文件
crontab -e 
# 查看定时任务
crontab -l 
# 删除定时任务
crontab -r 
```

定时任务格式：

```bash
* * * * * echo "hello, world"
```

每颗 `*` 分别代表：分钟（0-59），小时（0-23），日期（1-31），月份（1-12），星期（0-6）。此外还有几个特殊符号：`/` 代表每的意思，`*/5` 表示每 5 个单位，`-` 代表从某个数字到某个数字，`,` 分开几个离散的数字。

实例：

```bash
# 每1分钟执行一次command
* * * * * command
# 每小时的第3和第15分钟执行
3,15 * * * * command
# 在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * command
# 每隔两天的上午8点到11点的第3和第15分钟执行
3,15 8-11 */2 * * command
```

## 其他命令

```bash
# 查看命令所在目录
which <命令> 
# 查看命令所有关联目录, which强化版
whereis <命令> 
# 立刻关机
shutdown -h now 
# 10分钟后关机
shutdown -h 10 
# 10分钟后重启
shutdown -r 10 
# 踢掉指定用户终端
pkill -t <tty>
```

