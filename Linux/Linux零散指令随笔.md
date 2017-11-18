# Linux零散指令随笔
## 注意，这里的内容都是一些零散的指令收集


<hr>
<hr>

### Linux查看端口占用
安装工具netstat(net-tools)<br>
使用命令：`netstat -apn`显示所有进程和端口使用
```
netstat -apn | grep 8080
```

### grep
在文件中查找字符串（不区分大小写）
```
grep  -i  "yes" filename
```
输出匹配到的行和其中的后n行：
```
grep -A n -i "yes" filename
```
在某个目录下递归查询包含指定字符串的文件：
```
grep -r "字符串" 目录
```

### export
临时设置环境变量
```
export PATH=$PATH:/home/simon/app
```
拓展：永久设置<br>
```
1.修改/etc/profile文件
加入
export PATH="$PATH:/home/simon/app"
2.修改~/.bashrc文件
加入
export PATH="$PATH:/home/simon/app"
```

### nohup
后台挂起任务，且不输出内容
```
nohup command >/dev/null 2>&1 &
```

### find
查找指定文件（不区分大小写）
```
find -iname  "文件"
```
查找某个目录下的所有空文件
```
find  目录 -empty
```

### sort
升序排列文件内容
```
sort 文件
```
降序排列文件内容
```
sort -r 文件
```
但是前面这些都是没什么意义的，如果要对文件内容排序，还需要其他额外内容，比如，如果以第三个字段对passwd文件进行排序
```
sort -t: -k 3n /etc/passwd 
```

### tail
默认显示文件最后10行内容<br>
-n指定显示行数，-f实时查看<br>

### xarg
&nbsp;&nbsp;&nbsp;&nbsp;xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。它擅长将标准输入数据转换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。xargs也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。xargs的默认命令是echo，空格是默认定界符。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。xargs是构建单行命令的重要组件之一。[参考](http://man.linuxde.net/xargs)<br>
多行转单行输出：
```
cat file | xargs
```
单行转多行输出：
```
cat file | xargs -n3  //3是指列数
```
-d输出定界符：
```
echo "HelloXWorld" | xargs -dX
Hello World
```
复制所有jpg文件到指定目录：
```
ls *.jpg | xargs -n1 -I cp {} 目录
```
删除文件(如果单纯用rm删除，可能会报错：bin/rm Argument list too long)：
```
find . -type f - name "*.log" - print0 | xargs -0 rm -f
```
其中xarg -0是将\0作为定界符
查找所有jpg文件并压缩：
```
find . -type f -name "*.jpg" -print | xargs tar -czvf img.tar.gz
```
如果一个文件内全是下载链接，需要全部下载：
```
cat file | xargs wget -c
```

### seq
产生指定范围的数<br>
```
seq [选项]... 尾数 
seq [选项]... 首数 尾数 
seq [选项]... 首数 增量 尾数
选项
-f, --format=格式 使用printf 样式的浮点格式 -s, --separator=字符串 使用指定字符串分隔数字（默认使用：\n） 
-w, --equal-width 在列前添加0 使得宽度相同，不能和-f一起
```
demo:<br>
产生9到12的数：
```
seq -f"%3g" 9 12
%后面指定数字的位数 默认是%g，%3g那么数字位数不足部分是空格。
seq -f"str%03g" 9 11 
str009 
str010 
str011
```
生成数字间插入空格：
```
seq -s"`echo -e " "`" 9 12
9 10 11 12
```

### free
内存查看命令，默认以字节为单位输出<br>
参数：-g为GB，-m为MB，-k为KB，-b为字节，-t为所有内存的汇总<br>

### df
产看硬盘使用状况<br>
参数：<br>
-k:字节为单位<br>
-h:更方便看<br>
-T：显示文件系统类型
<br>

### mount
关在一个文件系统到目录<br>
```
mkdir /pan
mount /dev/sdb1 /pan
```

### uname
显示内核名称、主机名、内核版本号、处理器类型之类的信息<br>

### locate
显示一个文件路径<br>


### wathis
显示某个命令的描述信息，当不知道一个命令是什么作用时可以使用<br>





参考：<br>
[Linux命令大全 ](http://man.linuxde.net/)<br>