---
title: 使用私有git管理项目代码
date: 2017-02-26 20:21:13
tags: Github
---

某次电脑的神奇丢失数据后，我觉得用git来管理项目代码是很有必要的！但是实验室的项目代码直接扔到Github上面好像不是很合适，采用私有git就可以解决这一需求了。

<!--more-->

author: [@Huji][1]

*版权归作者所有，任何形式转载请联系作者。*


目前实验室的服务器上搭建了一个私有git采用的是`Gogs - Go Git Service`（其配置可以参考网络的文档[^Gogs]），访问地址是 `http://10.15.198.247:3000`。

我的注册邮箱是hujihchn@gmail.com
密码/用户名都是jessie

### 访问服务器并创建账号

在浏览器输入上面的访问地址，然后点击`register`，输入用户名，邮箱，密码进行注册。

![register][2]

注册好之后登录进去，在`explore`模块可以看到其他人最近更新的项目。

![explore][3]

### 学习Git的使用

Git可以从[这里][4]下载。具体的使用强烈推荐花一点时间把[廖雪峰的Git教程][5]看一遍。
看完之后，对下面这幅图[^git]能够理解的话，那么基本的入门已经完成了。
![git的核心操作][6]


### 常用操作总结

经过上面的学习，在这里再总结下一些最最常用的基础命令，省得自己老是忘记。

- **创建一个本地的git仓库**
```
$ mkdir learngit    # 新建一个项目文件夹，不新建也行
$ cd learngit       # 切换到项目文件夹
$ pwd               # 查看当前路径（可有可无的步骤...）
$ git init          # 初始化一个Git仓库
```

- **提交文件**

添加文件到Git仓库，分两步：

第一步，使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；

第二步，使用命令`git commit -m "本次提交的说明"`，完成。


- **提交到远程库**

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

- **管理修改**
```
git status              #查看工作区、暂存区的状态
git checkout -- <file>  #丢弃工作区上某个文件的修改
git reset HEAD <file>   #丢弃暂存区上某个文件的修改，重新放回工作区
```

[^Gogs]: https://blog.mynook.info/post/host-your-own-git-server-using-gogs

[^git]: http://wuchong.me/blog/2015/03/30/git-useful-skills/


  [1]: https://hujichn.github.io/
  [2]: http://7xsoqo.com1.z0.glb.clouddn.com/git/register.png
  [3]: http://7xsoqo.com1.z0.glb.clouddn.com/git/explore.png
  [4]: https://pan.baidu.com/s/1kU5OCOB#list/path=/pub/git
  [5]: http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
  [6]: http://ww4.sinaimg.cn/mw690/81b78497jw1eqnk1bkyaij20e40bpjsm.jpg