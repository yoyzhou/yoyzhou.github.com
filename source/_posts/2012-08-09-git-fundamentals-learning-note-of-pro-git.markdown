---
layout: post
title: "Git Fundamentals: Learning Note of Pro Git"
date: 2012-08-09 22:30
comments: true
categories: [Git, Github, Notes]
---

##git基础

###git优点
	1 直接快照,而非比较差异
	不像集中式版本管理系统那样只记录不同版本之间的Deta, 而是直接快照整个项目文件，这使得Git更像是一个文件系统

	2 近乎所有操作都可本地执行

	3 时刻保持数据完整性
	通过数据内容校验和计算(checksum)

	4 多数操作仅添加数据

###git配置文件的类型和位置

	/etc/gitconfig 系统级别的git的配置文件 通过 git config --system 选项指定进行配置


	～/.gitconfig 用户级别的配置文件 通过 git config --global 选项指定进行配置


	.git/config(当前工作项目目录下) 项目级别的配置文件

##git命令


###git init 
初始化git仓库

###git status 
查看git repos库状态  
status的形式:未跟踪，未修改，修改，暂存  
运行了git add之后又作了修订的文件,需要重新运行git add把最新版本重新暂存起来。


####演示git仓库中文件状态的例子
	1.vim <file> -- untracked v0  
	2.git add <file> -- tracked & staged v0  
	3.git commit -m '' -- version v0  
	4.vim <file> -- tracked & modified & staged v0  
	5.git add <file> -- tracked & staged v1  
	6.vim <file> -- tracked & modified & staged v1  
	7.git commit -m '' -- version v1  


###git diff 
--查看尚未暂存文件更新了什么，即查看工作目录中当前文件与暂存区快照之间的差异。用上面的例子来说，假设我们在第4步后面执行git diff, 这个命令查看的将是文件\<file\>在第4步[修改过但是还未暂存]与第3步[Staged v0]的差异  

###git diff --cached/--staged
查看已经暂存起来的文件和上次提交时的快照之间的差异。还是用上面的例子，假设我们在第7步后面执行命令git diff --staged，将查看文件\<file\>在第5步[Staged v1]和第3步[Version v0]之间的变化

###git commit -m 'message for commit...'
在之进行commit之前,一定要确认还有什么修改过的或新建的文件还没有git add过,否则提交的时候不会记录这些还没暂存起来的变化。所以,每次准备提交前,先用下,是不是都已暂存起来了,然后再运行提交命令git status看是不是都已经暂存起来了，然后再执行命令git commit。

###git rm \<file\> 
Remove file(s) from git's tracking list, meanwhile delete the file(s)

###git rm --cached \<file\> 
Remove file(s) from git's tracking list, but keep file(s) as cached, file(s) will not be removed.

###git mv file_from file_to 
Move file in git repos, equals to:

	1 mv file_from file_to  
	2 git rm file_from  
	3 git add file_to  

###git log [-options] 
View commit history  
-p 展开每一次提交的diffs  
-n 仅显示最近n次提交  
--stat 显示每次提交的文件修改统计信息  
--pretty 指定显示格式，oneline、short、full、fuller和format(后面指定格式)  
--since 从什么时间开始  
etc. 更多选项请使用 git log --help

###git commit --amend
进行重新提交操作，可修改上一次提交的commit comment或者添加忘记staged的更新

###git reset HEAD \<file\>
撤销对文件\<file\>的暂存操作，比如你需要对某一个文件进行单独提交时，可以：  
	git add . #将所有的文件暂存  
	git reset HEAD \<file\> #撤销对某文件的暂存  
	git commit -m 'commit message'  
	git add \<file\>  
	[可能的文件\<file\>的修改]  
	git commit \<file\> -m 'commit \<file\> seperately'  

###git checkout --\<file\>
取消对文件的修改，慎用。此命令将文件恢复到文件修改前的版本/暂存区内容，工作目录中所有对该文件的修改都将丢失。  
在git 1.7.9.5中测试可省略“--”直接使用git checkout \<file\>
 
##git远程仓库

###git remote -v
查看远程库信息 -v选项显示远程库地址  
<!-- MORE -->

###git remote add [\<options\>] \<short name\> \<git url\>     

添加一个远程库, \<short name\> 可以为远程库选一个简短的名字; \<git url\>远程库的git项目URL, e.g.   
	git remote add pb git://github.com/paulboone/ticgit.git    

###git fetch [remote-name]
从远程仓库抓取数据，e.g.:
	 git fetch pb #命令会到远程仓库pb中拉取所有你本地仓库中还没有的数据。  
注：当你使用 git clone命令克隆一个远程库时，默认会创建一个origin的remote repository，当使用 git fetch origin 实际上是拉取从你上一次clone以来别人传到此远程仓库的所有更新  
NOTE：fetch 命令只是将远端的数据拉到本地仓库,并不自动合并到当前工作分支,只有当你确实准备好了,才能手工合并。  

###git push [remote-name] [branch-name]
推送数据到远程仓库  
[remote-name] 为远程仓库名称，origin或者你使用git remote add 命令添加的远程仓库  
[branch-name] 本地分支名称, e.g.   
	git push origin master #将本地的master分支推送到origin服务器上  

###git remote show \<remote-name\>
查看远程仓库的信息, e.g.  
	git remote show origin  

###git remote rename \<old-remote-name\> \<new-remote-name\>
重命名远程仓库名称

###git remote rm \<remote-name\>
删除远程仓库

##git标签  

###git tag
显示当前项目中的所有标签

###git tab -l '\<expression\>' 
显示符合检索表达式的标签，e.g.
git tag -l 'v1.0.*'

###git tag -a \<tag-name\> -m 'tag message'
新建一个含附注的(annotated)标签, e.g.  
git tag -a v1.0.1 -m 'version 1.0.1'  

###git tag -a \<tag-name\> \<checksum\>
为某一次提交补上标签，提交的检验和可以使用前面的git log进行查看

###git show \<tag-name\>
显示标签信息  
git show v1.0.1

###git push [remote-name] \<tag-name\>
推送/分享某一个标签版本，而不是本地分支  
git push origin v1.0.1  


##git 使用技巧    

###添加tab自动补全功能     
	1 clone git源码库  
		git clone git://github.com/git/git.git  
	2 cp contrib/completion/git-completion.bash ~/.git-completion.bash  
	3 source ~/.git-completion.bash #添加.git-completion.bash到你的.bashrc文件中。

如果你不想克隆git源码库可以直接从git源码库中下载单个[git-completion.bash](https://raw.github.com/git/git/master/contrib/completion/git-completion.bash)文件,再进行复制和source操作。
  

###添加git命令别名
e.g.

	git config --global alias.co checkout 
	git config --global alias.st status 
etc.


##References
[Pro Git 简体中文版](http://git-scm.com/2010/06/09/pro-git-zh.html)  

[Pro Git Enlish Edition](http://git-scm.com/book)

\<To be continued...\>












