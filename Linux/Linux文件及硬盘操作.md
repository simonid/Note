# Linux文件及硬盘操作


#### 查找当前目录下所有含指定字符串的文件
```
grep -r "demo" .
或
find . "demo"
```

### grep命令

#### 从文件内容查找匹配指定字符串的行：
```
$ grep "被查找的字符串" 文件名
```

#### 从文件内容查找与正则表达式匹配的行：
```
$ grep –e “正则表达式” 文件名
```

#### 查找时不区分大小写：
```
$ grep –i "被查找的字符串" 文件名
```

#### 查找匹配的行数：
```
$ grep -c "被查找的字符串" 文件名
```

#### 从文件内容查找不匹配指定字符串的行：
```
$ grep –v "被查找的字符串" 文件名
```

#### 从根目录开始查找所有扩展名为.txt的文本文件，并找出包含"phpzixue.cn"的行
```
find . -type f -name "*.txt" | xargs grep "phpzixue.cn"

xargs 表示递归查找子目录
```


### du命令

#### 查看单独文件大小
```
du -lh
```

#### 查看磁盘分区的空间
```
df -lh
```

#### 文件排序
```
du -s / | sort -rn
或
du|sort -nr|more
#按字节排序

du -sh / | sort -rn
#按m来排序

du -sh / | sort -rn | head -n3
#选出从大到小排在前3的文件或目录

du -sh / | sort -rn | tail -n3
#选出从大到小排在后3的文件或目录
```
#### 输出当前目录下各个子目录所使用的空间
```
du -h  --max-depth=1
```

#### 参数：
```
-a或-all  显示目录中个别文件的大小。   

-b或-bytes  显示目录或文件大小时，以byte为单位。   

-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。 

-k或--kilobytes  以KB(1024bytes)为单位输出。

-m或--megabytes  以MB为单位输出。   

-s或--summarize  仅显示总计，只列出最后加总的值。

-h或--human-readable  以K，M，G为单位，提高信息的可读性。

-x或--one-file-xystem  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。 

-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   

-S或--separate-dirs   显示个别目录的大小时，并不含其子目录的大小。 

-X<文件>或--exclude-from=<文件>  在<文件>指定目录或文件。   

--exclude=<目录或文件>         略过指定的目录或文件。    

-D或--dereference-args   显示指定符号链接的源文件大小。   

-H或--si  与-h参数相同，但是K，M，G是以1000为换算单位。   

-l或--count-links   重复计算硬件链接的文件。
```


### find命令
```
$find  ~  -name  "*.txt"  -print    #在$HOME中查.txt文件并显示
$find  .   -name  "*.txt"  -print
$find  .   -name  "[A-Z]*"  -pri26nbsp;   #对匹配的文件使用cpio命令，将他们备份到磁带设备中
-prune                              #忽略某个目录
=====================================================
$find  ~  -name  "*.txt"  -print    #在$HOME中查.txt文件并显示
$find  .   -name  "*.txt"  -print
$find  .   -name  "[A-Z]*"  -print  #查以大写字母开头的文件
$find  /etc  -name  "host*"  -print #查以host开头的文件
$find  .  -name  "[a-z][a-z][0–9][0–9].txt"   -print  #查以两个小写字母和两个数字开头的txt文件
$find .  -perm  755  -print
$find  .  -perm -007  -exec ls -l {} \;  #查所有用户都可读写执行的文件同-perm 777
$find  . -type d  -print
$find  .  !  -type  d  -print
$find  .  -type l  -print
$find  .  -size  +1000000c  -print       #查长度大于1Mb的文件
$find  .  -size  100c        -print      # 查长度为100c的文件
$find  .  -size  +10  -print             #查长度超过期作废10块的文件（1块=512字节）
$cd /
$find  etc  home  apps   -depth  -print  | cpio  -ivcdC65536  -o  /dev/rmt0
$find  /etc -name "passwd*"  -exec grep  "cnscn"  {}  \;  #看是否存在cnscn用户
$find . -name "yao*"  | xargs file
$find  . -name "yao*"  |  xargs  echo   "" > /tmp/core.log
$find  . -name "yao*"  | xargs  chmod  o-w
======================================================
find  -name april*                      在当前目录下查找以april开始的文件
find  -name  april*  fprint file        在当前目录下查找以april开始的文件，并把结果输出到file中
find  -name ap* -o -name may*  查找以ap或may开头的文件
find  /mnt  -name tom.txt  -ftype vfat  在/mnt下查找名称为tom.txt且文件系统类型为vfat的文件
find  /mnt  -name t.txt ! -ftype vfat   在/mnt下查找名称为tom.txt且文件系统类型不为vfat的文件
find  /tmp  -name wa* -type l           在/tmp下查找名为wa开头且类型为符号链接的文件
find  /home  -mtime  -2                 在/home下查最近两天内改动过的文件
find /home   -atime -1                  查1天之内被存取过的文件
find /home -mmin   +60                  在/home下查60分钟前改动过的文件
find /home  -amin  +30                  查最近30分钟前被存取过的文件
find /home  -newer  tmp.txt             在/home下查更新时间比tmp.txt近的文件或目录
find /home  -anewer  tmp.txt            在/home下查存取时间比tmp.txt近的文件或目录
find  /home  -used  -2                  列出文件或目录被改动过之后，在2日内被存取过的文件或目录
find  /home  -user cnscn                列出/home目录内属于用户cnscn的文件或目录
find  /home  -uid  +501                 列出/home目录内用户的识别码大于501的文件或目录
find  /home  -group  cnscn              列出/home内组为cnscn的文件或目录
find  /home  -gid 501                   列出/home内组id为501的文件或目录
find  /home  -nouser                    列出/home内不属于本地用户的文件或目录
find  /home  -nogroup                   列出/home内不属于本地组的文件或目录
find  /home   -name tmp.txt   -maxdepth  4  列出/home内的tmp.txt 查时深度最多为3层
find  /home  -name tmp.txt  -mindepth  3  从第2层开始查
find  /home  -empty                     查找大小为0的文件或空目录
find  /home  -size  +512k               查大于512k的文件
find  /home  -size  -512k               查小于512k的文件
find  /home  -links  +2                 查硬连接数大于2的文件或目录
find  /home  -perm  0700                查权限为700的文件或目录
find  /tmp  -name tmp.txt  -exec cat {} \;
find  /tmp  -name  tmp.txt  -ok  rm {} \;
find   /  -amin   -10       # 查找在系统中最后10分钟访问的文件
find   /  -atime  -2         # 查找在系统中最后48小时访问的文件
find   /  -empty              # 查找在系统中为空的文件或者文件夹
find   /  -group  cat        # 查找在系统中属于 groupcat的文件
find   /  -mmin  -5         # 查找在系统中最后5分钟里修改过的文件
find   /  -mtime  -1        #查找在系统中最后24小时里修改过的文件
find   /  -nouser             #查找在系统中属于作废用户的文件
find   /  -user   fred       #查找在系统中属于FRED这个用户的文件
```

#### 搜索所有普通文件中含某个关键字的行内容
```
find . -type f -print | xargs grep "关键字"
```
#### 搜索当前目录下所有普通文件中含某个关键字的行内容
```
find . -name \* -type f -print | xargs grep "关键字"
```
注意，上述的"\"是转义<br>


#### 查当前目录下的所有普通文件
```
$ find . -type f -exec ls -l {} \;
```
#### 查当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出,并在/logs目录中查找更改时间在5日以前的文件并删除它们：
```
$ find logs -type f -mtime +5 -exec  -ok  rm {} \;
```

#### 查询当天修改过的文件
```
$ find  ./  -mtime  -1  -type f  -exec  ls -l  {} \;
```

#### 查询当前的终端窗口信息
```
$ who  |  awk  '{print $1"\t"$2}'

root    pts/0
root    pts/3
```

#### 在/tmp中查找所有的*.h，并在这些文件中查找“SYSCALL_VECTOR"，最后打印出所有包含"SYSCALL_VECTOR"的文件名
```
A) find  /tmp  -name  "*.h"  | xargs  -n50  grep SYSCALL_VECTOR
B) grep  SYSCALL_VECTOR  /tmp/*.h | cut   -d’:'  -f1| uniq > filename
C) find  /tmp  -name "*.h"  -exec grep "SYSCALL_VECTOR"  {}  \; -print
```

#### 要查找磁盘中大于3M的文件：
```
find . -size +3000k -exec ls -ld {} 
```

#### 将find出来的东西拷到另一个地方
```
find *.c -exec cp '{}' /tmp ';'
 
 如果有特殊文件，可以用cpio，也可以用这样的语法：
find dir -name filename -print | cpio -pdv newdir
```

#### 查找2004-11-30 16:36:37时更改过的文件
```
$ A=`find ./ -name "*php"` |  ls -l –full-time $A 2>/dev/null | grep "2004-11-30 16:36:37"
```

### df命令
#### 概览磁盘信息
```
df
```
#### 查看磁盘分区
```
$ df  -k |  awk '{print $1}' |  grep  -v  'none'
或
$ df  -k |  awk '{print $1}' |  grep  -v  'none' | sed  s"/\/dev\///g"

/dev/sda2
/dev/sda1
```
#### 参数
```
-a 全部文件系统列表

-h 方便阅读方式显示

-H 等于“-h”，但是计算式，1K=1000，而不是1K=1024

-i 显示inode信息

-k 区块为1024字节

-l 只显示本地文件系统

-m 区块为1048576字节

--no-sync 忽略 sync 命令

-P 输出格式为POSIX

--sync 在取得磁盘信息前，先执行sync命令

-T 文件系统类型

选择参数：

--block-size=<区块大小> 指定区块大小

-t<文件系统类型> 只显示选定文件系统的磁盘信息

-x<文件系统类型> 不显示选定文件系统的磁盘信息

--help 显示帮助信息

--version 显示版本信息
```

参考:<br>
http://zhaizhenxing.blog.51cto.com/643480/134885  (很乱)<br>
[find命令](http://www.jianshu.com/p/6e17ab338478)<br>
[find命令](http://blog.csdn.net/gexiaobaohelloworld/article/details/8206889)<br>