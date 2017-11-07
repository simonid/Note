# VPS上搭建SSR

SSR项目地址：
原项目:https://github.com/shadowsocksr-backup/shadowsocksr<br>
备份项目：https://github.com/zyfworks/ssr-backup<br>

#### 注意，上面的项目中就包含了服务端和客户端，有多个版本或者分支，每个版本功能不一，有单用户也有多用户，也有测试（本人用Python版单用户，也就是master分支的，下面就以master分支作为介绍）

### 服务端搭建：
在一级目录下执行：
```sh
bash initcfg.sh
```
再进入`~/shadowsocksr/shadowsocks`下，执行：
```sh
sudo nohup  python server.py -p serverport -k password -m aes-256 -O auth_aes128_sha1 -o tls1.2_ticket_auth_compatible   >/dev/null 2>&1  &
```
服务端不需要设置配置文件<br>
注意，前面没有协议的选项，用户可以通过`python server.py  -h`查看帮助，可以看到的确没有该选项，不过在manyuser分支上有。<br>
完成这些ssr就在vps后台运行了<br>
如果想查看后台运行情况，可以通过ps命令，但是还是建议用netstat扫描：
```sh
sudo netstat -lnp | grep :6669
```
这样就能找出使用制定端口的任务pid<br>
此外，官方还给出下面的内容：<br>
```sh
To run in the background:

./logrun.sh

To stop:

./stop.sh

To monitor the log:

./tail.sh

```
但是不建议使用，因为有配置文件的坑，新手容易犯错（后面会介绍）<br>
### 客户端搭建：
还是接着前面那个位置，在`~/shadowsocksr/shadowsocks`下，找到`local.py`，执行：<br>
```
sudo python local.py  -d  start
```
#### 注意，这里不要用配置参数，也不要自己新建一个json配置文件然后用-c参数启动，直接修改一级目录下的user-config.json，设置需要的内容
完成上面，那么服务端和客户端都配置完成了<br>

如果觉得socks代理有时打扰到网速，可以取消代理：
```sh
代理查看：
env|grep -i proxy
取消代理：
unset  代理名
```