# 经典 git work follow

![git-work](/resource/git-work.png)

# 推荐的 git 多人协同模式

## 前置约定

```
1. 以代码为主的项目,项目主负责人在项目初始化时,确保中心仓库有两个远程分支: master, dev.其中 master 是稳定的主线分支; dev 是从 master 上切出来的功能开发分支.
2. 建议项目负责人上 gitlab,在项目 Settings 里面将默认分支设定为 dev.
```

## 几点约定

- 尽可能的在自己的本地分支上开发新功能.
- dev 分支功能开发分支,是可供运行与测试的 hot code 分支,请确保每个合并到 dev 分支的功能可运行,可测试.
- master 分支为稳定主线分支,以在 master 分支打 tag 的静态版本来上线或回滚.
- 专人负责将 dev merge 到 master, 打 tag 和上线.
- 除非有必要,不建议将本地分支推送到远端
- 合并代码的基本原则: 先人后己

## 多人协作


### 开发排期较长(约定2天以上为长开发周期)

```
1. 参与项目,从中心仓库拉取代码
    $ git clone 工作项目的gitlab路径
2. 创建本地开发分支,以 feature 为例,具体的开发分支尽量取有意义的英文名
    $ git checkout master       # 从 master 分支创建最新开发分支,限定当前开发分支不依赖其他任何分支代码
    $ git pull origin master    # 容错操作
    $ git branch feature        # 创建本地开发分支
    $ git checkout feature      # 切到本地的开发分支
    $ git push origin feature   # 多人参与一个开发分支,所以需要将此分支推送到远端
  仅参与开发的同学,在执行完第1步后,执行以下命令:
  $ git fetch --all
  $ git checkout feature
3. 在本地分支开发新功能
4. 将新功能代码提交到本地分支中
    $ git add 新文件/修改的文件
    $ git commit            # 提交到本地
5. 将新功能推送到远端
    $ git push origin feature
6. 发起提测邮件进入测试流程.如果有问题,请重复 2-5 步;如果测试确认没有问题,发起上线.邮件形式通知相关人员
7. 项目负责人做 code review,并将 feature merge 到 master,完成上线
    $ git checkout master
    $ git pull origin master    # master 上有可能存在 hotfixes 代码
    $ git merge feature
    $ git push origin master
    打 tag,推送 tag 到远端,执行上线.
8. 完成上线后将新上线的功能 merge 到 dev 分支,并通知 dev 分支开发的同学拉取最新代码
    $ git checkout dev
    $ git pull origin dev   # 拉取远端最新代码
    $ git merge master      # 将 master 上 feature 功能 merge 到 dev 分支
    $ git push origin dev   # 将 feature 功能推到 dev 的远端
9. 为了防止以后误操作,做好善后
    $ git push origin :feature  # 删除已经完成上线的远端分支
    $ git branch -D feature     # 删除本地分支
```

### dev 分支多人开发

适合短平快的开发模式,开发周期在2天以内

```
1. 参与项目,从中心仓库拉取代码
    $ git clone 工作项目的gitlab路径
2. 创建本地开发分支,以 feature 为例
    $ git branch feature    # 创建本地开发分支
    $ git checkout feature  # 切到本地的开发分支
3. 在本地分支开发新功能
4. 将新功能代码提交到本地分支中
    $ git add 新文件/修改的文件
    $ git commit            # 提交到本地
5. 切到 dev 分支,并更新 dev,将他人的成果 merge 到本地分支,确保代码与中心库一致
    $ git checkout dev
    $ git pull origin dev
    $ git checkout feature
    $ git merge dev
merge 如果有冲突,请解决冲突后,再执行第 4 步.如果没有冲突,刚可以将新功能代码 merge 到 dev 分支.
    $ git checkout dev
    $ git merge feature
6. 将新功能推送到远端
    $ git push origin dev
7. 发提测邮件,进入测试流程.如果有问题,请重复 2-6 步;如果测试确认没有问题,发起上线邮件申请.
8. 项目负责人 code reveiw,并将 dev merge 到 master,完成上线
    $ git checkout master
    $ git pull origin master
    $ git merge dev
    $ git push origin master
    打 tag,推送 tag 到远端,执行上线.
```

### hotfixes

难免线上出现紧急问题,紧急修复流程如下:

```
1. 拉取最新线上代码,并创建紧急修复分支
    $ git checkout master
    $ git pull origin master
    $ git branch hotfixes
    $ git checkout hotfixes
2. 完成修复并提交代码
    $ git add 修复的代码文件
    $ git commit    # 提交到本地
    $ git checkout master
    $ git merge hotfixes
    $ git push origin master
3. 在线上预览机回归
4. 完成上线后的善后
    $ git checkout master
    $ git pull origin master
    $ git checkout dev
    $ git pull origin dev
    $ git merge master  # 把 hotfixes 的代码 merge 到 dev 分支
    $ git branch -D hotfixes    # 删除临时分支,防止以后误操作
```

## 申请测试的提测邮件模板

`收件人`: rd, qa, pm, op

`标题`: 『提测』xx功能

`正文`:

1、提测项目与分支

```
# 以 pandora 为例子
项目: pandora
分支: dev
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
07125f0..0ebf345  master -> master
```

以上内容作用有2点:

- 供 op 在上线后确认线上代码版本与代码库一致,确保代码上线正确性
- 回滚操作的重要依据.

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
4. 将新功能分支 work-branch 合并到 develop
  $ git fetch --all     # 抓取所有远端分支的最新代码
  # 以下操作为容错操作,建议所有合并代码的同学按此指导操作,因为很多功能开分的同学会忘记合并 master 最新代码
  $ git checkout work-branch    # 切到待合并分支
  $ git pull origin work-branch # 拉取功能分支最新代码并在本地完成自动合并
  $ git fetch --all     # 抓取所有远端分支的最新代码
  $ git merge origin/master     # 将远端 master 最新代码合并进入功能开发分支
  $ git push origin work-branch # 将变化推送到远端
  # 正式开始合并操作
  $ git checkout develop    # 切换分支
  $ git pull origin develop # 拉取并在本地合并 develop 分支最新代码
  $ git fetch --all     # 抓取所有远端分支的最新代码,或者使用 git fetch -p,在抓取无端分支的同时,会清除无端已经删除而本地有记录的分支
  $ git merge origin/work-branch  # 将功能分支的远端 origin/work-branch 合并进 develop 本地分支
如果有冲突,用 `git log -p 冲突文件` 查看改动作者,尽量和原作者共同裁决冲突.
  $ git push origin develop # 将合并结果推送到远端,完成合并

按第 4 步如法炮制,可以完成 work-branch 到 master 的合并.将功能分支合并至 master,完成上线后一定要按第 4 步那样将 master 再合到 develop 中去,并删除远端的 work-branch.
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

# 将 svn 项目源码迁到 github/gitlab

```
$ svn log svn-pro-src | grep '^r' | awk '{print $3 " = " $3 " <" $3 "@UFO.com>"}' | sort -u > users.txt
$ git svn clone svn-pro-src --authors-file=users.txt --no-metadata
$ cd project
$ git remote add origin github/gitlab远程源
$ git push -u origin master
```

