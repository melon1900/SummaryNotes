### 安装

windows
直接使用tortisegit就可好了，图形界面，没啥可说的。

```
https://tortoisegit.org/download/
```

CentOS/RHEL
```
yum install git-core
```
unbuntu

```
apt-get install git-core
```

初始化项目版本库

```
mkdir helloworld
cd helloworld
git init

```
### 配置

```
git config --global user.name "lhb"
git config --global user.email "lhb@163.com"
git config --global color.ui true
git config --global color.status auto
git config --global color.branch auto
git config --global core.editor vim

```
user.name和user.email设置为工作中常用的名字和邮箱，不要随便起名字，不然提交日志就混乱了。

配置分三级，优先级从高到低依次为：
- 版本库级 对应配置文件为工作区目录下.git/config
- 系统用户级(--global) 对应配置文件为用户home目录下的.gitconfig
- 系统级(--system) 一般对应配置文件为/etc/gitconfig（至少CentOS 6.x是）
高优先级的配置覆盖低优先级的配置

查看配置
```
git config --list
```

### 增删改

增

```
touch aa.c bb.c
git add aa.c bb.c
git commit -m "first add aa.c bb.c"

```
删

```
git rm bb.c
git commit -m "remove bb.c"
```
改

```
echo "#include <stdio.h>" > aa.c
git add aa.c
git commit -m "add include for aa.c"
```
git add/git rm只是把修改添加到暂存区(stage area)，需要git commit把暂存区的内容提交到当前分支。
暂存区，说白了就是缓冲区，我们把对工作区的修改（文件的增、删、改）先放到暂存区，一来避免了频繁提交，二来呢相当于给出了个犹豫时间，比如改着改着感觉其它放到暂存区的修改改错了，我们可以撤销暂存区的修改。


### 改什么了

哪些文件有改动

```
git status
```
git status用来查看工作区的当前状态，git的status做的很好，当你调用git status命令时，它会告诉你现在处在哪个分支，当前工作区有没有改动，有改动的话都有哪些修改，哪些修改已经加入到暂存区，哪些修改还未加入到暂存区，你需要提交的修改的话你该调用什么命令，丢弃你的修改该调什么命令，很像一个小指南。


通过git status知道了哪些文件有修改，那具体某个文件改什么了呢?
```
git diff HEAD -- aa.c
```
HEAD就相当于一个指针，总是指向当前所处分支的已提交的最新版本，HEAD\^表示上一个版本，HEAD\^\^表示上上一个版本，以此类推。上10个版本怎么办？要写10个\^吗？当然不用，可以用HEAD~10来表示，其实像HEAD~10基本用不到，不如直接用commit id来的清晰明了。--表示当前工作区版本。

### 改错了

#### 还没提交

```
git checkout -- [<file>]
```
没指定<file>的话就表示撤销整个工作区的，指定了的话就是撤销所制定file的修改。如果该文件加入到暂存区后又做了修改，就会撤销到刚加入暂存区后的状态；如果该文件在上次提交后做了修改还没有加入到暂存区，就会撤销到上次提交后的状态。

#### 已经提交了

##### 回滚到上一个版本
```
git reset --hard HEAD^
```
--hard表示回滚到上一版本的同时，舍弃自上次提交后对工作区的修改，与之对的是--soft会保留当前工作区已做的修改

##### 回滚到某个版本

首先要根据提交历史，确定要回滚到哪个历史版本

```
$git log --pretty=oneline --abbrev-commit
6b91f33 add printf for main
6285be4 add main for aa.c
015ccc1 add aa.c bb.c
```
然后

```
git reset --hard <commit id>
```

##### 回滚错了呢？

万一一个误操作回退到了某个历史版本怎么办呢？莫慌，办法还是有的。其实git的版本回滚只是将HEAD指针指向了某次commit，我们只要知道commit id就可以任意切换。但是回滚到某个历史版本后，该版本之后的提交历史通过git log就看不到了，怎么办呢? 

```
$ git reflog
6285be4 HEAD@{0}: 6285be4: updating HEAD
6b91f33 HEAD@{1}: commit: add printf for main
6285be4 HEAD@{2}: commit: add main for aa.c
015ccc1 HEAD@{3}: commit (initial): add aa.c bb.c
```
git reflog查看命令历史，在这里我们可以找到commit id。

### 修bug

不管是修bug，还是加feautre，标准做法都是创建一个分支来修改，修改测试完毕后再merge到master分支，这样保证master的每个版本都是稳定可用的版本。下来就来看一下关于分支的一些操作：

创建

```
git branch <name>
```
切换

```
git checkout <name>
```
创建&切换

```
git checkout -b <name>
```
合并某分支到当前分支

```
git merge <name>
```
删除

```
git branch -d <name>
```
假如分支还没被合并过，依然需要删除，需要使用-D选项

查看

```
git branch
```

假如我正在feature分支上加feature呢，这时突然线上有bug了，需要马上处理，但我现在的代码修改还不完整，还不能提交，还需要较长时间才能完成，而这个bug个把小时就能修好，这时该怎么办呢？git提供了stash命令来保存当前工作区尚未提交的更改。

比较正规的修bug流程，假如你当前正在feauture_xxx上加feauture，需要修master分支上的bug
```
git stash
git checkout master
git checkout -b issue123
....(fixing issue123)
git commit -m "issue 123 is fixed"
git checkout master
git merge --no-ff -m "the fix of issue 123 is merged" issue123
git branch -d issue123
git checkout feature_xxx
git stash pop

```
可以多次stash，恢复的时候，先
```
git stash list
stash@{0}: WIP on feature_xxx: 6285be4 add main for aa.c
stash@{1}: WIP on master: 6285be4 add main for aa.c
```
恢复指定的stash
```
git stash apply stash@{0}
```
用git stash apply恢复后，stash内容并不删除，需要用
```
git stash drop
```
来删除，或者直接用

```
git stash pop
```
恢复的同时把stash内容也删除

### tag

创建tag
```
git tag <tagname> [commit id]
```
或
```
git tag -a <tagname> -m "some comment..." [commit id]
```
查看所有tag

```
git tag
```
查看某一tag

```
git show <tagname>
```
推送一个tag到远程仓库

```
git push origin <tagname>
```
推送所有tag到远程仓库

```
git push origin --tags
```
删除本地tag

```
git tag -d <tagname>
```
删除远程tag

```
git push origin :refs/tags/<tagname>
```

.gitignore

这个文件的内容通常不用我们自己从头来写，对于每种开发语言的通用配置github已为我们备好https://github.com/github/gitignore ，我们只需把对应的.gitignore放到我们的版本中根据我们特定的需求合并修改即可。这个文件本身需要加入到版本控制。

强制添加被.gitignore过滤的文件

```
git add -f <file>
```
检验一下是不是.gitignore过滤规则的问题

```
git check-ignore -v <file>
```

### 远程仓库

现在功能需求越来越多，一个人已力不从心，需要别的小伙伴来帮忙，该怎么办？我电脑万一哪天挂掉了呢？我辛辛苦苦的劳动成果岂不都灰飞烟灭了吗？这时我们有两个选择：一个是使用像github这样的服务，一个是自己搭建一个git服务器。

#### github

首先要有github帐号，而且由于本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以你需要为你的github帐号添加SSH key。首先在你的工作电脑上

```
ssh-keygen -t rsa -C "your email@xxx.com"
```
在用户的home目录下的.ssh目录下会有id_rsa.pub，这是SSH Key的公钥，登录github将SSH key添加进来（Account settings->SSH and GPG Keys->Add SSH Key，然后拷贝id_rsa.pub的内容到key里面），如果你有多台工作电脑那就需要在每台电脑的key添加进来。

接下来我们就可以创建一个仓库，比如helloworld，来存放我们本地的helloworld仓库了。

```
git remote add origin git@github.com:<your github name>/helloworld.git
```
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，其实说白了就给远程库起了个别名。你可以添加多个，比如你需要将不同的分支存放到不同的远程库中，不过一般这么做的少。
接口下来就可以把我们的本地库推送远程库了：
如果想推送主分支，而当前没在主分支，需要先git checkout master切换到主分支

```
git push -u origin master
```
如果你在github上创建helloworld仓库时创建了README文件，那么你很可能会遇到如下错误：

```
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:melon1900/helloworld.git'
To prevent you from losing history, non-fast-forward updates were rejected
Merge the remote changes before pushing again.  See the 'Note about
fast-forwards' section of 'git push --help' for details.
```
这是因为远程仓库有提交操作，你需要先更新

```
git pull origin master
```
这样我们github上helloworld仓库的master分支就和我们本地仓库master的分支一样了，由于git push的时候加了-u参数，git会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。比如在master分支下提交修改后，就可以直接git push将修改同步到github上。体现在.git/config中就是多了如下内容：

```
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

#### 自建服务器

首先你要有至少两台电脑或虚拟机，且互通，假设你要两台电脑ip分别为192.168.1.100和192.168.1.101，我们准备把git服务器架设在192.168.1.100上。

在192.168.1.100上，假设你是以root用户操作的，非root用户sudo或su获取root权限来操作。

安装git，并创建git用户

```
yum install git-core
adduser -s /usr/bin/git-shell git

```
创建一个git仓库

```
git init --bare /teamx/helloword.git
chown -R git:git /teamx/helloword.git
```
创建authorized_keys文件，如果不存在的话

```
cd /home/git
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chown -R git:git .ssh
```
在192.168.1.101上执行

```
cat ~/.ssh/id_rsa.pub | ssh root@192.168.1.100 "cat - >> /home/git/.ssh/authorized_keys"
```
将id_rsa.pub的内容追加到192.168.1.100上的/home/git/.ssh/authorized_keys文件中。如果192.168.1.101上没有id_rsa.pub，通过ssh-keygen创建一个即可。

接着克隆一下远程仓库，检验一下git服务器是否搭建成功

```
git clone git@192.168.1.100:/teamx/helloword.git
```
至此一个简易的git服务器就搭建成功了。

### 结语
本文内容掌握了的话，基本就可以使用git工作了，但本文所提内容只是git的冰山一角，工作中难免会遇到一些问题，可以尝试如下方式解决：
- git help <command>一下
- 谷歌一下
- stackoverflow上搜索或提问
- 研读一下git官方文档

祝工作愉快！

参考资料：http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000