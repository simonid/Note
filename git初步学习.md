# git初步学习

## 创建一个github项目
创建github项目在网页中进行，在github可以找到，需要留意的是github中关于许可证的要求，用一张图简要概括：

![开源许可证](https://github.com/simonid/Note/blob/master/img/%E5%BC%80%E6%BA%90%E8%AE%B8%E5%8F%AF%E8%AF%81.jpg)

关于开源许可证的更多内容，可以参考知乎的讨论：
[开源许可证都有什么区别,一般开源项目用什么许可证?](https://www.zhihu.com/question/28292322)

## 初始化本地的git环境
本人使用的是ubuntu17,windows用户有官方图形工具使用。
```
sudo apt-get install git
```
### 添加ssh密钥对
如果不确定是否本地在之前已经生成密钥，可以查看：
```
cd ~/.ssh
ls
```
密钥文件有两个，但是只有一个才是需要用的， id_rsa.pub 或 id_dsa.pub 文件，详细原理自行搜索
#### 创建ssh key
```
ssh-keygen -t rsa -C "your_email@example.com"
```
-t 指定密钥类型，默认是 rsa ，可以省略。
-C 设置注释文字，比如邮箱
然后终端会提示你命名密钥文件，一路回车保持默认名就好
然后就有生成了密钥的信息：
```
Your identification has been saved in /home/simon/.ssh/id_rsa.
Your public key has been saved in /home/simon/.ssh/id_rsa.pub.
```
将id_rsa.pub文件的内容复制到github个人设置中
然后输入下面测试连接：
```
ssh -T git@github.com
```
如果有You've successfully authenticated提示就成功了

## 提交第一个项目
### 初始化
我们先初始化本地的版本库，如果你已经在github初始化了readme并且打算修改它，可以先clone下来
```
git init
git add .  //添加该路径下所有文件到本地缓存区
git commit -m "first commit"   //注意，commit是add和push之间必须有的操作，否则会报错
```
至此完成了本地版本库初始化，下面推送到远程
### 删除本地缓存区文件
如果把不想推送的文件加到了本地，那么应该删除<br>
首先要`git status`来确认状态。<br>
然后`git rm --cached "文件路径"`就可以删除了，这样不删除物理文件，仅将该文件从缓存中删除。<br>
还有另外一种 git rm --f  "文件路径"，不仅将该文件从缓存中删除，还会将物理文件删除（不会回收到垃圾桶
### 推送到远程
```
git remote add origin git@example.com  //将本地与远程仓库关联
git push -u origin master  //将本地所有文件推送到远程master分支，也可以不加u
```
至此完成第一个项目初步推送

#### 其他一些指令：
```
git status   //查看仓库当前状态，可以检查本地未添加到本地仓库的文件
git log --pretty=online  //现实最近的提交日志，格式化输出信息
```
#### 分支管理
```
git branch example  //创建名为example的分支
git checkout example //切换到example分支

上述两句可以简练为：git checkout -b example

git merge example  //合并分支

git log --graph --pretty=oneline --abbrev-commit   //查看分支合并情况

git branch -d example  //删除example分支
```

#### 解决冲突
有时候，不同的设备对某个文件的同一处地方做了不同修改，那么，晚push的那台设备会提示错误，那么应该按照下列方法解决：
```
git pull  //从远程拉下来
git diff  //对比冲突
修改冲突后重新git push
```

另外附上git冲突解决的一种方案：
```
1.git status           //查看当前的分支状态
2.git diff             //查看修改的分支内容 + 检查语法格式
3.mm -B                //编译
4.git add .            //添加修改的内容
5.git status           //查看当前状态【确保修改生效】
6.git log              //查看提交记录,有无最新提交
7.git commit           //提交到本地代码库

  ---参照9-temp/ellsion/环境搭建/errit投入
  ---Ctrl + O          //写入
  ---Ctrl + X          //退出

8.git log              //查看所有投入记录 + 记录提交的Commitid
9.git show commitid    //查看某次提交内容

10.git fetch fujitsu   //从远程获取最新版本到本地

//解决本地冲突
11.git diff fujitsu/MM/salvia/public/develop  //查看是否有冲突.若是有多余的修改---冲突执行 12,否则执行13

12.git rebase fujitsu/MM/salvia/public/develop //文件冲突的必备操作

13.git status          //查看当前状态
14.git log
15.git diff fujitsu/MM/salvia/public/develop   //再次确认有无冲突
16.mm -B
17.git push fujitsu HEAD:refs/for/MM/salvia/public/develop  //提交记录上传至远程gerrit服务器

上述内容来自csdn：Git 提交远程(包含本地远程的冲突解决) http://blog.csdn.net/cl18652469346/article/details/53256988
```
另外一种冲突解决思路：[解决因为本地代码和远程代码冲突，导致git pull无法拉取远程代码的问题](http://www.cnblogs.com/huanyou/p/6654813.html)
[git stash指令](http://blog.csdn.net/wh_19910525/article/details/7784901)

#### 回到过去
如果git仓库实在太乱，想退回到过去的版本，可以这样：
```
git reset --hard 想要回退的commit的id
```
如果又后悔了，跳回到现在的方式也类似，
可以用`git reflog`查看最近的commit信息


更多git命令，可以查看：
[Git 命令总结](https://zhuanlan.zhihu.com/p/25892137?utm_source=wechat_session&utm_medium=social&from=singlemessage)<br>
[廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)<br>