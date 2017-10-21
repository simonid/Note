# Linux 打包解压指令

压缩文件类型：<br>
```
*.gz                # gzip 程序压缩的文件
*.bz2               # bzip2 程序压缩的文件
*.tar               # tar 命令打包, 但并没有压缩过的文件
*.tar.gz            # tar 命令打包, 并且经过 gzip 的压缩的文件
*.tar.bz2           # tar 命令打包, 并且经过 bzip2 的压缩的文件
```
和tar相关的参数：
```
-c: 建立压缩档案 
-x：解压 
-t：查看内容 
-r：向压缩归档文件末尾追加文件 
-u：更新原压缩包中的文件
```
这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。
```
-z：有gzip属性的 
-j：有bz2属性的 
-Z：有compress属性的 
-v：显示所有过程 
-O：将文件解开到标准输出
下面的参数-f是必须的 
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
```
## tar
tar添加压缩文件：格式.tar<br>
#### tar -cf all.tar *.jpg 
　　这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包 
，-f指定包的文件名。 
#### tar -rf all.tar *.jpg
　　这条命令是将所有.jpg的文件增加到all.tar的包里面去。-r是表示增加文件
#### tar -uf all.tar logo.gif 
　　这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
#### tar -tf all.tar 
　　这条命令是列出all.tar包中所有文件，-t是列出文件的意思 
#### tar -xf all.tar 
　　这条命令是解出all.tar包中所有文件，-x是解开的意思<br>

### 如果格式是.tar.gz，那么就在前面.tar基础上加上z的选项 ; 
### 如果格式是.tar.bz2，那么就在前面.tar基础上加上j的选项

### 对于.gz结尾的文件 
　　gzip -d  /path  all.gz   //path是解压后文件放置的位置
　　gunzip  -d  /path  all.gz  
对于.bz2结尾的文件 
　　bzip2 -d  /path  all.bz2 
　　bunzip2 -d  /path  all.bz2 
## zip
### zip all.zip *.jpg 
　　这条命令是将所有.jpg的文件压缩成一个zip包 
### unzip all.zip 
　　这条命令是将all.zip中的所有文件解压出来，也可加上-d配置路径
## rar
### rar a all *.jpg 
　　这条命令是将所有.jpg的文件压缩成一个rar包，名为all.rar，该程序会将.rar 
扩展名将自动附加到包名后。 
　　# unrar e all.rar 
	这条命令是将all.rar中的所有文件解压出来 

## 总结一波
### 解压：
```
1、*.tar 用 tar -xvf 解压 
2、*.gz 用 gzip -d或者gunzip 解压 
3、*.tar.gz和*.tgz 用 tar -xzf 解压 
4、*.bz2 用 bzip2 -d或者用bunzip2 解压 
5、*.tar.bz2用tar -xjf 解压 
6、*.Z 用 uncompress 解压 
7、*.tar.Z 用tar -xZf 解压 
8、*.rar 用 unrar e解压 
9、*.zip 用 unzip 解压
```
### 压缩：
```
tar -cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg 
tar -czf jpg.tar.gz *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz 
tar -cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2 
tar -cZf jpg.tar.Z *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z 
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux 
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
```