刚刚购买的VPS默认使用root账户通过ssh的22端口登录，如果只是使用这样的默认配置，那么我们的VPS很容易被暴力破解，成为肉鸡,如果想知道被暴力破解的情况，可以输入：
```
sudo grep "Failed password for root" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | more
sudo grep "Failed password for invalid user" /var/log/auth.log | awk '{print $13}' | sort | uniq -c | sort -nr | more
```
然后就再终端中显示出试图暴力破解你vps的IP<br>
下面介绍一下vps的安全防范<br>
### 创建普通用户
管理员登录身份最好不要是root，使用普通账户为好。<br>
```
useradd -m username
passwd username
或者useradd username //这样后续就会让你输入密码和其他信息
```
<b>给普通用户添加sudo权限</b><br>
```
echo -e "\nusername ALL=(ALL) ALL\n" >> /etc/sudoers
tail -3 /etc/sudoers //检查
```

#### 解决tab不能补全，上下不能切换历史命令
这种情况常出现在新增用户并切换到该用户的执行环境，同时，用户名和主机名也不会出现，只会显示一个"$"符号。<br><br>
这是由于/etc/passwd里新增用户的shell和root用户不一样导致的，root使用/bin/bash，新增用户默认为/bin/sh。<br>
<br>
查看:<br>
```
ls -l /bin/sh
返回/bin/sh -> dash
```
修改：<br>
```
cd /bin
sudo ln -sf bash /bin/sh
```
检查：<br>
```
ls -l /bin/sh
返回/bin/sh -> bash
```


### 创建ssh密钥对
备注：centos的ssh服务名为`sshd`，Ubuntu是`ssh`<br>
切换到普通用户：
```
su -l username
```
如果出现`sudo: unable to resolve host xxx `：
<ol>
<li>查看hostname</li>
# head /etc/hostname<br>
    ubuntu<br>
<li>修改文件 /etc/hosts，增加一行内容 127.0.0.1 hostname:</li>
# echo -e "\n127.0.0.1 ubuntu\n" >> /etc/hosts<br>
# tail -3 /etc/hosts<br>
<br>
127.0.0.1 ubuntu
</ol>

生产密钥：
```
ssh-keygen
```
会输出的选项：<br>
`Enter file in which to save the key`：选择密钥目录的位置,保持默认就好<br>
然后要输入密钥文件的密码(二次验证)<br>

再执行：
```
$ cd ~/.ssh 
$ cat id_rsa.pub >> authorized_keys
$ ls 
authorized_keys  id_rsa  id_rsa.pub
$ chmod 400 authorized_keys
$ chmod 700 ./.ssh

如果要清空authorized_keys的内容，可以在vi命令模式下输入ggdG清空全部
```
`.ssh`目录下的id_rsa是客户端登录的私钥，我们需要将它从服务端下载到本地<br>
再客户端终端中输入：<br>
```
$ get id_rsa
```
这时私钥就安装到本地终端<br>
然后编辑ssh配置文件
```
# vi /etc/ssh/sshd_config
将Port 22改成其他端口
PermitRootLogin no #禁止root登录
RSAAuthentication yes #RSA认证
PubkeyAuthentication yes #开启公钥验证
AuthorizedKeysFile %h/.ssh/authorized_keys #验证文件路径(当前登录用户下的)
PasswordAuthentication no #禁止密码认证
PermitEmptyPasswords no #禁止空密码
# 最后保存，重启
/etc/init.d/ssh restart
```

如果是有多个VPS需要这么做，可以设置所有的VPS公私钥都一致：<br>
首先，打开第一台VPS的公钥authorized_keys，将其内容复制到第二台的公钥中，然后，第二台就能使用和第一台一样的私钥(即id_rsa)了。<br>
<br>
注意:一旦设置了禁止root登录，就算配了密钥对，也不可以登录。并且，用户需要注意密钥目录的位置和权限<br>


这里补充一下过去踩过的坑，有些VPS不自带SSH，需要用户自己安装，centos的例子：<br>
```
#rpm -qa |grep ssh 检查是否装了SSH包
没有的话yum install openssh-server
#chkconfig --list sshd 检查SSHD是否在本运行级别下设置为开机启动
#chkconfig --level 2345 sshd on  如果没设置启动就设置下.
#service sshd restart  重新启动
#netstat -antp |grep sshd  看是否启动了22端口.确认下.
#iptables -nL  看看是否放行了22口.
#setup---->防火墙设置   如果没放行就设置放行.
```

### 配置防火墙
安装：
```
$ sudo apt-get install iptables             # ubuntu
$ yum install iptables                      # centos
```
对于 centos/redhat（适用 centos 5, centos 6）：
```
$ sudo vi /etc/sysconfig/iptables

# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT   # 如果有修改 sshd 服务端口号，改为修改后的数字
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
-A INPUT -j DROP
COMMIT
```

Debian官方的建议配置：<br>
```
*filter

# Permette tutto il traffico su loopback (lo0) traffic e elimina tutto il traffico che non usa lo0 verso 127/8
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

# Accetta in entrata su tutte le connessioni stabilite
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Permette tutto il traffico in uscita
# Potrebbe essere modificato per permettero solo un certo tipo di traffico
-A OUTPUT -j ACCEPT

# Permette connessioni HTTP e HTTPS da qualsiasi parte provengano (le normali porte per i siti web)
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

# Permette le connessioni SSH
# Il numero --dport e' lo stesso di quello in /etc/ssh/sshd_config
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
## 注意这里，如果修改了ssh端口那么就要更改22为你的新端口，否则会拒绝连接

# Ora ci si dovrebbe informare sulle regole di iptables e considerare se l'accesso ssh
# per tutti sia realmente quello che si vuole. Molto probabilmente si preferisce 
# permettere l'accesso solo per alcuni IP.

# Permettere ping
# notare che bloccare altri tipi di pacchetti icmp è considerata da alcuni una cattiva idea
# rimuovere -m icmp --icmp-type 8 da questa riga per permettere tutti i tipi di icmp:
# https://security.stackexchange.com/questions/22711
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# registrare le chiamate negate di iptables (accesso via il comando 'dmesg')
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Respingere tutto il resto del traffico in entrata: politica del negare in modo predefinito quando non esplicitamente permesso
-A INPUT -j REJECT
-A FORWARD -j REJECT

COMMIT
```
补充说明：
```
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
## 注意这里，如果修改了ssh端口那么就要更改22为你的新端口，否则会拒绝连接
```

对于Ubuntu要稍微加点内容：<br>
```
$ sudo vi /etc/iptables.up.rules  # 添加上面的规则

$ sudo vim /etc/network/interfaces
```
并且在 `iface lo inet loopback` 后增加一行 `pre-up iptables-restore < /etc/iptables.up.rules`
应用防火墙规则：
```
$ sudo service iptables restart  # centos  5, 6

$ sudo iptables-restore < /etc/iptables.up.rules   # ubuntu
```
开机自启动防火墙：
```
$ sudo chkconfig iptables on  # redhat/centos  5, 6

$ sudo apt-get install sysv-rc-conf  # ubuntu
$ sudo sysv-rc-conf iptables on    # ubuntu
```
更多防火墙命令说明:
```
# 清除已有iptables规则
iptables -F
# 允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -i lo -j ACCEPT
# 允许已建立的或相关连的通行
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
#允许所有本机向外的访问
iptables -A OUTPUT -j ACCEPT
# 允许访问22端口，以下几条相同，分别是22,80,443端口的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#如果有其他端口的话，规则也类似，稍微修改上述语句就行
#允许ping
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
#禁止其他未允许的规则访问（注意：如果22端口未加入允许规则，SSH链接会直接断开。）
iptables -A INPUT -j REJECT 
iptables -A FORWARD -j REJECT
```

参考：[入手 VPS 后首先该做的事情](https://mozillazg.github.io/2013/01/linux-vps-first-things-need-to-do.html)<br>
[服务器VPS安全防护](https://zhuanlan.zhihu.com/p/26282070)
[Ubuntu UFW防火墙配置](https://www.logcg.com/archives/988.html)<br>
[购买了VPS之后你应该做足的安全措施](https://www.logcg.com/archives/884.html)<br>

#### 设置 Linux 上 SSH 登录的 Email 提醒

原文只针对`CentOS 6、 CentOS 7、 RHEL 6 和 RHEL 7`<br>
一、对全部用户生效<br>
```
[root@vps ~]# vi /etc/bashrc
[root@vps ~]# echo 'ALERT - Root Shell Access (vps.ehowstuff.com) on:' `date` `who` | mail -s "Alert: Root Access from `who | cut -d'(' -f2 | cut -d')' -f1`" recipient@gmail.com
```
二、只对root生效
```
[root@vps ~]# vi .bashrc
[root@vps ~]# echo 'ALERT - Root Shell Access (vps.ehowstuff.com) on:' `date` `who` | mail -s "Alert: Root Access from `who | cut -d'(' -f2 | cut -d')' -f1`" recipient@gmail.com
```
三、只对特定普通用户
```
[root@vps ~]# vi /home/skytech/.bashrc
[root@vps ~]# echo 'ALERT - Root Shell Access (vps.ehowstuff.com) on:' `date` `who` | mail -s "Alert: Root Access from `who | cut -d'(' -f2 | cut -d')' -f1`" recipient@gmail.com
```
最后文件的格式：
```
    # .bashrc
    # User specific aliases and functions
    alias rm='rm -i'
    alias cp='cp -i'
    alias mv='mv -i'
    # Source global definitions
    if [ -f /etc/bashrc ]; then
            . /etc/bashrc
    fi
    echo 'ALERT - Root Shell Access (vps.ehowstuff.com) on:' `date` `who` | mail -s "Alert: Root Access from `who | cut -d'(' -f2 | cut -d')' -f1`" recipient@gmail.com
```
不过，使用mail命令前先要配置信息：
<br>
一般高版本linux发行版都内置mailx这个应用<br>
可以通过`$ which mail`查看路径，
<br>
查看配置文件：
```
$ strings `which mail` | grep '\.rc'
```
然后修改配置文件，在末尾加入:
```
set from=TechForGeek@sina.com    smtp=smtp.sina.com
set smtp-auth-user=TechForGeek@sina.com    smtp-auth-password=TechForGeek    smtp-auth=login
```
测试：
```
$ mail -s "Subject" recepient@xxx
```
-s 选项后面跟的参数表示的是要发送的邮件的主题， recepient@xxx 表示收件人的邮箱地址。<br>
输入上面的命令并按下回车键后，你就可以在终端输入要发送的邮件的正文了，输入完正文后，按下 <Ctrl> + D 组合键。如果一切都配置正确的话，此时邮件就会发送出去了。<br>
不过最后执行的时候一直提示：`Could not connect: Operation now in progress
"/home/simon/dead.letter" 8/198
... message not sent`这样的错误，并且没收到邮件<br>
参考：[如何设置 Linux 上 SSH 登录的 Email 提醒 ](https://linux.cn/article-5334-1.html)<br>
另外一篇文章：[给VPS弄个SSH登录自动邮件提醒+短信提醒](https://www.youngfree.cn/seo/1324.html)<br>
[mailx配置](https://www.techforgeek.info/send_mail_from_terminal.html)<br>
### 使用工具
#### denyhosts
Denyhosts是一个Linux系统下阻止暴力破解SSH密码的软件，它的原理与DDoS Deflate类似，可以自动拒绝过多次数尝试SSH登录的IP地址，防止互联网上某些机器常年破解密码的行为，也可以防止黑客对SSH密码进行穷举<br>
[GitHub地址](https://github.com/denyhosts/denyhosts)<br>

<font color="red" size=4>简化版denyhost使用</font><br>
直接：`sudo apt-get install denyhost`<br>
这样，就不需要像下面那样复杂地使用了<br>

配置：<br>
将默认配置文件复制到`/etc`下：
```
cp denyhosts.conf /etc
sudo vim /etc/denyhosts.conf
```
权限设定：
```
cp daemon-control-dist daemon-control
sudo chown root daemon-control
sudo chmod 700 daemon-control
```
编辑文件：
```
sudo vim ./daemon-control
```
文件主要修改`denyhosts`的位置，可以通过命令`whereis denyhosts`获取它的路径：
```
DENYHOSTS_BIN   = "/usr/local/bin/denyhosts.py" //这时我的
```
修改这个守护程序的所属用户和权限<br>
另外，根据GitHub源仓库的提示，把文件的内容修改：
```
DENYHOSTS_BIN   = "/usr/local/bin/denyhosts.py" 

#默认/usr/sbin/denyhosts
#如果没有修改这里就会出现
starting DenyHosts: /usr/bin/env python /usr/sbin/denyhosts –daemon –config=/etc/denyhosts.conf
python: can’t open file ‘/usr/sbin/denyhosts’: [Errno 2] No such file or directory错误
如果使用默认配置也行，不过要麻烦点，先新建这个不存在的文件，然后 ln -s /usr/local/bin/denyhosts.py /usr/sbin/denyhosts

DENYHOSTS_LOCK  = "/var/lock/subsys/denyhosts"
DENYHOSTS_CFG   = "/etc/denyhosts.conf"
```
运行：
```
~/denyhosts/daemon-control start
```
编辑`/etc/rc.local`添加到开机自启：
```
~/denyhosts/daemon-control start
```

详细配置：
```
1 SECURE_LOG = /var/log/secure               #指定ssh日志文件
 2 HOSTS_DENY = /etc/hosts.deny               #记录阻止登陆系统IP的文件
 3 PURGE_DENY =                               #清理HOSTS_DENY文件的时间
 4 BLOCK_SERVICE  = sshd                      #在HOSTS_DENY中定义要阻止的服务
 5 DENY_THRESHOLD_INVALID = 5                 #系统不存在用户失败次数
 6 DENY_THRESHOLD_VALID = 10                  #除root外，系统存在用户失败次数
 7 DENY_THRESHOLD_ROOT = 1                    #root用户失败次数
 8 DENY_THRESHOLD_RESTRICTED = 1              #针对WORK_DIR下定义的限制用户名的失败次数
 9 WORK_DIR = /usr/share/denyhosts/data       #将deny的host或ip记录到WORK_DIR中
10 SUSPICIOUS_LOGIN_REPORT_ALLOWED_HOSTS=YES  #来自于allowed-hosts中的可以尝试，是否报告 
11 HOSTNAME_LOOKUP=YES                        #是否做域名反向解析
12 LOCK_FILE = /var/lock/subsys/denyhosts     #保证同时只有一个denyhosts程序运行的锁文件
13        
14 ADMIN_EMAIL = 123@456.789                  #设置管理员邮箱，系统开启了sendmail就会发邮件
15 SMTP_HOST = localhost                      #SMTP服务器
16 SMTP_PORT = 25                             #SMTP端口
17 SMTP_FROM = DenyHosts <nobody@localhost>   #通知邮件的发信人地址
18 SMTP_SUBJECT = DenyHosts Report            #发信的主题
19 AGE_RESET_VALID=5d                         #指定时间没有失败登陆记录，将此主机的失败计数重置为0，（不适用于root）
20 AGE_RESET_ROOT=25d                         #root用户的重置时间
21 AGE_RESET_RESTRICTED=25d                   #针对有限制用户的
22 AGE_RESET_INVALID=10d                      #针对无效用户的
23    
24 DAEMON_LOG = /var/log/denyhosts            #程序后台运行的日志记录
25  
26 DAEMON_SLEEP = 30s                         #每次读取日志的时间间隔
27 DAEMON_PURGE = 1h                          #清除机制在 HOSTS_DENY 中终止旧条目的时间间隔
```

#### fail2ban
fail2ban是一个方式暴力破解密码的工具
```
sudo apt-get install fail2ban
```
详细配置（本人用默认）：
[在Ubuntu中用Fail2Ban保护SSH](http://blog.topspeedsnail.com/archives/262)<br>

补充配置fail2ban:<br>
复制一份本地配置文件<br>
```
sudo mv /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
编辑jail.local:<br>
```
# 以空格分隔的列表，可以是 IP 地址、CIDR 前缀或者 DNS 主机名
# 用于指定哪些地址可以忽略 fail2ban 防御
ignoreip = 127.0.0.1 172.31.0.0/24 10.10.0.0/24 192.168.0.0/24
 
# 客户端主机被禁止的时长（秒）
bantime = 86400
 
# 客户端主机被禁止前允许失败的次数 
maxretry = 5
 
# 查找失败次数的时长（秒）
findtime = 600
 
mta = sendmail
 
[ssh-iptables]
enabled = true
filter = sshd
action = iptables[name=SSH, port=ssh, protocol=tcp]
sendmail-whois[name=SSH, dest=your@email.com, sender=fail2ban@email.com]
# Debian 系的发行版 
logpath = /var/log/auth.log
# Red Hat 系的发行版
logpath = /var/log/secure
# ssh 服务的最大尝试次数 
maxretry = 3
```
修改后重启：<br>
```
sudo service fail2ban restart
或
sudo systemctl restart fail2ban
```
验证fail2ban是否开启：<br>
```
$ sudo fail2ban-client ping
Server replied: pong #开启返回信息
```
查看日志：<br>
```
$ sudo tail -f /var/log/fail2ban.log
```

参考：[Linux中国](https://linux.cn/article-5067-1.html)<br>

#### ddos deflate
该工具是一个防DDOS的工具<br>
下载：
```
wget http://www.inetbase.com/scripts/ddos/install.sh
```
安装：
```
sudo chmod 700 install.sh
sudo ./install.sh
```
安装完成后会弹出一堆说明<br>
修改配置（本人用默认）：
```
sudo vim /usr/local/ddos/ddos.conf
```
配置信息
```
##### Paths of the script and other files
PROGDIR="/usr/local/ddos"
PROG="/usr/local/ddos/ddos.sh"
IGNORE_IP_LIST="/usr/local/ddos/ignore.ip.list"  //IP地址白名单
CRON="/etc/cron.d/ddos.cron"    //定时执行程序
APF="/etc/apf/apf"
IPT="/sbin/iptables"
##### frequency in minutes for running the script
##### Caution: Every time this setting is changed, run the script with --cron
#####          option so that the new frequency takes effect
FREQ=1   //检查时间间隔，默认1分钟
##### How many connections define a bad IP? Indicate that below.
NO_OF_CONNECTIONS=150     //最大连接数，超过这个数IP就会被屏蔽，一般默认即可
##### APF_BAN=1 (Make sure your APF version is atleast 0.96)
##### APF_BAN=0 (Uses iptables for banning ips instead of APF)
APF_BAN=1        //使用APF还是iptables。推荐使用iptables,将APF_BAN的值改为0即可。
##### KILL=0 (Bad IPs are'nt banned, good for interactive execution of script)
##### KILL=1 (Recommended setting)
KILL=1   //是否屏蔽IP，默认即可
##### An email is sent to the following address when an IP is banned.
##### Blank would suppress sending of mails
EMAIL_TO="root"   //当IP被屏蔽时给指定邮箱发送邮件，把root换成自己的邮箱即可
##### Number of seconds the banned ip should remain in blacklist.
BAN_PERIOD=600    //禁用IP时间，默认600秒，可根据情况调整
```
修改一些问题：<br>
编辑`/usr/local/ddos/ddos.sh`,将`117行`的`netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr > $BAD_IP_LIST`替换为<br>
`netstat -ntu | grep ":" | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr > $BAD_IP_LIST`<br>
运行中错误：
```
./ddos.sh: 13: [: /usr/local/ddos/ddos.conf: unexpected operator
DDoS-Deflate version 0.6
Copyright (C) 2005, Zaf <zaf@vsnl.com>
$CONF not found.
```
将`/usr/local/ddos/ddos.sh`首行内容改为bash格式<br>
报错：
```
Failed to restart crond.service: Unit crond.service not found.
```
本人用Ubuntu，服务名是`cron`，所将前面提到的脚本中所有`crond`改为`cron`<br>
命令：
```
# 显示帮助
 ddos –h

#创建计划任务定期运行脚本 【启动】
ddos –c

# 阻止超过n个连接的所有IP地址。
 ddos -k  数量（默认150）
```
参考：<br>
[ddos deflate 安装到使用详解](http://www.jianshu.com/p/f1e44408c195)<br>


#### 系统安全检测工具Rkhunter
Rkhunter是一个非常全面的安全工具，具体能实现什么，可以检查服务器是否感染了病毒或可疑文件，对root权限进行检测。<br>
网站：<br>http://rkhunter.sourceforge.net/<br>

依赖安装：
```
sudo apt-get install binutils libreadline5 libruby ruby ruby ssl-cert unhide.rb mailutils
```

下载安装包并安装
```
wget http://downloads.sourceforge.net/project/rkhunter/rkhunter/1.4.4/rkhunter-1.4.4.tar.gz

tar xzvf rkhunter*

cd rkhunter*

sudo ./installer.sh --layout /usr --install
```

检查更新
```
sudo rkhunter --update
```

检查
```
sudo rkhunter --check
```

跳过前面检查中的回车
```
/usr/bin/rkhunter --checkall --skip-keypress 
```

创建定时任务
输入`sudo crontab -e`后输入：
```
0 3 * * 3 root /usr/bin/rkhunter --checkall --cronjob
可以先用whereis rkhunter确定其位置
```
指定每周三的3：00做一次检查<br>
 
常用功能<br>
```
–checkall (-c) :全系統檢測，rkhunter 的所有檢測項目
–createlogfile :建立登錄檔，一般預設放在 /var/log/rkhunter.log
–cronjob :可以使用 crontab 來執行，不會有顏色顯示
–report-warnings-only :僅列出警告訊息，正常訊息不列出！
–skip-application-check :忽略套件版本檢測(如果您已確定系統的套件已patch)
–skip-keypress :忽略按鍵後繼續的舉動(程式會持續自動執行)
–quiet :僅顯示有問題的訊息，比 –report-warnings-only 更少訊息
–versioncheck :檢測試否有新的版本在伺服器上
–update :更新 rkhunter 的資料庫來取得最新的資訊
```

参考:<br>
[鸟哥的教程](http://linux.vbird.org/linux_security/0420rkhunter.php)<br>
http://smallken.com/blog/2006/11/27/linux/34.html<br>


#### 权限枚举工具linenum
LinEnum是一款Linux文件枚举及权限提升检查工具<br>
[Github地址](https://github.com/rebootuser/LinEnum)<br>
安装:`wget -N --no-check-certificate https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
`<br>


#### 恶意软件扫描工具maldet
github仓库：https://github.com/rfxn/linux-malware-detect<br>
Maldet 也被称为 Linux Malware Detect（LMD）,用于处理共享托管环境中的常见威胁。 它使用来自网络边缘入侵检测系统的威胁数据来提取主动攻击的恶意软件，并生成用于检测的签名。<br>
软件的一大特点是可以自动处理受感染的文件和应用，并且也可以发送邮件警告用户<br>
<br>
安装最新版：<br>
```
wget http://www.rfxn.com/downloads/maldetect-current.tar.gz
tar -zxvf maldetect*
cd ./maldetect-*
sudo bash ./install.sh
```
执行过程中可能会遇到依赖问题而无法安装，那么执行：<br>
```
sudo apt-get install clamav -y
```
ClamAV做为LMD的扫描引擎。<br>
更新病毒库：`freshclam`<br>
file目录下有一个conf.maldet 配置文件，可以对其配置：<br>
```
将 email_alert 设置为 1
在 email_addr 添加你的电子邮件地址
将 email_ignore_clean 设置为 1，可以在自动清除恶意软件时忽略警报邮件。
将 quarantine_hits 设置为 1 可以自动隔离恶意软件和受影响的文件
将 quarantine_clean 设置为 1 可以自动清除受影响的文件（不推荐）
将 quarantine_suspend_user 设置为 1 可以暂停受影响的账户（不推荐）
```
手动扫描：`sudo maldet --scan-all /root`<br>
自动扫描：<br>
在 Maldet 的安装过程中，cronjob 功能也将安装到 /etc/cron.daily/maldet，这样就可以自动扫描主目录以及最近每天更改的任何文件和文件夹。<br>
扫描目录：<br>
```
maldet -a /document
```
查看扫描报告：<br>
```
maldet --report 161101-0651.14136  #maldet -e list 列出所有报告
```
监控一个目录：<br>
```
maldet -m /root
```
查看监控日志：<br>
```
tail -f /usr/local/maldetect/logs/inotify_log
```
定时任务：<br>
```
* 4 * * * /usr/local/sbin/maldet -b -a /root
```
重启cron：
```
/etc/init.d/cron restart
```
还原被隔离的软件：<br>
```
sudo maldet –restore/-s FILENAME
```


参考：<br>
[如何使用Maldet检测和清除Linux中的恶意软件](https://www.sysgeek.cn/maldet/)<br>
[centos安装恶意软件检测工具MalDet(配置说明丰富)](https://chaihongjun.me/os/linux/214.html)<br>
[CentOS 7安装LMD杀毒软件](http://blog.topspeedsnail.com/archives/10192)<br>
[How to install and use Linux Malware Detect (LMD) with ClamAV on Ubuntu 16](https://www.globo.tech/learning-center/install-use-lmd-clamav-ubuntu-16/)<br>

<br><br>

[三种工具来扫描Linux服务器](https://www.howtoing.com/how-to-scan-linux-for-malware-and-rootkits)<br>
[推荐5款针对Ubuntu系统的最佳杀毒软件](https://www.sysgeek.cn/ubuntu-antivirus-program/)<br>