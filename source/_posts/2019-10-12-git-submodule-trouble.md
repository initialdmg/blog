---
title: Hexo 主题与 Git Submodule
date: 2019-10-12 00:14:17
tags:
- git
excerpt: 记录更换 Hexo 主题遇到的问题与解决方案，顺便学习一下 git submodule。
---

# 添加克隆的第三方仓库遇到的submodule问题

今天想把博客的默认theme给换了，于是fork了一个theme改成了自己的，然后克隆到项目目录下。在修改了theme后，git自动将克隆的theme文件夹识别为了submodule，我就直接提交更新了。修改了所有配置以后，我发现github pages里面反而啥也没有了。

一看仓库里面，主题的文件夹根本就不能点击，我就怀疑 ci 拉不到这个主题，我也没按文件的形式上传呀！于是乎想直接添加submodule，因为我的主题文件夹已经是完整的git项目了。

```shell
$ git submodule add git@github.com:initialdmg/hexo-theme-clover.git themes/clover

# 结果
'themes/clover' already exists in the index
```

这样一看是不行，应该是 clone或者commit 的时候默认添加了子模块。可是我的项目下边也没有`.gitmodules`呀……

```shell
$ git submodule
fatal: no submodule mapping found in .gitmodules for path 'themes/clover' # j就没生成 .gitmodule 文件
```

好嘛！那我就删掉重来，结果还是告诉我有问题，他找不到……

```shell
$ git rm -r themes/clover
fatal: could not lookup name for submodule 'themes/clover'
```

在搜索过后，找到了解决方案：

```shell
$ git rm -r themes/clover --cached
rm 'themes/clover'

$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    themes/clover

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        themes/clover/
```

接着先不提交，直接重新添加submodule:

```shell
$ git submodule add git@github.com:initialdmg/hexo-theme-clover.git themes/clover
Adding existing repo at 'themes/clover' to the index

$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitmodules
```

git 自动识别了这个 git 仓库，成功生成了 .gitmodules 文件。这说明添加成功了。将它提交，CI 报错了，克隆不下来。

```shell
$ git submodule update --init --recursive
Submodule 'themes/clover' (git@github.com:initialdmg/hexo-theme-clover.git) registered for path 'themes/clover'
Cloning into '/home/travis/build/initialdmg/blog/themes/clover'...
Warning: Permanently added the RSA host key for IP address '192.30.253.113' to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.
```

无法访问这个仓库，应该是没有给 Travis 权限导致的。[Travis CI 官方是默认支持 Git Submodules](https://docs.travis-ci.com/user/customizing-the-build/#git-submodules)，在拉取仓库时会默认拉取子模块的仓库，可以手动关闭该特性。

由于在使用 git submodule 时，添加了 git@github.com 的仓库地址，使用 SSH 的协议，所以拉取失败了，下面提供两个解决方案：

1. [Adding to SSH Known Hosts](https://docs.travis-ci.com/user/ssh-known-hosts/) - 官方提供的解决方案
2. 手动修改 .gitmodules 里配置的仓库地址或者update，将使用 git 协议的仓库链接改为 https 协议

将 url 修改为 https 后执行 `git submodule sync` 将gitconfig的url同步，然后提交就可以成功构建了。

## 克隆子模块

当我们在其他地方进行仓库的克隆时，发现 themes/clover 是个空文件夹，意味着没有安装该主题。那么如何 clone 一个完整的仓库呢？只需如下操作即可：

```shell
# 方式一：使用  submodule init / update
git clone git@github.com:initialdmg/hexo-theme-clover.git blog && cd blog
git submodule init && git submodule update

# 方式二：添加 --recursive 参数
git clone git@github.com:initialdmg/hexo-theme-clover.git --recursive
```

当该第三方主题更新了，我们可以更新子模块：

```shell
git submodule update --remote themes/clover
```

或者切换至 themes/clover 目录下使用 git 命令切换到不同的历史版本，如果对子模块执行了相关操作后，会提示 modified: themes/clover (new commits)：

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   themes/clover (new commits)
```

然后按正常流程提交并推送到远程仓库即可。

# 参考

- https://y0ngb1n.github.io/a/2270d168.html
- https://my.oschina.net/jerikc/blog/513039
- https://www.jianshu.com/p/9e5e95984c28
