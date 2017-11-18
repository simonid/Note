# git删除内容
## git删除本地缓存
如果一次操作不小心将不想add的文件add到了暂存区，那么可以用下面指令删除暂存区文件：<br>
```
git rm --cached 文件名
```
上面的操作并不会删除本地的文件，只是删除暂存区缓存罢了。<br>
如果是需要删除本地文件，那么使用:
```
git rm 文件名
```

## git删除远程内容
删除git远程仓库的内容，事实上也是做一次修改提交。<br>
```
git rm -r -n --cached 目录名  // -n：加上这个参数，执行命令时是不会删除任何文件的，只是展示该命令要删除的目录列表信息
git rm -r --cached 目录名 //删除缓存区的目录
git commit -m "xxx" //提交修改信息
git push origin master //提交到远程
```
上述操作的确删除了远程文件，但是实际上，文件过去提交的信息还是没有被删除。<br>

### git删除本地分支
```
git branch -d xxx
```
### git删除远程分支
```
git push origin --delete xxx
或
git push origin :xxx  //推送空分支到制定远程分支，相当于删除
```
### git删除远程tag
```
git push origin --delete tag xxx
```

