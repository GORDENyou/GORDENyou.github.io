---
title: "git之旅"
date: 2018-06-14T20:22:42+08:00
categories: ["技术之美"]
tags: ["git"]
draft: false
---

 让我们带着问题阅读文章：

- 我们为什么需要`git`？
- `git`的本地结构与远程库的交互方式
- `git`中版本控制术语
- 

## 为什么需要`git`？

还记得那年我们写的毕业论文吗？

每次拿着刚打印好的毕业论爬上楼给你的导师修改，下了楼你的论文变成了草稿纸，而且这样的事情还不止一两次。

![](../index.assets/git_paper.png)

<center>毕业也不是个容易的事</center>
更别说给你提需求的甲方或者老板了，各种方案和修改满天飞，说不定到了`deadline`， 老板一句：“好像还是第一版最好。”，这时候你就知道了版本控制的好了。

#### 方案一： 本地版本控制

![](../index.assets/git_version1.png)

<center>好像和我们本地复制文件没什么区别</center>
这个版本解决的问题还是有限的，比如如何协同他们一起处理文件？

#### 方案二：`SVN`

这时候我们鼎鼎大名的 `SVN` 就出场了，`SVN`属于集中式版本管理，初步解决了协同开发。

![](../index.assets/git_version2.png)

但是由于开发版本集中在一台计算机保管处理，这时候问题又来了：假如这台计算机挂了咋办？主角`git`要登场了！

#### 方案三：`git`

都已经8102年了，假如你还不知道`git`，那……你现在就跟着我了解下吧。

先上图：

![](../index.assets/git_version3.png)

为了解决上面所述的问题，`git`采用了分布式的版本控制管理，除了服务器有所有版本控制软件外，每台协同开发的计算机也拥有其版本备份。

## `git`的本地结构与远程库的交互方式

![](../index.assets/git_struct.png)

<center>本地结构</center>

![](../index.assets/git_connect.png)

<center>本地库与远程库的交互</center>



## `git`中版本控制术语

在这里我仅仅列出我是用的较少的术语：

#### 添加`tag`标签

```bash
git tag -a v1.0 //tag可以为我们的提交打上标签，比如软件版本的迭代。
git tag  // 列出所有标签

在git版本 2.13 之后，我们就可以使用 git log 输出所有的 commit 和 tag 了。
git tag -d v1.0 //删除 v1.0 这个标签

如果我们想向很久以前的 commit 添加标签：
git tag -a v1.0 a79070 // 我们只需提供 SHA 即可。
```



#### 分支

```bash
git branch // 列出所有分支
git branch test // 创建 test 分支

我们虽然创建了test分支，但是我们还没有切换到test分支。所以我们提交的 commit 还是会保存到 master 分支中
git checkout test // 分支切换

git branch -d test //　删除分支，假如要强制删除需要大写Ｄ
```

