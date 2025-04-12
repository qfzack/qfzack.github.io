---
title: "ubuntu下git的使用"

date: 2020-10-11T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Linux", "Git"]
author: "Qingfeng Zhang"
---

  * 最近在看廖雪峰老师的git教程，感觉讲的很好，但是容易忘记，而且命令也比较多，所以就在这整理一下笔记；
  * git是目前最先进的分布式版本控制系统，是linux之父linus开发的，用于管理linux内核开发，与分布式版本控制系统相对的是集中式版本控制系统；

## 1.git的安装

  * 先安装git：

```shell
apt-get install git  
```
  
  * 安装完成后还需要设置用户名和邮箱：

```shell
git config --global user.name "Your Name"  
git config --global user.email "email@example.com"  
```
  
## 2.git版本库

  * 进入一个文件夹，将其设置为git可管理的仓库：

```shell
git init  
```

此时，文件夹里会多了一个.git目录，默认隐藏；

  * 当增加或修改了一个文件`file.txt`，将其**从工作区添加到暂存区** ：

```shell
git add file.txt  
```

然后将文件**从暂存区提交到当前分支** ：

```shell
git commit -m 'creat a new txt file'  
```

`-m`表示添加说明，最好加上；

  * 查看当前仓库的状态：`git status`
  * 查看文件修改的部分：`git diff file.txt`

## 3.git文件操作

  * 查看**文件历史记录** ：

```shell
git log  
```
  
或是`git log --pretty=oneline`和`git log --oneline --graph`简洁查看；

  * 将文件**退回到上一版本** ：

```shell
git reset --hard HEAD^  
```

使用`HEAD^^`退回到上上个版本；  
或是根据文件历史记录的`commit id`**退回到指定版本** ：

```shell
git reset --hard 1904a  
```
  
文件的版本是由HEAD指向决定的；

  * 查看每一次的命令：`git reflog`

**撤销文件的修改** ：

  * 如果文件只是在工作区进行了修改，没有添加到暂存区：`git checkout -- file.txt`
  * 如果文件已经从工作区提交到暂存区：先用`git reset HEAD <file.txt>`，再用`git checkout -- file.txt`

**删除仓库中的文件** ：  
文件已经添加到了暂存区：

  * 在用`rm file.txt`删除文件后，还需要使用`git rm file.txt`
  * 如果使用`rm file.txt`误删了文件，通过`git checkout -- file.txt`恢复

## 4.git远程仓库

github就是一个提供git仓库托管服务的服务器网站，本地git仓库与github之间的传输时通过SSH加密的，再第一次使用的时候需要创建**远程连接**
，先**创建SSH key** ：

```shell
ssh-keygen -t rsa -C "youremail@example.com"  
```

密码可以不输入，运行后会在主目录中的.ssh目录里由id_rsa和id_rsa.pub两个文件，分别是私钥和公钥；  
然后，在**github网站页面** 的ssh keys中点击Add SSH
key，将id_rsa.pub的内容复制到文本框内，点击添加；github允许添加多个ssh key；

  * **提交到github上的远程仓库** ： 

在SSH连接github后，如果想让远程仓库与本地仓库同步，先在github页面创建了新的repository，然后与本地repository进行关联：

```shell
git remote add origin git@github.com:qfzack/my_repository.git  
```

origin是远程仓库的默认名称，可以修改，然后在本地进行第一次推送：  
按照上文的git操作，先初始化本地仓库，然后添加所有文件，再进行提交，最后进行推送：

```shell
git push -u origin master  
```
  
-u表示建立关联，master表示当前分支，之后的推送可以简化为：
    
```shell
git push origin master  
```
  
  * **克隆远程仓库到本地仓库** ： 

使用仓库的地址进行克隆：

```shell
git clone git@github.com:michaelliao/gitskills.git  
```
  
## 5.后续

后面的教程还包括分支管理和标签管理，是为了便于多人协同工作以及项目的管理，目前还用不上，就只是看了一遍留个印象，估计不久就忘记了。

## 6.使用过程中遇到的问题

  * **问题一：**

在尝试将服务器中的项目提交到github中，按照操作在push的时候出现了以下的错误：

```shell
error: failed to push some refs to 'git@github.com:qfzack/YOLOv3-pytorch-PCB.git'  
hint: Updates were rejected because the remote contains work that you do  
hint: not have locally. This is usually caused by another repository pushing  
hint: to the same ref. You may want to first integrate the remote changes  
hint: (e.g., 'git pull ...') before pushing again.  
hint: See the 'Note about fast-forwards' in 'git push --help' for details.  
```
  
这是因为在github创建repository的时候建立了README.md文件，而在本地没有这个文件，因此报错；  
这种情况在实际的分工协作中也会出现，如果在我提交之前已经有另外一个人提交了，那么我的master分支就落后与远端，就需要先与远端同步再push，因此使用命令：

```shell    
git pull --rebase origin master  
```

使得远端与本地同步，也就是本地的文件夹中多了README.md文件，然后再提交

  * **问题二：**

在push的过程中又遇到了新的问题：

```shell
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.  
remote: error: Trace: 1035a7622c0700369401b4d1756d93cd  
remote: error: See http://git.io/iEPt8g for more information.  
remote: error: File weights/darknet53.conv.74 is 154.96 MB; this exceeds GitHub's file size limit of 100.00 MB  
```
  
是因为原文件夹里包含大文件，在提交的时候就不要提交这些文件了,但是删除之后再次push还是显示有之前的大文件，因为之前的提交没有撤回，先查看提交：

```shell
git log  
```
  
然后执行：

```shell
git reset --soft ID  
```

然后再次添加、提交后，push成功

  * **笔记一：**

针对上面遇到的问题，查了一些相关的资料，发现一个需要注意的点：如果想要把本地的改动更新到github服务器，必须要使用git进行操作，即如果想在本地删除文件，要用`git
rm`不能直接用`rm`；  
还有一个是`git add .`和`git add -u`和`git add -A`的区别  
一开始我都是用`git add .`表示添加所有文件，但是在删除一些文件后用`git add .`再提交并push就不行了，因为`git add
.`会将工作时的变化提交到暂存区，包括**文件的修改(modified)和新文件(new)** ，但就是**不包括已删除的文件** ，总结一下就是：

> `git add .`提交所有修改和新建的数据到暂存区；  
> `git add -u`提交修改和被删除的文件到暂存区，其中-u等于-update；  
> `git add -A`提交所有被删除、替换、修改、新建的文件到暂存区，其中-A等于-all；

  * **笔记二：**

当有多个commit的信息重复时，可以对其进行合并：  
先使用`git log --oneline`查看提交记录，然后合并前10条提交：  
`git rebase -i HEAD~10`  
然后会出现vim编辑界面，将2到10条(第一条是最早的提交)提交的`pick`修改为`squash`(可以用`s`),表示与之前的提交合并，保存并退出会继续对合并信息进行编辑，编辑并保存即可
