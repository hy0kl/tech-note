# 查看远程分支

[FROM](http://zengrong.net/post/1746.htm)

加上-a参数可以查看远程分支，远程分支会用红色表示出来（如果你开了颜色支持的话）：

```
# git branch -a
  master
  remote
  tungway
  v1.52
* zrong
  remotes/origin/master
  remotes/origin/tungway
  remotes/origin/v1.52
  remotes/origin/zrong
```

# 删除在本地有但在远程库中已经不存在的分支

[FROM](http://blog.csdn.net/xhl_will/article/details/8450193)

查看远程库的一些信息，及与本地分支的信息。有时候可能遇到如下情况:

```
$ git remote show origin

* remote origin
  Fetch URL: ... .git
  Push  URL: ... .git
  HEAD branch: master
  Remote branches:
    dev                     tracked
    jqmobi                  tracked
    master                  tracked
    refs/remotes/origin/3.1 stale (use 'git remote prune' to remove)
    refs/remotes/origin/tc  stale (use 'git remote prune' to remove)
    refs/remotes/origin/xhl stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev    merges with remote dev
    master merges with remote master
  Local refs configured for 'git push':
    dev    pushes to dev    (up to date)
    jqmobi pushes to jqmobi (up to date)
    master pushes to master (up to date)
```

其中3.1, tc, xhl三个分支在远程库已经不存在了（你之前从远程库拉取过，可能之后被其他人删除了，你用 git branch -a 也是不能看出它们是否已被删除的），这时候我们可以删除本地库中这些相比较远程库中已经不存在的分支：

```
$ git remote prune origin
warning: more than one branch.master.remote
Pruning origin
URL: ... .git
 * [pruned] origin/3.1
 * [pruned] origin/tc
 * [pruned] origin/xhl
```

再 git branch -a 查看.搞定

新版本可以使用以下命令:
$ git fetch -p

# windows warning: CRLF will be replaced by LF.

参考: [1](http://stackoverflow.com/questions/17628305/windows-git-warning-lf-will-be-replaced-by-crlf-is-that-warning-tail-backwar) [2](http://blog.csdn.net/feng88724/article/details/11600375)

解决:

```
$ git config --global core.autocrlf false
```

[备份记录](http://blog.csdn.net/feng88724/article/details/11600375):

```
warning: LF will be replaced by CRLF | fatal: CRLF would be replaced by LF

遇到这两个错误， 基本上都是叫你将 autocrlf 设置为 false. 但是我觉得这样很不妥。

如果你的源文件中是换行符是LF，而autocrlf=true, 此时git add就会遇到 fatal: LF would be replaced by CRLF 的错误。有两个解决办法：
1. 将你的源文件中的LF转为CRLF即可【推荐】
2. 将autocrlf 设置为 false

如果你的源文件中是换行符是CRLF，而autocrlf=input,  此时git add也会遇到 fatal: CRLF would be replaced by LF 的错误。有两个解决办法：
1. 将你源文件中的CRLF转为LF【推荐】
2. 将autocrlf 设置为true 或者 false

我的建议：在Mac上设置 autocrlf = input, 在Windows上设置autocrlf = true（默认值）。
------------------------------------------------------------------------------------------
这样的话，
Windows：（true）
提交时，将CRLF 转成 LF再提交；
切出时，自动将LF 转为 CRLF;

MAC/Linux:  (input)
提交时,   将CRLF 转成 LF再提交；
切出时，保持LF即可

这样即可保证仓库中永远都是LF. 而且在Windows工作空间都是CRLF, 在Mac/Linux工作空间都是LF.
-----------------------------------------------------------------------------------------

core.autocrlf

假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。

Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。用core.autocrlf来打开此项功能，如果是在Windows系统上，把它设置成true，这样当签出代码时，LF会被转换成CRLF：

$ git config --global core.autocrlf true
Linux或Mac系统使用LF作为行结束符，因此你不想 Git 在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把core.autocrlf设置成input来告诉 Git 在提交时把CRLF转换成LF，签出时不转换：

$ git config --global core.autocrlf input
这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。

如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置false取消此功能，把回车符记录在库中：

$ git config --global core.autocrlf false
```

# git push force 以后应该如何 pull 呢

[参考一](https://ruby-china.org/topics/5163)

```
$ git fetch origin
$ git reset --hard origin/master
```

[参考二](http://stackoverflow.com/questions/1125968/force-git-to-overwrite-local-files-on-pull)

```
$ git fetch --all
$ git reset --hard origin/master
```

# git 拉取远程分支

[参考](http://git-scm.com/book/zh/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)

```
$ git checkout --track origin/remote-branch
```

# git log 实现 svn log -v 效果

```
$ git log --stat
```

# 垃圾回收

$ git gc

# git pull 发生冲突, 让服务端代码覆盖掉本地代码

```
$ git fetch --all
$ git reset --hard origin/master

$ git fetch downloads the latest from remote without trying to merge or rebase anything. Then the $git reset resets the master branch to what you just fetched.
```

# git 忽略已经跟踪文件的改动

```
# 本地修改不提交到远程仓库
$ git update-index --assume-unchanged 文件名
# 取消本地忽略,上面命令的反操作
$ git update-index --no-assume-unchanged 文件名
# 查看本地仓库哪些文件被加入忽略列表
$ git ls-files --other --exclude-standard
```

# 如何只克隆 git 仓库中的一个分支？

```
$ git clone -b <branch> <remote_repo>   # git clone -b 指定的分支名字
```

# git tags 使用 远程创建git tags 删除（小抄版）

[see](http://hlee.iteye.com/blog/1616461)

```
0. 获取远程tag
$ git clone 代码仓库
$ git ls-remote --heads origin

$ git pull origin refs/tags/v0.13.3
$ git pull origin refs/heads/2-2-stable

1. 本地创建tags
$ git tag -a v1.1 -m "new release"

1.5 查看本地tags
$ git tag -l

2. push到服务器端
$ git push --tags
或
$ git push origin v1.1

2.1 删除本地tag
$ git tag -d v1.1

3. 删除服务器端远程tag
$ git push origin :refs/tags/v1.1
或
$ git push origin --delete tag <tagname>

4. 查看远程服务器端
$ git remote show origin
$ git ls-remote --heads origin
$ git ls-remote --tags  # 列出远端的所有 tags
```

# git 统计相关

## git 代码行统计命令集[参考](https://segmentfault.com/a/1190000002434755)

```
# 添加或修改的代码行数
$ git log --stat | perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/;'
# 或者
$ git log  --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2; } END { printf "added lines: %s removed lines : %s total lines: %s\n", add, subs, loc; }'

# 总提交数
$ git log --oneline | wc -l

# 贡献者数量统计
$ git log --pretty='%aN' | sort -u | wc -l

## 仓库提交者[按邮箱]排名TOP10
$ git log --pretty=format:%ae | awk '{ ++c[$0]; } END { for(cc in c) printf "%5d %s\n", c[cc], cc; }' | sort -u -n -r | head -n 10

# 仓库提交者[按名字]排名TOP10
$ git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 10

# 统计某人的代码提交量,包括增加/删除
$ git log --author="$(git config --get user.name)" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2; } END { printf "added lines: %s removed lines : %s total lines: %s\n", add, subs, loc; }'
```

# 问题描述

新建并推送了远程分支后
```
develop$ git pull
You asked me to pull without telling me which branch you
want to merge with, and 'branch.develop.merge' in
your configuration file does not tell me, either. Please
specify which branch you want to use on the command line and
try again (e.g. 'git pull <repository> <refspec>').
See git-pull(1) for details.

If you often merge with the same branch, you may want to
use something like the following in your configuration file:

    [branch "develop"]
    remote = <nickname>
    merge = <remote-ref>

    [remote "<nickname>"]
    url = <url>
    fetch = <refspec>

See git-config(1) for details.
```

# 解决方法:

```
$ git config --global branch.master.remote origin
$ git config --global branch.master.merge refs/heads/master

develop$ git branch --set-upstream develop origin/develop
Branch develop set up to track remote branch develop from origin.
```

.git/config 配置文件会加入以下内容:

```
[branch "develop"]
    remote = origin
    merge = refs/heads/develop
```

# 查询当前分支的最新版本号(commit id)

```
$ git rev-parse HEAD
```
