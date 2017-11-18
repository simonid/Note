## 本文收集了在使用git中遇到的一些bug。<br>

### gitbash Windows端中文显示乱码
```
$ git config --global core.quotepath false  		# 显示 status 编码
$ git config --global gui.encoding utf-8			# 图形界面编码
$ git config --global i18n.commit.encoding utf-8	# 提交信息编码
$ git config --global i18n.logoutputencoding utf-8	# 输出 log 编码
$ export LESSCHARSET=utf-8
# 最后一条命令是因为 git log 默认使用 less 分页，所以需要 bash 对 less 命令进行 utf-8 编码
```
修改`/etc/inputr`:
```
set output-meta on #bash中可以正常输入中文 set convert-meta off
```
如果ls命令不能正常显示中文：
```
alias ls='ls --show-control-chars --color=auto' #ls能够正常显示中文
```

### 分支内容未提交，git checkout错误
```
$ git checkout master
error: Your local changes to the following files would be overwritten by checkout:
        class/微机原理/微机原理课程.md
Please commit your changes or stash them before you switch branches.
error: The following untracked working tree files would be removed by checkout:
        other/google voice注册记.md
        other/双系统安装记.md
Please move or remove them before you switch branches.
Aborting
```
尝试切换分支出错。<br>
经历：在远程由一个微机原理课程文件，然后本地修改了内容并没有做其他任何操作，还有本地other目录下的两个文件也没有做add操作，然后就想要切换到master<br>
`git checkout`会更新HEAD指针指向该分支，用该分支只想的树更新暂存区和工作区，留意你的暂存区或工作区里面那些没有提交的修改，它会和你即将检出的分支产生冲突从而阻止git为你切换分支。ag：<br>
![]()
如果想要切换分支，就要删除这个冲突文件或者将冲突文件移动到当前源码之外的地方。<br>
百度一下，查找到相关方法：http://blog.csdn.net/iefreer/article/details/7679631<br>
如果希望保留生产服务器上所做的改动,仅仅并入新配置项，输入：<br>
```
git stash
git pull
git stash pop
git diff -w +文件名 来确认代码自动合并的情况
```
如果希望用代码库中的文件完全覆盖本地工作版本：
```
$ git reset --hard
$ git pull
```
如果想针对文件回退本地修改,使用`git checkout HEAD file/to/restore`
我选第一种。<br>
但是再次checkout又提示新bug：
```
$ git checkout master
error: Your local changes to the following files would be overwritten by checkout:
        class/微机原理/微机原理课程.md
Please commit your changes or stash them before you switch branches.
Aborting
```
这时要切换分支，就要先在master新建一个提交，或者用stash将当前工作区和暂存区压入git栈<br>
然后我做了一次版本回退：
```
$ git reset --hard
HEAD is now at 0bcf1bc google voice注册说明
```
这下可以切换了<br>
参考：http://www.iloveandroid.net/2015/10/05/studyGit_2/<br>
http://junch.github.io/%E6%8A%80%E5%B7%A7/2014/01/11/reprint-git-collision.html
<br>
[stackoverflow上关于Please, commit your changes or stash them before you can switch branches.
Aborting](https://stackoverflow.com/questions/22424142/error-when-changing-to-master-branch-my-local-changes-would-be-overwritten-by-c)
[合并分支](http://wiki.jikexueyuan.com/project/git-reference/branch-merge.html)