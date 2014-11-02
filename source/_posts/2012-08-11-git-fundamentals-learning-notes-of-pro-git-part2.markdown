---
layout: post
title: "Git Fundamentals - Branch: Learning Notes of Pro Git"
date: 2012-08-11 16:15
comments: true
categories: [Git, Github, Notes]
---

##分支(BRANCH)    

{% pullquote %}  
理解分支之前再来回顾一下前面所讲到的git与其他版本控制系统的区别**“Git 保存的不是文件差异或者变化量,而只是一系列文件快照”**。就是这个差别使得git分支的实现要比其他版本控制系统要轻松的多，git的分支是_“难以置信的轻量级”_，分支操作几乎在瞬间完成，而不像其他版本控制系统需要创建一个完整的源代码目录副本，对于大型项目来说需要耗费大量的项目时间。{"git中创建分支的操作就像移动指针那么简单，因为git分支的本质是指向commit对象的可变指针。"}        
{% endpullquote %}

下面我们介绍git分支的一般操作：  

###git branch \<branch-name\>
从当前提交点创建一个新的分支  
git branch myBranch  


###git checkout \<branch-name\>
切换当前工作目录到某一分支，当你切换到分支时，git的HEAD指针就指向了你切换到的分支，这样里就能够在另外的分支里工作了，这里HEAD就是标志当前工作分支的一个指针。  
git checkout myBranch

有一个快捷的方式创建一个新分支并切换到该分支  
git checkout -b myBranch  
这条命令就相当于执行了上面两条命令。  

讲完基本的创建分支和检出分支操作之后，将介绍合并(merge)操作。  

###git中合并有两种方式：快进合并和三方合并

1 快进合并(Fast Forward): 当你想要并入(merge in)的分支是并进(merge into)分支的直接下游时，git采用快进的合并方式，如：  
C0 \<\-- C1 \<\-- C2 \<\-- C3 \<\-- C4   
                 |               |        
               master           myBranch   
当我们打算将myBranch(merge in branch)并入master(merge into branch)分支时，由于myBranch是master的直接下游，git只需将指针直接右移到C4，这就相当于快进了。  
具体操作如下：  
	git checkout master #切换到master分区
	git meger myBranch #合并myBranch分区到master分区

2 三方合并(Recursive Merge): 当并入分区和并进分区不再是直接的上下游时，即出现了分叉，git将找出两者的共同祖先提交使用三方合并。

###合并冲突  

合并产生冲突时，git会提示产生冲突的文件，只有冲突解决之后，merge才能完成，实际上是用户手动处理冲突之后提交冲突文件。
<!-- more -->
##分支的管理

###git branch

不加任何参数，列出当前所有分支

###git branch -v

加参数 -v 列出所有分支，并且显示每一个分支最后一次commit的信息

###git branch --merge

查看哪些分支已经被并入当前分支

###git branch --no-merge

查看哪些分支没有并入当前分支

###git branch -d \<branch-name\>

删除分支,但是当分支包含未被并入的工作时删除分支将导致失败，不过如果你坚信要删除该分支可以使用:  
git branch -D \<branch-name\>

##远程分支(Remote Branch)

远程分支(remote branch)是对远程仓库状态的索引。它们是一些无法移动的本地分支;只有在进行 Git的网络活动时才会更新。远程分支就像是书签,提醒着你上次连接远程仓库时上面各分支的位置。   
远程分支使用**\<remote-repos-name\>/\<remote-branch-name\>**来表示，例如常见的远程分支：**origin/master**  
如果你没有和网络中远程分支进行通讯，本地的远程分支始终指向上一次通讯时的位置，并不能得到更新。进一步，当你的同事更新了远程分支，服务器中的远程分支将是你同事的版本，而你本地的远程分支并没有得到更新！当你需要推送(将在后面讲到)你的工作到服务器时，注意这是服务器中的版本已经发生了改变，你必须先拉取远程分支中的更新到你的本地分支，再进行推送操作。拉取操作我们前面讲过是git fetch \<remote-name\>。  

###推送(push)  

要想和其他人分享某个分支,你需要把它推送到一个你拥有写权限的远程仓库。  
git push \<remote-repos-name\> \<branch-name\>  
如果你有个叫funstaff的分支需要和他人一起开发,可以运行：  
git push origin funstaff  


这样当你的同伴想要合并你的funstaff分支到当前分支的话，可以使用：  
git merge origin/funstaff
或者新建一个分支，独立继续开发：   
git checkout -b myFunStaff origin/funstaff

###跟踪分支
从远程分支检出的本地分支,称为跟踪分支(tracking branch)。在跟踪分支里输入这些分支里运行git push, Git会自行推断应该向哪个服务器的哪个分支推送数据。
上面的列子我们可以使用下面的命令：
git checkout --track origin/funstaff

##衍合(rebase)

git checkout myBranch  
git rebase master  
*rebase命令,可以把在一个分支里提交的改变在另一个分支里重放一遍。*  
衍合的金科玉律：

{% blockquote %}  
永远不要衍合那些已经推送到公共仓库的更新。  
{% endblockquote %}


##References
[Pro Git 简体中文版](http://git-scm.com/2010/06/09/pro-git-zh.html)  

[Pro Git Enlish Edition](http://git-scm.com/book)


