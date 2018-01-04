# 经典 git work follow

![git-work](/resource/git-work.png)

# 推荐的 git 多人协同模式

## 最佳实践建议

- 尽可能的在自己的本地分支上开发新功能.
- 功能开发分支,是可供运行与测试的`hot code`分支,请确保每个提交测试分支的功能可运行,可测试,并且包含了最新的`master`分支功能.
- `master`分支为稳定主线分支,以在`master`分支打`tag`的静态版本来上线或回滚.
- 专人负责将功能开发分支**merge**到`master`,**code review**,打`tag`和上线.
- 千万不要使用`git add *`,`git add .`,`git commit -a`,除非你知道自己在做什么
- 尽量使用`git commit -i`,`git commit`,并唤起默认编辑器,可以有效防止误提交
- 分支合并时尽量使用`git merge --no-ff origin/branch-name`,尤其是将功能分支代码合并进`master`分支时,目的是人为制造分叉点,清晰描述合并过程

# git 多分支合并最佳实践

```
以 work-branch 为例,约定此分支为从最新的 master 分支创建的功能开发分支.

1. 创建功能开发分支
  $ git checkout master     # 切到 master 分支
  $ git pull origin master  # 拉取最新 master 分支代码,因为可能有新功能已经合并上线
  $ git checkout -b work-branch # 创建并切换新的分支
2. 在功能开发分支开发新功能,建议频繁按小功能完成点提交代码,完成自测
3. 将 master 最新功能合并进功能开发分支,并推送到远端
  $ git fetch --all     # 抓取所有远端分支的最新代码
  $ git merge origin/master     # 将远端 master 最新代码合并进入功能开发分支
如果有冲突,用 `git log -p 冲突文件` 查看改动作者,尽量和原作者共同裁决冲突.
  $ git push origin work-branch     # 将功能开发分支推送到远端
4. 将新功能分支 work-branch 合并到 master
  $ git fetch --all     # 抓取所有远端分支的最新代码
  # 以下操作为容错操作,建议所有合并代码的同学按此指导操作,因为很多功能开分的同学会忘记合并 master 分支上的最新代码
  $ git checkout work-branch    # 切到待合并分支
  $ git pull origin work-branch # 拉取功能分支最新代码并在本地完成自动合并
  $ git fetch --all     # 抓取所有远端分支的最新代码
  $ git merge origin/master     # 将远端 master 最新代码合并进入功能开发分支
  $ git push origin work-branch # 将变化推送到远端
  # 正式开始合并操作
  $ git checkout master    # 切换分支
  $ git pull origin master # 拉取并在本地合并 master 分支最新代码
  $ git fetch --all     # 抓取所有远端分支的最新代码,或者使用 git fetch -p,在抓取远端分支的同时,会清除远端已经删除而本地有记录的分支
  $ git merge origin/work-branch  # 将功能分支的远端 origin/work-branch 合并进 master 本地分支
如果有冲突,用 `git log -p 冲突文件` 查看改动作者,尽量和原作者共同裁决冲突.
  $ git push origin master # 将合并结果推送到远端,完成合并
5. (可选)打 tag 上线,删除已合并的本地分支和远端分支
  $ git tag -a v1.0.0 -m 'release-v1.0.0'   # 打 tag,请具体对待
  $ git push --tags                         # 推送 tags 到远端
  $ git branch -d work-branch               # 删除本地已经合并的功能开发分支
  $ git push origin :work-branch            # 删除远端已合并的分支
```

# 版本控制之道

1. 先人后己<br/>
    合并代码的基本原则: 先人后己
1. 小批量提交<br/>
    小成果,及时提交,尽早发布
1. 走心的提交日志<br/>
    提交日志其实和代码同样重要,甚至是一个开发者的门面,所以**请珍惜你的脸面**

## 申请测试的提测邮件模板

`收件人`: rd, qa, pm, op

`标题`: 『提测』xx功能

`正文`:

1、提测项目与分支

```
# 以 pandora 为例子
项目: pandora
分支: work-branch
```

2、功能点说明

```
  1). 例行升级维护
  2). 其他
```

3、预计上线时间和其他说

```
# 请简单说明
```

## 申请上线的邮件模板

回复『提测』邮件,将标题换为『上线』

在正文件中贴上最后提交记录,形如:

```
0ebf345  master
```
以上内容作用有2点:

- 供 op 在上线后确认线上代码版本与代码库一致,确保代码上线正确性
- 回滚操作的重要依据.

# 将 svn 项目源码迁到 github/gitlab

```
$ svn log svn-pro-src | grep '^r' | awk '{print $3 " = " $3 " <" $3 "@UFO.com>"}' | sort -u > users.txt
$ git svn clone svn-pro-src --authors-file=users.txt --no-metadata
$ cd project
$ git remote add origin github/gitlab远程源
$ git push -u origin --all
$ git push -u origin --tags
```

# git 交叉合并后出现的灵异现象

```
所谓的交叉合并类似场景为,从 master 上创建分支 dev和test,在 test 分支开发新功能,他人的新功能被合并到 dev 分支;这时将 dev 合并进 test,然后将 test 合并至 dev;之后将 test 合到 master 上线,完成上线后又将 master 合至 dev.
这样操作后出现的诡异现现象目前遇到两种:

1. 递归合并.合并两个分支时,没有任何文件需要合并,仅提示进行了递归合并. git diff 两个无端分支时没有差异,但人工查看代码时发现,有分支代码并没有合并进去.

提示语如下:
Already up-to-date!
Merge made by the 'recursive' strategy.

2. 每次合并都会出现几个与本分支功能无关的代码冲突,反复解决,反复冲突.

建议: 合并代码时一定要搞清楚合并方向,完成合并后在 push 之前,做好必要的 code review,利人利己.
```

# It looks like git-am is in progress. Cannot rebase.

[see](http://lamb-mei.com/550/git-git-am-%E6%9B%B4%E6%96%B0%E9%8C%AF%E8%AA%A4%E8%99%95%E7%90%86%E6%96%B9%E5%BC%8F/)

## 修复冲突

```
$ git apply PATCH --reject
$ edit edit edit
$ git add FIXED_FILES
$ git am --resolved
```

## 放弃更新

```
$ git am --abort
```

## 直接刪除 rebase-apply

```
$rm -rf .git/rebase-apply
```

# rebase 还是 merge?

`rebase`和`merge`是代码观察者查看代码线性变化的不同维度,对最终合并的代码来讲,没有任何区别.`merge`是完全的以时间为线性轴,体现出源码在不同时间点上发生的变化;而`rebase`是提交源码作者为轴,将同一作者的提交在目标源码的最后基线上线性的合并,表现为分支功能的代码提交是线性的,而不是与协作者提交相穿插.

建议同分支合并使用`rebase`,跨分支合并使用`merge`,这样不会丢失关键的`merge message`.

# [使用自己生成SSL证书时，Git报错的解决办法](http://my.oschina.net/jlan/blog/506065)

git clone https://....

时报错，说证书校验有问题：

最简单的解决方法是加一个环境变量：

```
$ export GIT_SSL_NO_VERIFY=1
$ git config --global http.sslVerify false
```

# [OS X Yosemite(10.10)下无法使用git svn解决方法](http://blog.puhao.me/%E5%90%90%E6%A7%BD/OS-X-Yosemite(10.10)%E4%B8%8B%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8git-svn%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/)

自己在OS X下面没有找到好用的免费SVN客户端，又不想使用破解软件，更因为觉得SourceTree实在是太好，对它爱不离手，因此当自己遇到代码管理使用SVN的时候，还是习惯使用git svn做一个桥接，让代码在我本地是通过GIT来管理的。

近日电脑系统升级到了Yosemite后，使用git svn clone命令报错，错误信息如下：
`Can’t locate SVN/Core.pm in @INC (you may need to install the SVN::Core module)`

添加Perl库链接
这个错误以前在10.9的时候也遇到过，Google一下就找到了[解决方案](http://www.contrid.co.za/2014/10/solved-os-x-yosemite-git-svn-broken/)。那个时候Perl库用的是5.16版本，现在系统升级到了5.18版本后，跟以前一样，添加两条软连接命令：

```
sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/SVN /System/Library/Perl/Extras/5.18/SVN
sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/auto/SVN/ /System/Library/Perl/Extras/5.18/auto/SVN
```

# [OS X El Capitan下git-svn无法使用](https://paulschreiber.com/blog/2015/11/09/fixing-git-svn-on-os-x-el-capitan/)

Unfortunately, the old solutions no longer work due to El Capitan’s System Integrity Protection:

```
$ sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/SVN /System/Library/Perl/Extras/5.18/SVN
ln: /System/Library/Perl/Extras/5.18/SVN: Operation not permitted
```

While you can disable SIP, that’s unnecessary in this case.

Here’s how you get git-svn working:

```
$ sudo mkdir /Library/Perl/5.18/auto
$ sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/SVN /Library/Perl/5.18/darwin-thread-multi-2level
$ sudo ln -s /Applications/Xcode.app/Contents/Developer/Library/Perl/5.18/darwin-thread-multi-2level/auto/SVN /Library/Perl/5.18/auto/
```

You can’t write to /System, but you can still write to /Library.

# `git`使用`https`方式记住密码

[see git-scm](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)
[see other](http://www.yalewoo.com/git_https_password.html)

git使用https方式进行连接时，默认每次推送时都要输入用户名和密码。

`git config credential.helper store`

为当前仓库设置记住密码，设置后，只要在推送一次，以后就不需要用户名和密码了

# 查看某次合并操作中的代码改动

1. 找到合并操作对应的log

    ```
    $ git log
    ```
2. 有合并操作的地方,会产生形如以下的日志:

    ```
    commit 1a079789427f9aa7b47f7dc699355947cc9e83ff
    Merge: 0e9fa5d e5521e2
    ...
    ```
3. 通过版本号区间的方式查看合并操作中代码的改动

    ```
    $ git diff 0e9fa5d e5521e2
    ```

# git管理二进制大文件

## 推荐方案: `git lfs`

目前`gitlab`,`github`,`coding.net`均支持.

[官方网站](https://git-lfs.github.com/) [github](https://github.com/git-lfs/git-lfs)

上面有详细的使用说明,请移步查看,此文档不再摘抄.
