---
title: Git使用
tag: git
date: 2024-07-19
categories: Git
index_img: https://s2.loli.net/2024/07/31/mBz1ZMHwft8nRUc.jpg
---
# Git使用

## git基本了解

### 学习资料

[超详细的Git使用教程(图文)-CSDN博客](https://blog.csdn.net/qq_37883866/article/details/105349257)

[Git基本使用教程（一）：入门及第一次基本完整操作_git的使用-CSDN博客](https://blog.csdn.net/qq_35206244/article/details/97698815)

[git--一文弄懂git的工作区、索引区、本地仓库、远程仓库以及add、commit、push三个操作 - at_today - 博客园 (cnblogs.com)](https://www.cnblogs.com/Jing-Wang/p/10991008.html)

[【原创】Git删除暂存区或版本库中的文件 - cposture - 博客园 (cnblogs.com)](https://www.cnblogs.com/cposture/p/git.html)

[Git如何从暂存区（index/cache）中移除文件|极客教程 (geek-docs.com)](https://geek-docs.com/git/git-questions/611_git_how_to_remove_a_file_from_the_staging_area_index_cache_in_git.html)

[git status 状态命令——查看文件状态_git status 看非新增文件-CSDN博客](https://blog.csdn.net/weixin_44567318/article/details/119701438)

[Git 学习（三）本地仓库操作——git add & commit - feesland - 博客园 (cnblogs.com)](https://www.cnblogs.com/feeland/p/4500721.html)

[git branch的详细使用，10个常见用法_git branch -vv-CSDN博客](https://blog.csdn.net/chaogu94/article/details/111057918)

[Git基础 - git tag 一文真正的搞懂git标签的使用-CSDN博客](https://blog.csdn.net/qq_39505245/article/details/124705850)

### git下载

**官网：** https://git-scm.com/downloads

## git使用方法

#### 注册GitHub账号

账号：1

密码：1

#### git注册

```
	//设置账号，如果去掉 --global 参数只对当前仓库有效。
	$ git config --global user.name "zhengyiyang"
	
	//设置邮箱，如果去掉 --global 参数只对当前仓库有效。
	$ git config --global user.email "903372205@qq.com"
	
	//设置SSH，如果去掉 --global 参数只对当前仓库有效。
	$ ssh-keygen -t rsa -C "903372205@qq.com"
	
	储存位置：/c/Users/114008/.ssh/id_rsa
	
	id_rsa.pub为公共密钥
	id_rsa为私人密钥
```

#### github配置SSH

**打开id_rsa.pub文件，全选，复制全文**


```
	ssh-rsa AAAAB3NzaC1yc2EA...
```

**github->账户->setting**

Title：1

Key type：Authentication Key

#### 测试是否成功连接

```
	$ ssh -T git@github.com
	//代表成功连接
	Hi yangyangdeyi0119! You've successfully authenticated, but GitHub does not provide shell access.  
```

#### 建立Github云端仓库

[github仓库建立及配置教程新手教程_github创建仓库-CSDN博客](https://blog.csdn.net/qq_44722674/article/details/117200397)

修改用户名

[GitHub修改昵称和用户名（图解详细教程）_github改名-CSDN博客](https://blog.csdn.net/weixin_44285445/article/details/107833418)

#### 创建本地仓库

```
	//首先需要新建一个文件夹作为本地仓库
	//初始化该文件为本地仓库
	$ git init  
	
	//下载下来的test文件夹也是本地仓库
	$ git clone https://github.com/yangyangdeyi0119/test.git  
```

#### 将文件加入暂存区

文件本身在工作区，需要通过文件锁定，将其加入暂存区

```
	//可以同时添加多个文件夹
	$ git add /test /test1
	
	//添加单个文件
	$ git add README.md 
    
    //将修改操作的文件和未跟踪新添加的文件添加到git系统的暂存区，注意不包括删除  
	$ git add .   
	
	//将文件包的所有文件加入暂存区
	$ git add -f .
    
    //将已跟踪文件中的修改和删除的文件添加到暂存区，不包括新增加的文件，注意这些被删除的文件被加入到暂存区再被提交并推送到服务器的版本库之后这个文件就会从git系统中消失了。 
	$ git add -u 
	
	//表示将所有的已跟踪的文件的修改与删除和新增的未跟踪的文件都添加到暂存区。
	$ git add -A
	
	//暂存区各类状态
	- untracked 未跟踪（未被纳入版本控制）
	- tracked 已跟踪（被纳入版本控制）
	- Unmodified 未修改状态
	- Modified 已修改状态
	- Staged 已暂存状态
```

#### 将文件移除暂存区

```
	//仅删除暂存区的文件，不影响工作区的文件
	$ git rm --cache <file/aaa>
	
	//删除暂存区和工作区的文件
	$ git rm -f <file/aaa>
	
	//撤销对暂存区的修改。这个命令主要用于丢弃或还原文件的更改(测试未成功)
	$ git restore --staged <file/aaa>
	
	//将分支回退到之前的提交，并且还可以选择是否保留暂存区的更改
	$ git reset
	
	//撤销对工作区修改；这个命令是以最新的存储时间节点（add和commit）为参照，覆盖工作区对应文件file；这个命令改变的是工作区
	$ git checkout 文件名
	
```

#### 查看文件状态

```
	//获取文件状态-完整
	$ git status
	
	//获取文件状态-简洁
	$ git status -s 更加简洁
		' ' （空格）表示文件未发生更改
		M 表示文件发生改动。
		A 表示新增文件。
		D 表示删除文件。
		R 表示重命名。
		C 表示复制。
		U 表示更新但未合并。
		? 表示未跟踪文件。
		! 表示忽略文件。
	
	//显示分支和跟踪信息 --branch
	$ git status -s -b
	
	//显示变更的文本内容，在不使用 -s 选项时才会显示变更内容
	//只有一个 -v 选项时，显示版本库和暂存区之间比较发生变更的内容。
	$ git status -v
	//而有两个 -v 选项时，显示暂存区和工作区之间比较发生变更的内容。
	$ git status -v -v
	
	//显示未跟踪文件
	$ git status -s -u[<mode>]
		no —— 不显示未跟踪的文件
		normal —— 显示未跟踪的文件和目录。
		all —— 还显示了未跟踪目录下的文件
		
	//用来查看暂存区中文件信息
	$ git ls-files -参数
		--cached(-c)显示暂存区中的文件，git ls-files命令默认的参数
		--deleted(-d)显示删除的文件
		--modified(-m) 显示修改过的文件
		--other(-o)显示没有被git跟踪的文件
		--stage(-s) 显示mode以及文件对应的Blob对象，进而我们可以获取暂存区中对应文件里面的内容。
```

#### 文件加入分支

提交更改，实际上就是把暂存区的所有内容提交到当前分支，需要提交的文件修改通通放到暂存区；然后，一次性提交暂存区的所有修改

```
	// 把暂存区的所有修改提交到分支，须输入描述信息
	$ git commit -m "描述信息"
	
	//更改之前一次commit的描述信息
	$ git commit --amend
	
	//提交暂存区的指定文件到仓库区（不行，最好单个提交git add，然后在git commit）
	$ git commit <file1> <file2> ... -m "message"
	
	//-a 参数设置修改文件后不需要执行 git add 命令，直接来提交（不好用）
	$ git commit -a
	
	出现报错"nothing to commit, working tree clean"
    只需要修改该文件夹下的任意一个文件，因为检测到版本未发生改变
    
    //查找推送版本号
    $ git log 
    
    //选择回退版本，回退后，版本之后的将会丢失
    git reset --hard <目标版本号>
    
    //软回退，不修改代码，回到暂存区
    git reset --soft HEAD~n
    
    //只显示一行信息
    git log --oneline
```

#### 分支管理

```
	//创建分支命令
	$ git branch <branchname>
	
	//切换分支命令
	$ git checkout <branchname>
	
	//列出分支
	git branch
	//查看本地分支+上次提交的信息
	$ git branch -v
	//查看本地分支+远程分支
	$ git branch -a
		- 红色代表云端仓库分支
		- 白色代表本地仓库分支
		- 绿色代表目前所在分支
	//查看本地分支+上次提交的信息+本地和远程分支的关系
	$ git branch -vv
	//查看本地分支+上次提交的信息+本地和远程分支的关系+远程分支
	$ git branch -vv -a
	//只查看远程分支
	$ git branch -r
	
	//创建新分支并立即切换到该分支下
	$ git checkout -b <branchname>
	
	//删除本地分支
	$ git branch -d <branchname>
	//强制删除分支
	$ git branch -D aaa
	
	//合并分支
	$ git merge <branchname>
	
	//删除远程分支
	$ git push <主机名> -d <分支名>
	
	//将本地分支推送到远程分支，如果远程分支不存在，则创建。
	$ git push <远程主机名> <本地分支名>:<远程分支名>
	$ git push --set-upstream origin dev
	
	//跟踪远程分支
	git checkout -b zhanghanlun origin/zhanghanlun
	//用于解决远程 HEAD 指向一个不存在的引用，无法检出。
```

#### 将文件推送到云端仓库

```
	//第一次推送代码指令
	$ git push -u origin <branchname>
	//-u参数可以在推送的同时,将origin仓库的master分支设置为本地仓库当前分支的upstream(上游)。添加了这个参数,将来运行git pull命令从远程仓库获取内容时,本地仓库的的这个分支就可以直接从origin的master分支获取内容,省去了另外添加参数的麻烦。
	
	//之后推送
	$ git push origin master
	
	//不同分支之间推送
	$ git push -u origin <branchname1>:origin/<branchname2>
	
	//云端仓库分支更新到本地仓库
	$ git remote update origin
		后缀加上 --prune则可以与云端仓库分支一致，多余的会被删除
		
	//报错
	error: failed to push some refs to 'https://github.com/yangyangdeyi0119/Learning.git'
	hint: Updates were rejected because the remote contains work that you do not
	hint: have locally. This is usually caused by another repository pushing to
	hint: the same ref. If you want to integrate the remote changes, use
	hint: 'git pull' before pushing again.
	hint: See the 'Note about fast-forwards' in 'git push --help' for details.
	直接git pull之后就可以了
	
	//删除现有远程仓库
	$ git remote rm origin
	
	//添加新远程仓库
	$ git remote add origin url
	
	//查看远程仓库的地址
	$ git remote -v
	
	//更换远程仓库地址，URL为新地址
	$ git remote set-url origin URL
```

#### 暂存空间使用

stash是本地的，不会通过git push命令上传到git server上

发现有一个类是多余的，想删掉它又担心以后需要查看它的代码，想保存它但又不想增加一个脏的提交。这时就可以考虑git stash。

使用git的时候，我们往往使用分支（branch）解决任务切换问题，例如，我们往往会建一个自己的分支去修改和调试代码, 如果别人或者自己发现原有的分支上有个不得不修改的bug，我们往往会把完成一半的代码commit提交到本地仓库，然后切换分支去修改bug，改好之后再切换回来。这样的话往往log上会有大量不必要的记录。其实如果我们不想提交完成一半或者不完善的代码，但是却不得不去修改一个紧急Bug，那么使用git stash就可以将你当前未提交到本地（和服务器）的代码推入到Git的栈中，这时候你的工作区间和上一次提交的内容是完全一样的，所以你可以放心的修Bug，等到修完Bug，提交到服务器上后，再使用git stash apply将以前一半的工作应用回来。

经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。解决这个问题的办法就是git stash命令。储藏(stash)可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

```
	//将未提交的修改保存至堆栈中
	$ git stash
	
	//为此次stash添加说明信息，便于以后查看
	$ git stash save "stash message info"  
	
	//查看stash栈中的内容
	$ git stash list
	
	//将stash中的内容弹出，并应用到当前分支对应的工作目录上，该命令将堆栈中最近保存的内容删除（出栈操作）
	$ git stash pop
	
	//将指定id的内容应用到当前分支的工作目录，内容不会删除，可以在多个分支上重复进行操作
	$ git stash apply stash名称
	
	//从堆栈中移除某个指定的stash
	$ git stash drop stash名称
	
	//清除堆栈中的所有内容
	$ git stash clear
	
	//查看堆栈中最新保存的stash和当前目录的差异。
	$ git stash show
	
	//从最新的stash创建分支。
	$ git stash branch
```

#### 代码标签

tag 中文我们可以称它为标签，tag 就是 对某次 commit 的一个标识，相当于起了一个别名。

【轻量标签 】： 只是某个commit 的引用，可以理解为是一个commit的别名；

【附注标签】 ：是存储在git仓库中的一个完整对象，包含打标签者的名字、电子邮件地址、日期时间以及其他的标签信息。它是可以被校验的，可以使用 GNU Privacy Guard (GPG) 签名并验证。

```
	//直接列出所有的标签
	$ git tag
	
	//可以根据<tagname>进行标签的筛选
	$ git tag -l <tagname*>
	
	//查看标签的提交信息
	$ git show 标签名
	
	//在提交历史中查看标签
	$ git log --online --graph
	
	//创建轻量标签
	$ git tag 标签名
	$ git tag 标签名 提交版本
	
	//创建附注标签
	$ git tag -a 标签名称 -m 附注信息
	$ git tag -a 标签名称 提交版本号 -m 附注信息
		-a : 理解为 annotated 的首字符，表示 附注标签
		
	//删除标签
	git tag -d 标签名称
	
	//将指定的标签上传到远程仓库
	$ git push origin <tagname>
	
	//将所有不在远程仓库中的标签上传到远程仓库
	$ git push origin --tags
	
	//删除远程仓库中的 指定标签
	$ git push origin  :regs/tags/<tagname>
	$ git push origin --delete <tagname>
```

#### 一台电脑多个ssh账号的注册

情况所有的ssh，不清空也行，清空更快一些

```
git config --global --unset user.name
git config --global --unset user.email
```

注册两个账号

```
# 生成github的ssh-key，注册邮箱填github的注册邮箱，然后在用户主目录下/.ssh/下会生成id_rsa(私钥)、id_rsa.pub(公钥)
$ ssh-keygen -t rsa -C "注册邮箱" -f ~/.ssh/id_rsa 
# 生成gitlab的ssh-key，注册邮箱填gitlab的注册邮箱,然后在用户主目录下/.ssh/下会生成id_rsa_work(私钥)、id_rsa_work.pub(公钥)
$ ssh-keygen -t rsa -C "注册邮箱" -f ~/.ssh/id_rsa_work
```

添加全局权限，可以不用每次都输密码

```
$ ssh-agent bash
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/id_rsa_work
$ ssh-add -l 
#如果添加成功，这里会打印对应的配置信息
```

配置config文件

```
$ vim ~/.ssh/config  //创建配置文件

# 账号1-github
HOST github.com #github别名
hostname github.com  #github地址
User githubUsername #github用户名
IdentityFile /home/user/.ssh/id_rsa_work #github私钥地址
PreferredAuthentications publickey #首选认证方式

# 账号2-creality
HOST gerrit #gitlab私服别名
hostname xxx.xx.xxx.xx #gitlab私服地址
Port xxxx #端口
User xxxxxxxxxx #填写gitlab私服的用户名
IdentityFile /home/user/.ssh/id_rsa_work #gitlab私钥地址
PreferredAuthentications publickey #首选认证方式
```

需要去网站上面配置ssh，参照最上面内容

测试是否连接成功

```
$ ssh -T git@github.com
$ ssh -T git@git.gitlab.com
```

遇到的问题
1.测试Gitlab的SSH Key是否连接成功，出现Permission denied (publickey).

```
$ ssh -T git@git.gitlab.com
git@git.gitlab.com: Permission denied (publickey).

#使用ssh -v查看详细日志
$ ssh -vT git@git.gitlab.com
...
...
debug1: send_pubkey_test: no mutual signature algorithm
debug1: No more authentication methods to try.
git@git.gitlab.com: Permission denied (publickey).
```

关注的最后面三行日志，显示没有匹配的算法，查阅相关资料，openssh8.2版本之后，默认关闭SSH-RSA算法，该算法存在安全隐患（OpenSSH to deprecate SHA-1 logins due to security risk | ZDNet），当然我们可以重新启用它，但存在安全风险。在config的Gitlab配置中添加如下一行

```
PubkeyAcceptedKeyTypes +ssh-rsa
再次运行
```

```
$ ssh -T git@git.gitlab.com
出现 You’ve successfully authenticated, but GitHub does not provide shell access，连接成功！
```

最后在对应的项目下设置对应的用户名以及邮箱名就可以快乐的提交代码了

```
git config  user.email "useremail"
git config  user.name "username"
```

## 私有库的访问

私有库的访问需要设置token，在git clone 的时候输入密码时把token输入进去，才可以实现git clone出来

```
Settings->Developer settings->Personal access tokens->Generate new token。
创建新的访问密钥，勾选repo栏，选择有效期，为密钥命名。
复制这段密钥。（注意，密钥只显示一次，记得妥善保管）
git push时，作为用户密码来使用。
```

私有库每次都需要输入账号密码的解决办法

全局设置（只有一个账号）

```
git config --global credential.helper store

git config --global user.email "email"
git confgi --global user.name "name"
```

局部设置（多个账号）

```
## local
git config --local credential.helper store

git config --local user.email "email"
git config --local user.name "name"
```

之后配置token

```
git remote set-url origin https://github.com/xxx/xxx.git
```

