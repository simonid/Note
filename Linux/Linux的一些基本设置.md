# Linux上的一些基本设置
以下包含了桌面版和服务器版<br>

### Linux设置时间
#### 最粗暴的手动设置时间
```
date -s 10/11/17
#设置日期为2017.10.11

date -s 00:00:00
#设置准确时间

date 1011170000.00
#设置时间为2017.10.11 00:00:00
```

#### 查看和设置系统硬件时间
```
查看：
hwclock --show
或
clock --show

设置：
hwclock --set-date="10/11/17 00:00"
或
clock --set-date="10/11/17 00:00"
```

#### 同步系统和硬件时钟
```
hwclock --hctosys 或者 # clock --hctosys  hc代表硬件时间，sys代表系统时间
hwclock --systohc或者# clock --systohc  即用系统时钟同步硬件时钟
```

#### 时区设置
输入`tzselect`后选中时区，选择完成后系统提示输入`TZ='Asia/Shanghai'; export TZ`，输入后执行`cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime`，然后更新时间：`ntpdate time.windows.com`<br>
或者可以通过修改配置文件：<br>
修改`/etc/sysconfig/clock`，将内容改为`ZONE="Asia/Shanghai"`，然后` /etc/localtime`，创建一个软链接：`ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`，重启完成<br>