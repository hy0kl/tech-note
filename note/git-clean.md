# 实用小工具

[delete-forever](https://github.com/hy0kl/profile/blob/master/local/bin/delete-forever.sh)

# git永久性删除文件，包括日志清除，让文件无法再次恢复

[git永久性删除文件，包括日志清除，让文件无法再次恢复](http://www.zhetenga.com/view/git%E6%B0%B8%E4%B9%85%E6%80%A7%E5%88%A0%E9%99%A4%E6%96%87%E4%BB%B6%EF%BC%8C%E5%8C%85%E6%8B%AC%E6%97%A5%E5%BF%97%E6%B8%85%E9%99%A4%EF%BC%8C%E8%AE%A9%E6%96%87%E4%BB%B6%E6%97%A0%E6%B3%95%E5%86%8D%E6%AC%A1%E6%81%A2%E5%A4%8D-512646118.html)

git是一个强大的版本控制软件，它有一个最大的特点是无论对一个文件做了什么改变，包括删除了都是可以恢复回来的。
有的时候，我们删除了一个文件后并不想让人有机会恢复它，或者我添加了一个很大的文件，删除后它还是占用着大量的空间。
要解决这个问题，也就是永久性地从git库中删除某个文件，包括日志等，方法如下：

```
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch file_name.txt" HEAD
rm -rf .git/refs/original/
git reflog expire --all
git gc --aggressive --prune
```

到这里，文件删除了，也再也恢复不了了。不过这时候有可能留下空的提交日志，清除办法如下：

```
git filter-branch --prune-empty
```

这样就完成了，再 force push 上去就完事了。

# 永久删除git所有储存的版本信息

[永久删除git所有储存的版本信息](http://huanghl97.blog.163.com/blog/static/59388860201188112347791/)

首先查看现有版本库的大小:

```
$ du -hs
  10M    .
```

删除步骤：

```
$ git filter-branch --tree-filter 'rm -f testme.txt' HEAD
    Rewrite bb383961a2d13e12d92be5f5e5d37491a90dee66 (2/2)
    Ref 'refs/heads/master'
    was rewritten
$ git ls-remote .
    230b8d53e2a6d5669165eed55579b64dccd78d11        HEAD
    230b8d53e2a6d5669165eed55579b64dccd78d11        refs/heads/master
    bb383961a2d13e12d92be5f5e5d37491a90dee66        refs/original/refs/heads/master
$ git update-ref -d refs/original/refs/heads/master bb383961a2d13e12d92be5f5e5d37491a90dee66
$ git ls-remote .
    230b8d53e2a6d5669165eed55579b64dccd78d11        HEAD
    230b8d53e2a6d5669165eed55579b64dccd78d11        refs/heads/master
$ rm -rf .git/logs
$ git reflog --all
$ git prune
$ git gc
```

再次查看版本库的大小:

```
$ du -hs
  108K    .
```

## [git彻底删除文件](http://www.tuliang.org/git-completely-remove-file/)

一般git删除文件使用`git rm`就可以了，但是有时候会产生一些文件你甚至不想让它出现在git的log中。[github](https://help.github.com/articles/remove-sensitive-data/)有一篇文章详细说明了如何操作，经过尝试是可以成功将文件彻底删除，在log中都找不到。

比如删除根目录下的file文件夹彻底删除：

```
git filter-branch --force --index-filter \
  'git rm -rf -r --cached --ignore-unmatch file' \
  --prune-empty --tag-name-filter cat -- --all
```

然后：

```
git push origin master --force
```

这样git仓库中这个文件夹就被彻底删除了。

如果你还想清除本地的一些缓存，可以依次进行下面几部操作：

```
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
```

亲测有效
