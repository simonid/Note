# Linux创建自启动任务

### 创建自启动服务的方式
<ol>
<li>ln -s 在/etc/rc.d/rc*.d中创建/etc/init.d/服务的软连接</li>
<li>直接编辑/etc/rc.local，补充启动命令</li>
<li>chkconfig/sysv-rc-conf命令行</li>
<li>ntsysv伪图形运行级别设置</li>
</ol>

### 理解Linux启动步骤
首先了解一下Linux启动的步骤：
<ol>
<li>根据/etc/inittab启动init进程</li>
<li>init进程执行/etc/rc.d/rc.sysinit</li>
<li>init进程完成/etc/rc.d/rc*.d或/etc/rc*.d/下的软连接，它们指向/etc/rc.d/init.d/或/etc/init.d</li>
<li>init进程执行/etc/rc.local</li>
<li>init进程完成tty</li>
</ol>

参考：[Linux 中 rc.local、init.d、rc.x、init 这几个文件（夹）各有什么作用？启动执行的脚本应该均放在 rc.local 中吗？ - 大水的回答 - 知乎](
https://www.zhihu.com/question/20126189/answer/26598868)<br>

上述的内容可以基本概括为：
<ol>
<li>系统初始化:rc.sysinit</li>
<li>服务启动:启动rc.*下的任务</li>
<li>本地初始化:rc.local</li>
</ol>

<br>

#### rc*.d中的优先级
```
# 0 - 停机（千万不要把initdefault 设置为0 ）
# 1 - 单用户模式
# 2 - 多用户，但是没有 NFS 
# 3 - 完全多用户模式
# 4 - 没有用到
# 5 - X11 
# 6 - 重新启动 （千万不要把initdefault 设置为6 ）


对各个运行级的详细解释：
0 为停机，机器关闭。
1 为单用户模式，就像Win9x 下的安全模式类似，只允许root用户对系统进行维护。
2 为多用户模式，但是没有NFS 支持。
3 为完整的多用户模式，是标准的运行级，字符界面。
4 一般不用，在一些特殊情况下可以用它来做一些事情。
例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。
5 就是 X11 ，进到 X Window 系统了，图形界面。
6 为重启，运行 init 6 机器就会重启。
```
来自：[init.d，rc.d详解 Linux运行时详解](http://www.cnitblog.com/windone0109/archive/2008/04/23/42651.aspx)

<br>

`/etc/init.d`下的服务可以使用`service 服务名 动作`这样的格式来控制<br>

补充一下`/etc/init.d/`下的动作：
```
start
stop
reload
restart
force-reload
```

#### 上述总结：
<br>
三个目录 =>
<ul>
    <li>/etc/rcN.d/ 是System V init系统启动时查找的服务程序目录。</li>
    <li>/etc/init.d/ 目录是System V init系统真正的服务程序所在地。</li>
    <li>/etc/init/ 是Upstart系统寻找作业配置文件的地方。</li>
</ul>
<br>

两个文件 =>
<ul>
    <li>/etc/inittab 是System V init系统的配置文件，其中有设置默认的运行级别。</li>
    <li>/etc/rc.local 是一个用户常用来添加系统启动脚本的地方。</li>
</ul>
<br>

一个命令 =>
<br>
service 是用来操作System V init脚本或Upstart作业的命令接口。

参考：<br>
[http://monklof.com/post/14/](http://monklof.com/post/14/)
<br>

### 通过ln -s方式创建自启动任务

`/etc/rc[0~6].d`其实是`/etc/rc.d/rc[0~6].d`的软连接，主要是为了保持和Unix的兼容性才做此策。不同的发行版之间可能处在差异，另外，同个发行版可能因为自身版本原因也会存在差异，比如ubuntu16.04不存在`/etc/rc.d`、`/etc/rc.d/init.d/`目录<br>

每个`/etc/rc*.d`目录下都有以`K`或`S`开头的一些软链接文件，基本格式是这样的：
```
K/S+num+name

K：运行级别加载时需要关闭
S：~开启

num：任务优先级

name：任务名
```
可以通过`ls -l`命令来查看指向：
```bash
simon@simonc:/etc/rc0.d$ ls -l 
total 4
lrwxrwxrwx 1 root root  29 Dec 13 10:38 K01apache-htcacheclean -> ../init.d/apache-htcacheclean
lrwxrwxrwx 1 root root  17 Dec 13 10:38 K01apache2 -> ../init.d/apache2
lrwxrwxrwx 1 root root  15 Dec 13 11:26 K01caddy -> ../init.d/caddy
lrwxrwxrwx 1 root root  18 Dec 13 11:04 K01fail2ban -> ../init.d/fail2ban
lrwxrwxrwx 1 root root  19 Dec 13 10:38 K01fetchmail -> ../init.d/fetchmail

...
```

假如我们要创建一个开机自启动任务，可以将需要自启的脚本拷贝到`/etc/init.d`目录下，然后在`/etc/rc*.d`(也可能是`/etc/rc.d/rc*.d`)下做软连接：
```bash
ln -s /etc/init.d/sshd /etc/rc3.d/S100ssh
```

### 通过在/etc/rc.local下补充指令创建自启动
注意`/etc/rc.local`下的命令以`root`身份执行，所以不需要加sudo
<br>

### 通过chkconfig/sysv-rc-conf创建自启动任务
rpm系的命令：chkconfig<br>
deb系的命令：sysv-rc-conf<br>

chkconfig执行时只需使用`chkconfig 服务名 on`即可，若想关闭，将`on`改为`off`<br>

在默认情况下，chkconfig会自启动2345这四个级别，如果想自定义可以加上`--level`选项<br>
<br>
level：
```
等级0表示：表示关机
等级1表示：单用户模式
等级2表示：无网络连接的多用户命令行模式
等级3表示：有网络连接的多用户命令行模式
等级4表示：不可用
等级5表示：带图形界面的多用户模式
等级6表示：重新启动
```

使用范例：
```bash
chkconfig --list        #列出所有的系统服务
chkconfig --add httpd        #增加httpd服务
chkconfig --del httpd        #删除httpd服务
chkconfig --level httpd 2345 on        #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list        #列出系统所有的服务启动情况
chkconfig --list mysqld        #列出mysqld服务设置情况
chkconfig --level 35 mysqld on        #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
chkconfig mysqld on        #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级
```

参考：[Linux下chkconfig命令详解](http://www.cnblogs.com/panjun-Donet/archive/2010/08/10/1796873.html)<br>

sysv-rc-conf的命令格式和chkconfig基本一致<br>

参考：[sysv-rc-conf管理任务](http://blog.csdn.net/gatieme/article/details/45251389)<br>

### 通过ntsysv创建自启动任务
该命令和chkconfig一致，只是多了图形界面操作<br>

<br><br>

补充，用户可以同`update-rc.d`命令来管理任务启动项，详情参考：<br>
http://blog.csdn.net/shb_derek1/article/details/8489112<br>

