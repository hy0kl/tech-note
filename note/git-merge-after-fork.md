```
背景:
fork 了一个感兴趣的项目,例如 memcached,它的开发者很活跃,很快会遇到将主项目合并到 fork 出的分支中来的实际问题,远程分支合并.

具体步骤:
1. 显示数据仓库
$ git remote -v
origin  git@github.com:hy0kl/memcached.git (fetch)
origin  git@github.com:hy0kl/memcached.git (push)
可以看到当前的git库中,有一个默认的远程数据仓库,后面的(fetch)和(push)是 pull 抓取数据和 push 推送数据的地址.
我们要合并的是 git://github.com/memcached/memcached.git

2. 方法
$ git checkout -b  feature    # 创建并切换到新的 feature 分支来进行 merge 操作,降低操作风险
$ git remote -v   # 查看当前的远程仓库配置
$ git remote add dev-memcached git://github.com/memcached/memcached.git
$ git remote -v   # 查看新增后的变化,形如:
    dev-memcached   git://github.com/memcached/memcached.git (fetch)
    dev-memcached   git://github.com/memcached/memcached.git (push)
    origin  git@github.com:hy0kl/memcached.git (fetch)
    origin  git@github.com:hy0kl/memcached.git (push)
$ git fetch dev-memcached # 将 dev-memcached(即 memcached 最新源码) fetch 下,准备进行 merge
$ git merge dev-memcached/master  # 将 dev-memcached 下 maste 分支的内容 merge 到当前的数据仓库.其他的分支也可以,将master替换即可
$ git remote rm dev-memcached     # 删除远程数据仓库,避免以后误操作

3. 合并代码
之后如果代码冲突解决完成,可以按照正常的代码提交流程进行代码提交:
$ git commit -am "msg"
$ git pull
$ git push

如果是 master 分支开发,则
$ git checkout master
$ git merge feature
# 有冲突,解决后先 commit
$ git push  # 完成

PS: 感谢姜同学
```
