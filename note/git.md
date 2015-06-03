# 经典 git work follow

![git-work](resource/git-work.png)

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

## 多人协作

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
7. 发起提测.如果有问题,请重复 2-6 步;如果测试确认没有问题,发起上线.
8. 将 dev merge 到 master,并上线
    $ git checkout master
    $ git pull origin master
    $ git merge dev
    $ git push origin master
    打 tag,推送 tag 到远端,执行上线.
```

