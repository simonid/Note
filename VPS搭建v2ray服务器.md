# VPS搭建v2ray
v2ray是不久前才出现的项目，目前还在维护中，集成了许多的功能，但是高级的配置难度较大。官网有教程，并且还提供了一个白话文版教程<br>
补上另外一个教程:https://toutyrater.github.io<br>

### 关于客户端
首先需要到v2ray的GitHub仓库下载软件基础包，然后可以自己去找一些第三方的软件(注意这些第三方的软件是依赖官方的基础包，所以要解压到同一个目录下)，建议使用v2rayW(有图形化的设置界面)<br>

## CentOS6
### 基础使用
#### 安装
官方有v2ray的一键配置脚本，但是不适用与Centos(里面用到centos6没有的systeml)。<br>
不过网上有一个centos6的一键脚本：<br>
https://raw.githubusercontent.com/wuming2018/wuming/master/v2-centos6.sh<br>
安装完成后，v2ray的位置在`/usr/bin/v2ray`的目录下，默认情况下，其配置文件也在该目录下，名字为`config.json`。<br>
#### 配置
服务端<br>
配置工具(可以傻瓜式快速生成)：
<br>
https://htfy96.github.io/v2ray-config-gen/<br>
UUID可以使用上面提供的，如果需要自己设置，可以上网搜UUID generator，网页打开后就自动生成UUID。<br>
后台运行：<br>
```bash
sudo nohup ./v2ray -config ./config.json start >/dev/null 2>&1 &
```
<br><br>
客户端:<br>
安卓手机上使用`v2rayNG`，按照前面的配置，将ip、端口、uuid、传输协议kcp、加密协议填上就可以，其他都为空<br>
关于加密协议的问题，"aes-128-gcm"推荐在 PC 上使用
"chacha20-poly1305"推荐在手机端使用<br>
所有平台的客户端：<br>
https://www.v2ray.com/chapter_01/3rd_party.html<br>


## Ubuntu以及CentOS7以上
获取一键脚本
```
wget https://toutyrater.github.io/install-release.sh
```
打开[v2ray配置生成器](https://htfy96.github.io/v2ray-config-gen/)<br>
建议不要使用kcp，建议使用tcp+mux(mux自默认选中)<br>
编辑`/usr/bin/v2ray/config.json`，粘贴上生成器的配置<br>
<br>
启动：
```
sudo nohup /usr/bin/v2ray/v2ray start > /dev/null 2>&1 &
```

自启动：<br>
编辑`/etc/rc.local`,末尾补充：
```
nohup /usr/bin/v2ray/v2ray start > /dev/null 2>&1 &
```


<br><br>
### 高级配置
待续<br>
#### HTTP混淆
