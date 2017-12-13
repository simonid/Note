# VPS上搭建属于自己的云盘

介绍两个网盘工具，一个是`Filemanager`网盘管理工具，另外一个是`ResilioSync` P2P同步工具。

## Filemanager
<b>这个教程主要参考该链接的文章：<br>
https://doub.io/jzzy-3/</b><br>
<br>
filemanager是一个在线文件管理工具，有着大部分网盘的功能，但是较为遗憾的是不支持离线上传和下载的功能，我的VPS使用caddy来拓展。
其他的基本功能如下：
```
支持 上传文件
支持 按类型 搜索文件
支持 批量压缩 文件下载
支持 多用户管理(权限可控)
支持 在网页执行 Linux命令
支持 创建 共享链接(限时/永久)
支持 在线编辑 各类文本文件
支持 在线浏览 图片/文本/视频等
支持 新建/重命名/移动/删除 文件和文件夹等
部署简单，几步完成，无需任何依赖环境
```
[Caddy 文档](https://caddyserver.com/docs/http.filemanager)
<br>
[Github 项目](https://github.com/hacdias/filemanager)
<br>
<br>

### Caddy安装
```
wget -N --no-check-certificate https://softs.fun/Bash/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
 
# 如果上面这个脚本无法下载，尝试使用备用下载：
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
```

创建目录：
```
mkdir /usr/local/caddy/www && mkdir /usr/local/caddy/www/file
```

### Caddyfile配置
和之前另外一文提到过的caddy配置文件类似，如下：<br>
```
IP:port {
 root /usr/local/caddy/www/ssr
 timeouts none
 tls xxx@xxx.com
 gzip
}
#上面这个无需填，是另外一个教程caddy伪装流量的

IP:80 {
 root /usr/local/caddy/www/file
 timeouts none
 gzip
 filemanager / /usr/local/caddy/www/file {
  database /usr/local/caddy/filemanager.db
 }
}
```
本人使用HTTP端口，如果要使用其他方式可以参考原文<br>

### 升级Filemanager
因为Filemanager是Caddy的扩展，是融合成一个文件的，升级Filemanager=升级Caddy（加扩展），所以只需要重新执行下面的命令覆盖安装Caddy即可（只会覆盖 Caddy自身，不影响配置文件），覆盖安装后启动Caddy即可（ `/etc/init.d/caddy start` ）。<br>
<br>

```
wget -N --no-check-certificate https://softs.fun/Bash/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
 
# 如果上面这个脚本无法下载，尝试使用备用下载：
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh install http.filemanager
```

然后重启caddy：<br>
```
sudo service caddy restart
```

<b>注意，启动caddy可能什么也没输出，但是也有可能是启动失败的</b>

查看日志：<br>
```bash
tail -f /tmp/caddy.log
```
如果日志文件是`IP+端口`的类似输出，那么就说明成功了<br>

接着打开网址，是`IP:Port`的格式，如果用http当然就不需要加端口了。<br>
默认账号和密码都是 admin，可自行修改<br>

### 可能出现的问题
#### Caddy下载文件频繁中断
编辑caddy的配置文件，补上：`timeouts none`<br>

#### caddy启动失败
有些vps默认装了apache且是启动状态的，需要手动关闭<br>
检查端口占用：
```
sudo netstat -lntp
```
关闭apach2：
```
sudo /etc/init.d/apache2 stop
# 尝试使用上面这个关闭，如果没效果或者提示什么错误无法关闭，那就用下面这个强行关闭进程。
kill -9 $(ps -ef|grep "apache2"|grep -v "grep"|awk '{print $2}')
```
如果不想要apache2了k可以禁用apache的自启：
```
cd /etc/rc3.d
ls
#找到apache2，然后
sudo mv S02apache2 K02apache2

以S开头表示开机自启动，K开头表示开机不启动
```

当然也可以卸载:
```
# 以下代码仅限 Debian/Ubuntu 系统 #
sudo apt-get remove --purge apache2
```

#### 防火墙干扰
```
# 删除防火墙规则，内容一样把 -I 换成 -D 就行了：
iptables -D INPUT -m state --state NEW -m tcp -p tcp --dport 端口 -j ACCEPT
iptables -D INPUT -m state --state NEW -m udp -p udp --dport 端口 -j ACCEPT

#添加规则
# 删除防火墙规则，内容一样把 -I 换成 -D 就行了：
iptables -D INPUT -m state --state NEW -m tcp -p tcp --dport 端口 -j ACCEPT
iptables -D INPUT -m state --state NEW -m udp -p udp --dport 端口 -j ACCEPT
```

#### 忘记密码怎么办
```
/etc/init.d/caddy stop
rm -rf /usr/local/caddy/filemanager.db
/etc/init.d/caddy start
```

#### aria2加速下载
参考：<br>
[BT/种子/磁力链接下载工具 —— Aria2 一键安装管理脚本](https://doub.io/shell-jc4/#使用说明)<br><br>
[一个支持 离线下载/BT/磁力链接 的Aria2在线管理面板 —— Aria2 WebUI](https://doub.io/wlzy-4/)<br>
注意，如果是同时用filemanager和aria2，那么caddy配置不需要browse参数了，[参考这里](https://doub.io/jzzy-3/#如果你是%20Aria2%20教程里过来的，那么请看这个示例和说明)<br>

<br><br>

## Resilio Sync Web UI
这个是P2P下载软件(被墙了，挂了代理速度依旧慢，不过别人分享的资源倒是挺多，如kindle书)，可以Google详细了解，注意，谨慎导入太多别人的密钥，因为是P2P，一旦在线用户都较少，然后有人不断下载，会加重VPS负担<br>
<br>

官方网站：https://www.resilio.com/
<br>
客户端下载地址：https://www.resilio.com/platforms/desktop/
<br>

<br>

### 下载及安装
```bash
uname -m
# 64位选第一行，32位选第二行。
wget --no-check-certificate -O sync.tar.gz https://download-cdn.resilio.com/stable/linux-x64/resilio-sync_x64.tar.gz
wget --no-check-certificate -O sync.tar.gz https://download-cdn.resilio.com/stable/linux-i386/resilio-sync_i386.tar.gz
# 解压后赋予执行权限。
tar -xzf sync.tar.gz && rm -rf sync.tar.gz
chmod +x rslsync
```

### 设置时间(caddy有同步时间需要)
```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
### 启动
```
./rslsync --webui.listen 0.0.0.0:8888
```
浏览器打开 `http://ip:8888` 就会看到 Sync Web UI 界面。<br>

### 关闭同步
```
kill -9 $(ps -ef|grep "rslsync"|grep -v grep|awk '{print $2}')
```