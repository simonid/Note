### ssh连接提示“Connection closed by remote host”
通常是连接数过多导致的<br>
查看有多少用户连接ssh：`who`
断开某个用户：`ps aux | grep sshd`，获得pid后`kill`掉
<ol>
<li>方法一：将ssh连接数增大</li>
编辑/etc/ssh/sshd_config，并且将里面的MaxSessions 数值改大，然后重启ssh
<li>正常退出ssh连接</li>
每次退出的时候输入"exit"
<li>重启</li>
</ol>

### ssh连接提示connection is closed by foreign host
大致前后是这样的，本人买了LA的一个节点，但是被ban了，然后不知为什么挂了美国节点的代理都不能ssh访问，挂其他地区倒是可以<br>
查看了`/etc/hosts.deny`也没有禁止我的代理IP，然后在`/etc/hosts.allow`中添加我的代理IP一样不管用<br>
打开`/var/log/auth.log`日志：
```
Oct 23 06:25:03 LightgreyTerrific-VM CRON[1544]: pam_unix(cron:session): session closed for user root
Oct 23 06:25:04 LightgreyTerrific-VM sshd[1699]: error: Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Oct 23 06:25:04 LightgreyTerrific-VM sshd[1699]: error: Could not load host key: /etc/ssh/ssh_host_ed25519_key
```
自己添加的IP居然不能加载？？？<br>
参考了网上一篇教程，大致内容：
```
修改配置文件

/etc/ssh/sshd_config

去掉注释
ClientAliveInterval 60
ClientAliveCountMax 3

然后重启 sshd 服务

service sshd restart

发现出错信息
bad permissions : ignore key :/etc/ssh/ssh_host_rsa_key

bad permissions : ignore key :/etc/ssh/ssh_host_dsa_key

授权：

chmod 700 /etc/ssh/ssh_host_rsa_key
chmod 700 /etc/ssh/ssh_host_dsa_key

service sshd restart
```
然而最后还是不能连接<br>
后查看另外一篇，大致：
```
ssh 在主机上直连报错信息：
ssh_exchange_identification: read: Connection reset by peer
意思是 断开主机链接了，出现这种问题，跟你的IPTABLES，防火墙什么的都没关系。

造成这个原 因是因为原来连接到SSHD服务器进程的22端口，当你的客户端突然断开时，服务器端的TCP连接就处于一个半打开状态。当下一次同一客户机再次建立 TCP连接时，服务器检测到这个半打开的TCP连接，并向客户机回传一个置位RST的TCP报文，客户机就会显示connection closed by foreign host。
这是TCP协议本身的一个保护措施，并不是什么错误，但重新连接还是无法连上。

后查看，自己的网卡设置，主机有两张网卡，且分别都配置了网关，取消一个网关即可以连接上。

或者启用iptables配置os层面的系统路由策略
```
上述:http://www.360doc.com/content/14/1208/19/17673261_431357070.shtml<br>

### 关于取消auth.log日志下的”pam_unix(cron:session): session opened for user root by (uid=0)“信息
其实这段日志内容也不是什么危险警告，只是有登出的 root在背景時，`Crontab` 检查而已，但是这样的内容会重复很多。可以用下面方法屏蔽<br>
在`/etc/pam.d `中找到`common-session-noninteractive`
在上面加一行：
`session [success=1 default=ignore] pam_succeed_if.so service in cron quiet use_uid`
然后重启cron<br>

#### ping出现ping: unknown host的错误
首先检查能否ping到网关地址，如果可以，那么查看`/etc/resolve.conf`内的配置，加入DNS规则如：
```
nameserver 8.8.8.8
```

#### Linux TAB键命令补全失效后者出现乱码
出现这种情况是因为命令解析出错<br>
编辑`/etc/passwd`，找到当前用户<br>
配置文件格式：
```
   （1）：用户名。

   （2）：密码（已经加密）

   （3）：UID（用户标识）,操作系统自己用的

   （4）：GID组标识。

   （5）：用户全名或本地帐号

   （6）：开始目录

   （7）：登录使用的Shell，就是对登录命令进行解析的工具。
```
比如`user:x:1001:1001::/home/user:/bin/sh `，将其修改为：
`user:x:1001:1001::/home/user:/bin/bash`

