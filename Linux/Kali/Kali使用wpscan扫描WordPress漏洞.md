wpscan是一款可以检测wordpress漏洞的软件，kali完整版自带，树莓派需要先下载。<br>

## 基本操作
### 基本扫描一个站点
通过这个命令，可以大致扫描一遍网站，显示出网站版本、主题等信息(并没有扫描用处),并且给出一部分漏洞信息 <br>
```
wpscan --url url
或
wpscan -u url
```
如果输出的内容中含有红色的[!]标识符，说明这是一个可利用的漏洞，接着下面的操作<br>

### 枚举查找用户信息
```
wpscan -u url --enumerate u
或
wpscan -u url -e u vp 
```
扫描成功后会出现表格，里面有id、Login、Name信息<br>

### 破解用户密码
开了4个线程扫描：
```
wpscan -u url --wordlist ~/wordlist.txt --username username --threads 4 
```

为WPScan指定一个wordlist文件，使用--wordlist 选项<br>

一些参数说明：
```
--enumerate | -e [option(s)] Enumeration.
option :
u – usernames from id 1 to 10
u[10-20] usernames from id 10 to 20 (you must write [] chars)
p – plugins
vp – only vulnerable plugins
ap – all plugins (can take a long time)
tt – timthumbs
t – themes
vt – only vulnerable themes
at – all themes (can take a long time)
Multiple values are allowed : “-e tt,p” will enumerate timthumbs and plugins
```