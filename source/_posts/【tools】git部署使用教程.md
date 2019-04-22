---
title: 【tools】git部署使用教程
date: 2018-10-29 14:34:12
tags: 
    - git	
---

### 配置Git

1. 创建本地ssh key。

```shell
ssh-keygen -t rsa -C "your_email@youremail.com"
#后面的your_email@youremail.com改为你在github上注册的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。
```

2. 复制`~/.ssh/id_rsa.pub`里面的`key`。
3. 回到github上，进入 Account Settings（账户配置），左边选择SSH Keys，Add SSH Key,title随便填，粘贴在你电脑上生成的key。
4. 验证是否成功，在git bash下输入：

```shell
ssh -T git@github.com
#如果是第一次的会提示是否continue，输入yes就会看到：You've successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。 
```

5. 设置username和email，因为github每次commit都会记录他们。

```shell
git config --global user.name "your name"
git config --global user.email "your_email@youremail.com"
```

6. 进入要上传的仓库，右键git bash，添加远程地址：

```shell
git remote add origin git@github.com:yourName/yourRepo.git
#后面的yourName和yourRepo表示你再github的用户名和刚才新建的仓库，加完之后进入.git，打开config，这里会多出一个remote "origin"内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。 
```

7. 创建新文件夹，打开，然后执行` git init`以创建新的 git 仓库。

### 检出仓库

执行如下命令以创建一个本地仓库的克隆版本：

```shell
git clone /path/to/repository 
```

 如果是远端服务器上的仓库，你的命令会是这个样子： 

```shell
git clone username@host:/path/to/repository
```

### 工作流

你的本地仓库由 git 维护的三棵"树"组成。第一个是你的 `工作目录`，它持有实际文件；第二个是 `暂存区（Index）`，它像个缓存区域，临时保存你的改动；最后是 `HEAD`，它指向你最后一次提交的结果。         

你可以提出更改（把它们添加到暂存区），使用如下命令：

```shell
git add <filename>
git add *
```

这是 git 基本工作流程的第一步；使用如下命令以实际提交改动：

```shell
git commit -m "代码提交信息"
```

现在，你的改动已经提交到了 **HEAD**，但是还没到你的远端仓库。         ​      

### 推送改动

你的改动现在已经在本地仓库的 **HEAD** 中了。执行如下命令以将这些改动提交到远端仓库：

```shell
git push origin master
```

可以把 *master* 换成你想要推送的任何分支。

可以把 *master* 换成你想要推送的任何分支。

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：

```shell
git remote add origin <server>
```

 如此你就能够将你的改动推送到所添加的服务器上去了。 

 如此你就能够将你的改动推送到所添加的服务器上去了。 

### 分支

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，*master* 是"默认的"分支。在其他分支上进行开发，完成后再将它们合并到主分支上。 		

创建一个叫做"feature_x"的分支，并切换过去：

```shell
git checkout -b feature_x
```

切换回主分支：

```shell
git checkout master
```

再把新建的分支删掉：

```shell
git branch -d feature_x
```

除非你将分支推送到远端仓库，不然该分支就是 *不为他人所见的*：

```shell
git push origin <branch>
```