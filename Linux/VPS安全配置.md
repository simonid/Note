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
一路回车再执行：
```
$ cd ~/.ssh 
$ cat id_rsa.pub >> authorized_keys
$ ls 
authorized_keys  id_rsa  id_rsa.pub
$ chmod 400 authorized_keys
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
AuthorizedKeysFile .ssh/authorized_keys #验证文件路径
PasswordAuthentication no #禁止密码认证
PermitEmptyPasswords no #禁止空密码
# 最后保存，重启
/etc/init.d/ssh restart
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

参考：[入手 VPS 后首先该做的事情](https://mozillazg.github.io/2013/01/linux-vps-first-things-need-to-do.html)<br>
[服务器VPS安全防护](https://zhuanlan.zhihu.com/p/26282070)

### 使用工具
#### denyhosts
Denyhosts是一个Linux系统下阻止暴力破解SSH密码的软件，它的原理与DDoS Deflate类似，可以自动拒绝过多次数尝试SSH登录的IP地址，防止互联网上某些机器常年破解密码的行为，也可以防止黑客对SSH密码进行穷举<br>
[GitHub地址](https://github.com/denyhosts/denyhosts)<br>
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
