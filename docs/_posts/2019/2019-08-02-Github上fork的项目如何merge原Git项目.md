---
layout: post
title: Github上fork的项目如何merge原Git项目
date: 2019-08-02 17:11:00
author: 薛师兄
tags: Github
---
## 问题场景

小明在Github上`fork`了一个大佬的项目，并`clone`到本地开发一段时间，再提交`merge request`到原Git项目，过了段时间，原作者联系小明，扔给他下面这幅截图并告知合并处理冲突，让他自行解决。

![](./20190802Github上fork的项目如何merge原Git项目/40953964.png)

小明看到后犯难了，我在IDEA上`merge`的时候，也没看到原Git项目分支的选项呀，只能`merge` `fork`后项目的其它分支，这该如何是好？

## 问题解决

解决方法很简单，只需要把当前Git仓库关联一下原远程Git仓库就行了。

操作步骤：

在本地Git仓库目录执行下面命令，查看当前关联的远程库：


```shell
$ git remote -v
origin  git@github.com:xiaoming/wechatpay.git (fetch)
origin  git@github.com:xiaoming/wechatpay.git (push)
```

然后开始执行关联远程仓库的命令：

```shell
$ git remote add upstream 原始项目仓库的git地址
```

再次通过`git remote -v`就可以查看到最新信息了！

小明再去打开心爱的IDEA，在merge的时候果然发现多了原Git仓库master分支选项，问题解决～

![](./20190802Github上fork的项目如何merge原Git项目/54599842.png)