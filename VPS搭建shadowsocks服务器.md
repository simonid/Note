## VPS搭建shadowsocks服务器
参考网址：[逗逼根据地](doub.io)  有很多关于科学上网教程，但是因为最近网站被封得严，为了做好备份，我参考复制了部分内容，但都有注明<br>
首先购买一个VPS，本人使用virmach的vps(可以油管高清，但是晚上ssh访问吃力)，kvm  $1.25每月，也可以考虑搬瓦工，性价比高，搭建容易（官方就有中文教程）。还有digitalocean，领取github学生包可以免费使用一年

使用doub.io(https://doub.io/ss-jc60/)的一键搭建SSR环境的脚本不能给linux的ss使用，也尝试了网上很多的ss系列代理软件都不成功，所以开始自己搭建
客户端：
[ss-qt5(ubuntu17上会卡)](https://github.com/erguotou520/electron-ssr/releases)
建议用命令行项目：shadowsocks-libev

访问vps，可以在本地终端操作，也可以使用第三方ssh工具，但是觉得还是本地终端好用
```
ssh root@0.0.0.0   //后面的ip是vps的ip
```
然后输入秘码就能进去了<br>
可以参考：[如何在Linux服务器上配置SSH密钥验证](https://www.howtoing.com/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

### 服务端搭建
#### 下载相关工具
```
$ sudo apt-get  install python-setuptools && easy_install pip
$ pip install shadowsocks
```
上述安装完成后创建配置文件：
```
$ vi /etc/shadowsocks.json
```
内容如下：
```
{
    "server":"your_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"auooo.com",
    "timeout":300,
    "method":"aes-256-cfb",  
    "fast_open": false  //使用TCP_FASTOPEN, 参数选项true / false，一般保持默认即可。部分内核支持
}
```
#### 启动ss
```
ssserver -c /etc/shadowsocks.json -d start
```
该项目地址：
[shadowsocks-github](https://github.com/ziggear/shadowsocks)
上述这个项目的信息：
backup of https://github.com/shadowsocks/shadowsocks
copy from release 2.8.2

其他简单的一些指令
```
ssserver -p 443 -k password -m aes-256-cfb   //最简单的开启命令
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start //后台运行
sudo ssserver -d stop //关闭
sudo less /var/log/shadowsocks.log   //日志查看
```
#### 后台开启shadowsocks-libev多进程多配置
创建多个配置文件，如：
```
/etc/shadowsocks-libev/config1.json /etc/shadowsocks-libev/config2.json /etc/shadowsocks-libev/config3.json
```
新建内容如下的脚本：
```
#!/bin/bash
 
USER="nobody"
GROUP="nogroup"
DAEMON=/usr/bin/ss-server
 
# 配置文件1
CONFFILE1=/etc/shadowsocks-libev/config1.json
# 相应的进程PID文件
PIDFILE1=/var/run/shadowsocks-1.pid
# 启动ss-server
start-stop-daemon --start --quiet --pidfile $PIDFILE1 --chuid $USER:$GROUP --exec $DAEMON -- \
    -c "$CONFFILE1" -u -f $PIDFILE1
# 配置文件2
CONFFILE2=/etc/shadowsocks-libev/config2.json
PIDFILE2=/var/run/shadowsocks-2.pid
start-stop-daemon --start --quiet --pidfile $PIDFILE2 --chuid $USER:$GROUP --exec $DAEMON -- \
    -c "$CONFFILE2" -u -f $PIDFILE2
# 配置文件3
CONFFILE3=/etc/shadowsocks-libev/config3.json
PIDFILE3=/var/run/shadowsocks-3.pid
start-stop-daemon --start --quiet --pidfile $PIDFILE3 --chuid $USER:$GROUP --exec $DAEMON -- \
    -c "$CONFFILE3" -u -f $PIDFILE3
```
保存后加上x权限<br>
### 客户端使用
####下载相关软件工具
[所有平台的ss客户端](https://shadowsocks.org/en/download/clients.html)
但是不知道为什么linux不能用pip方式下载到shadowsocks-libev
不过我们还是有上述server端提到的那个ss项目，实际上它也是包含client，使用的指令是sslocal

编辑客户端的配置的文件时内容和服务端一致，然后启动客户端
```
$ ./sslocal  -c config.json -d start
```
其他命令：
```
sslocal  -p 443 -k password -m aes-256-cfb   //最简单的开启命令
 sslocal -p 443 -k password -m aes-256-cfb --user nobody -d start //后台运行
sudo ssslocal -d stop //关闭
```
至此以上已经配置好了shadowsocks的服务了，可是还不能正常使用，原因很简单，shadowsocks是使用的是socks5代理，如果是浏览器使用，需要安装特定的插件，当然firefox可以直接配置proxy即可，chrome 需要安装SwitchyOmega类似的插件，并且配置。这样一来就不能是全局的使用了，仅仅局限于浏览器。有很多代理转发工具，但是本文将介绍Polipo进行http/https的代理转发（ubuntu版本）

#### polipo安装
```
sudo apt-get install polipo
```
修改配置
```
vim /etc/polipo/config
```
参考配置内容：
```
logSyslog = true
logFile = /var/log/polipo/polipo.log

allowedPorts = 1-65535
tunnelAllowedPorts = 1-65535

proxyAddress = "0.0.0.0"
socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
chunkHighMark = 50331648
objectHighMark = 16384
serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
```

```
重启一下：/etc/init.d/polipo restart
```
设置polipo代理：
```
export http_proxy="http://127.0.0.1:17070"
export http_proxy="http://127.0.0.1:17070"
```
查看代理：
```
env | grep -i proxy
```
取消代理
```
unset http_proxy
unset https_proxy
```
上述内容参考：(http://blog.csdn.net/jon_me/article/details/53525059
http://www.auooo.com/2015/06/26/shadowsocks%EF%BC%88%E5%BD%B1%E6%A2%AD%EF%BC%89%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%8C%87%E5%8D%97/)

### 给ss服务器装加速软件
openvz不支持加速插件
如果想了解虚拟化技术
```
apt-get install virt-what -y
virt-what
```
就可以显示

#### 安装BBR加速：
```
wget --no-check-certificate https://raw.githubusercontent.com/wn789/BBR/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```
安装过程中后提示重启生效
查看是否启动：
```
lsmod | grep bbr
```
#### 安装锐捷加速
开心搬锐速安装
```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder.sh && bash serverspeeder.sh
# 上面这个是新的锐速脚本，如果安装存着问题，可以尝试使用下面这个 旧的锐速脚本。
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```
最后两项输入y开机自动启动锐速，y立刻启动锐速。（这个参数设置一般情况下都是不会出现的。）
卸载：
```
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```
该脚本命令：
```
#重启锐速
/serverspeeder/bin/serverSpeeder.sh restart
#启动锐速
/serverspeeder/bin/serverSpeeder.sh start
#停止锐速
/serverspeeder/bin/serverSpeeder.sh stop
#查看锐速运行情况
/serverspeeder/bin/serverSpeeder.sh status
```
注意：锐捷加速和BBR不能同时共存
锐速安装参考：[优秀的VPS TCP加速软件 —— 一键锐速安装脚本（开心版）](https://doub.io/ruisu-jc1/)

### 自启动和服务器定时重启
#### 自启动
一般而言，我们需要ss在server启动时就能开启，那么我们需要将其加入自启动的队列中
首先在`/etc/rc.local`中加入执行命令，比如
```
ssserver -c /etc/shadowsocks.json -d start
```
然后再完成下面这个步骤加多一层保险
编辑启动文件/etc/init.d/shadowsocks
内容是：
```
nohup ssserver -c /etc/shadowsocks/config.json > /etc/shadowsocks/log &
```
再执行
```
sudo update-rc.d xxx defaults (xxx是脚本文件名)
sudo update-rc.d test defaults  //更新后就加入到系统常驻任务
sudo update-rc.d -f test remove  //卸载
```
文件要注意权限
#### 定时重启
通过crontab实现
首先要矫正时间
```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
vps一般都是又crontab的，如果没有自行下载
```
#安装Crontab
yum install vixie-cron crontabs
#设置开机启动Crontab
chkconfig crond on
#启动Crontab
service crond start

#安装Crontab
apt-get install cron
#重启Crontab
/etc/init.d/cron restart
```
接下来添加执行命令
```
crontab -e

0 4 * * * reboot  //每天4点重启一次
```
基本格式
```
crontab基本格式 : 
*　　*　　*　　*　　*　　command 
分　时　日　月　周　命令 
第1列表示分钟1～59 每分钟用*或者 */1表示 
第2列表示小时1～23（0表示0点） 
第3列表示日期1～31 
第4列表示月份1～12 
第5列标识号星期0～6（0表示星期天） 
第6列要运行的命令 
```
最后执行`service cron restart`或重启一下vps

### 安装服务器监控探针脚本
下面分为两个脚本，一个是监测VPS的负载，另外一个是监测SSR的可用性
先来说第一个--ServerStatus
#### 安装ServerStatus
github地址：
(https://github.com/ToyoDAdoubi/ServerStatus-Toyo)
(https://github.com/tenyue/ServerStatus)
注意，这个脚本需要服务端和客户端相互配合才能使用，安装了client客户端脚本的并且满足server配置才能访问网页
默认使用端口：35601
VPS要求：
CentOS 7 / Debian 7+ / Ubuntu 14.04 +  python2.7+
安装：
```
wget -N --no-check-certificate https://softs.fun/Bash/status.sh && chmod +x status.sh
 
# 如果上面这个脚本无法下载，尝试使用备用下载：
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/status.sh && chmod +x status.sh
```
操作：
```
# 显示客户端管理菜单
bash status.sh c
 
# 显示服务端管理菜单
bash status.sh s
```
server和client安装完成后都要先自定义配置
然后就会弹出安装和配置、查看等等选项
安装server的时候会弹出
```
是否由脚本自动配置HTTP服务(服务端的在线监控网站)[Y/n]，选择Y，会自动安装caddy
```
配置:
```
sever：   用户名: 随便 密码: 随便 节点名: 随便类型: KVM 位置:随便 

client :       IP:vps的ip  帐号和密码要和server的一致
```
客户端命令：
```
启动：service status-client start

停止：service status-client stop

重启：service status-client restart

查看状态：service status-client status
```
服务端命令：
```
启动：service status-server start

停止：service status-server stop

重启：service status-server restart

查看状态：service status-server status
```
caddy命令：
```
启动：service caddy start
停止：service caddy stop
重启：service caddy restart
查看状态：service caddy status

Caddy配置文件：/usr/local/caddy/Caddyfile

默认脚本只能一开始安装的时候设置配置

```

```
安装目录：/usr/local/ServerStatus

网页文件：/usr/local/ServerStatus/web

配置文件：/usr/local/ServerStatus/server/config.json

客户端查看日志：tail -f tmp/serverstatus_client.log

服务端查看日志：tail -f /tmp/serverstatus_server.log
```
如果要修改网页标题或者网页顶部公告内容，打开 /usr/local/ServerStatus/web/index.html 文件修改即可

上述内容参考：[『原创』多服务器 云探针、云监控 —— ServerStatus 一键管理脚本](https://doub.io/shell-jc3/comment-page-1/#comments)

#### SSR在线监控脚本ssrstatus
这个脚本是快速监测SSR是否可用的，支持IPV6测试
Github项目：(https://github.com/ToyoDAdoubi/SSRStatus)
安装：
```
wget -N --no-check-certificate https://softs.fun/Bash/ssrstatus.sh && chmod +x ssrstatus.sh && bash ssrstatus.sh
 
# 如果上面这个脚本无法下载，尝试使用备用下载：
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssrstatus.sh && chmod +x ssrstatus.sh && bash ssrstatus.sh
```
下载完成后需要修改一下脚本内容
```
vi ssrstatus.sh
```
找到第 16行的 SSR_folder="/root/shadowsocksr/shadowsocks" 参数，改成自己vps中ssr子目录绝对路径，doub的ssr脚本位置是/usr/local/shadowsocksr/shadowsocks

命令：
```
# 管理菜单
./ssrstatus.sh
 
# 检测所有账号配置（快捷参数）
./ssrstatus.sh t
# 检测单独账号配置（快捷参数）
./ssrstatus.sh o
# 检测自定义账号配置（快捷参数）
./ssrstatus.sh a
 
# 查看日志输出（快捷参数）
./ssrstatus.sh log
```
默认设置下，打开 “vps的ip:8888"就可以显示了，前面那个脚本是80端口(可以不写)
ssrstatus内容参考：[『原创』ShadowsocksR/SS账号 在线云监控 — SSRStatus 一键脚本](https://doub.io/shell-jc5/)


### 浏览器使用
在客户端中，如果在不开启全局代理或者浏览器代理的情况下，那么实际上是不能科学上网的。这里不推荐ubuntu的全局模式，因为开了系统全局，整个系统打应用都要走代理。
#### chrome
不知从何时起，linux端的chrome不支持浏览器内设置代理（会提示你chrome默认使用全局代理），但是通过chrome命令还是单独可以设置代理的。可以通过google-chrome查看全局指令，下面只提到简单常用的
```
google-chrome  --proxy-server="socks://127.0.0.1:1080"  //开本地sock5代理
google-chrome  --no-proxy-server  //取消代理
```
但是，上面的做法还是有不足，因为浏览器并不能识别国内和国外网站，如果用户浏览国内网站依旧走代理，速度又慢又耗流量。建议配和switchyomega插件，并且加上pac文件。本人使用的是[gfwlist的项目](https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList)

### firefox
和chrome类似，自行查找，如foxyproxy