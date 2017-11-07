# Linux 脚本运行

## 脚本后台运行
一般大致有两种方式：
1. command &:后台运行，但是终端关闭之后会停止运行
2. nohup command &：后台运行，关闭终端也不会终止

当我们执行一个脚本的时候，按下ctrl+z就可以把前台任务放到后台运行。可以看到系统提示：
```
[1]+ Stopped /root/bin/rsync.sh
```
然后我们可以把程序调度到后台执行：（bg 后面的数字为作业号）
```
#bg 1
[1]+ /root/bin/rsync.sh &
```
用 jobs 命令查看正在运行的任务：
```
#jobs
[1]+ Running /root/bin/rsync.sh &
```
如果想把它调回到前台运行，可以用
```
#fg 1
/root/bin/rsync.sh
```
这样，你在控制台上就只能等待这个任务完成了。
总结一下前面的基本内容：
```
& 将指令丢到后台中去执行
[ctrl]+z 將前台任务丟到后台中暂停
jobs 查看后台的工作状态
fg %jobnumber 将后台的任务拿到前台来处理
bg %jobnumber 将任务放到后台中去处理
```
### 对前面内容的另外补充：
可以对后台任务内容输出做重定向
```
command >out.file 2>&1 &
2>&1表示所有的标准输出和错误输出都将被重定向到一个叫做out.file 的文件中
```
但是，如果觉得输出的内容很多，那么可以将输出内容重定向到黑洞文件:
```
command >/dev/null 2>&1 &
```
#### nohup
前面提到了nohup命令(可以理解为no hang up)可以使得后台运行的任务更加方便。用户退出终端之后并不会停止。本质上是让程序忽略hangup信号从而让程序可以继续运行
```
nohup conmmand &  //缺省情况下，输出重定向到一个名为nohup.out文件
nohup command > myout.file 2>&1  //输出被重定向到myout.file文件中
nohup command >/dev/null 2>&1 &  //相当于不输出任何内容
```
#### setsid
本质是让程序在一个新的session中运行，所以父进程就不会是当前终端 
```
setsid ./test.sh
```

#### disown
如果事先在命令前加上 nohup 或者 setsid 就可以避免 HUP 信号的影响。但是如果我们未加任何处理就已经提交了命令，该如何补救才能让它避免 HUP 信号的影响呢
格式：
```
用disown -h jobspec来使某个作业忽略HUP信号。
用disown -ah 来使所有的作业都忽略HUP信号。
用disown -rh 来使正在运行的作业忽略HUP信号
```

### 查后台
#### ps命令
```
ps -A  //显示所有进程信息

ps -A | grep + 通配符或正则 //显示关键字进程

ps -ax  //显示当前所有进程

ps -ax | less 或 ps -ax |more //前面输出很多，可以配合less方便查看

ps -ef  //显示所有进程信息，连同命令行

ps -u user1  //查看用户user1进程

ps -aux | less  //显示cpu和内存等信息
  衍生：
  ps -aux --sort -pcpu | less  //根据 CPU 使用来升序排序
  ps -aux --sort -pmem | less  //根据 内存使用 来升序排序
  ps -aux --sort -pcpu,+pmem | head -n 10  //通过管道显示前10个结果

ps -C getty  //ps -C getty
  衍生： ps -f -C getty  //-f参数来查看格式化的信息列表
```
非常实用的实时监控命令：
```
watch -n 1 ‘ps -aux --sort -pmem, -pcpu’  //每秒刷新一次进程信息，比win的任务管理器还全，类似top。

watch -n 1 ‘ps -aux --sort -pmem, -pcpu | head 20’   //如果输出太长，我们也可以限制它，比如前20条

watch -n 1 ‘ps -aux -U pungki u --sort -pmem, -pcpu | head 20’  //如果你只需要看名为'pungki'用户的信息

```

#### top命令
格式：
```
top [-] [d] [p] [q] [c] [C] [S] [s]  [n]
```
参数说明：
```
d 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。 
p 通过指定监控进程ID来仅仅监控某个进程的状态。 
q 该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。 
S 指定累计模式 
s 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。 
i 使top不显示任何闲置或者僵死进程。 
c 显示整个命令行而不只是显示命令名 
```
惯用操作
```
top   //每隔5秒显式所有进程的资源占用情况
top -d 2  //每隔2秒显式所有进程的资源占用情况
top -c  //每隔5秒显式进程的资源占用情况，并显示进程的命令行参数(默认只有进程名)
top -p 12345 -p 6789//每隔5秒显示pid是12345和pid是6789的两个进程的资源占用情况
top -d 2 -c -p 123456 //每隔2秒显示pid是12345的进程的资源使用情况，并显式该进程启动的命令行参数
```
### 其他检测后台命令
`netstat -ltnp`可以查看后台运行中的脚本（不一定都会显示所有脚本），有PID、名字、Local Address、Foreign Address、State

## 杀后台
#### kill
```
1.法1：通过jobs命令查看job号（假设为num），然后执行kill %num<br/>
2。法2：通过ps命令查看job的进程号（PID，假设为pid），然后执行kill pid
立即关闭后台：kill -s 9 pid
```
#### pkill
和kill不同，kill输入的是pid，pkill直接输入任务名

## 批处理执行
#### at命令
```
$ at -f backup.sh 10 am tomorrow  //明早10点执行
```
#### watch命令
```
watch  -n 10 sh  test.sh  &  #每10s在后台执行一次test.sh脚本
```
#### crontab定时运行
安装：
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
添加一条简单命令：
```
crontab -e

0 4 * * * reboot  //每天4点重启一次
```
基本格式：
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
最后执行service cron restart或重启一下生效

参考：https://linux.cn/article-4743-1.html
<br>
http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316399.html
<br>
https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/index.html