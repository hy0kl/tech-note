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
