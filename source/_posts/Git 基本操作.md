---
title: Git 基本操作
date: 2019-3-30
tags: [git] 
---

---
### **1、git config 查看配置信息及config下面所有的操作**
---

查看系统config

```
git config --system --list
```
　　
查看当前用户（global）配置


<!--more-->

```
git config --global  --list
```

查看当前仓库配置信息

```
git config --local  --list
```

本地测试效果如图1所示。

config 配置有system级别 global（用户级别） 和local（当前仓库）三个 设置先从**system --> global-->local  底层配置会覆盖顶层配置** 分别使用--system/global/local 可以定位到配置文件。


--global中存储了提交用户的email和用户名 如果需要手动设置则可以使用如下指令:

```
    git config --global user.name "myname"
    git config --global user.email  "test@gmail.com"
```

---
### **2、git branch 分支管理**
---

```
 git branch 不带参数：列出本地已经存在的分支，并且在当前分支的前面加“*”号标记

 git branch -r 列出远程分支
  
 git branch -a 列出本地分支和远程分支
  
 git branch < branch name >创建一个新的本地分支，需要注意，此处只是创建分支，不进行分支切换
  
  
 git branch -m | -M oldbranch newbranch 重命名分支，如果newbranch名字分支已经存在，则需要使用-M强制重命名，否则，使用-m进行重命名。
 
 git push origin < branchname >   把分支推到远程

 git branch -d -r branchname 删除远程branchname分支(删除的时候可能会有权限的问题)
 
 git branch -d branchname  删除本地分支，有时候是需要 -D
 
 git push origin :chrou   删除远程的chrou分支  
  git push origin --delete chrou
``` 

---
### **3、git chekout 分支的操作**
---


```
## 操作文件

git checkout filename   //放弃单个文件的修改

git checkout .   //放弃当前目录下的修改

## 操作分支

git checkout master   //将分支切换到master

git checkout -b master //如果分支存在则只切换分支，若不存在则创建并切换到master分支，repo start是对git checkout -b这个命令的封装，将所有仓库的分支都切换到master，master是分支名，

 git checkout -b < branch name >  创建新分支并切换当前分支到新建的分支

```


---
### **4、git cherry-pick** 
---

git cherry-pick可以选择某一个分支中的一个或几个commit(s)来进行操作。

例如，假设我们有个稳定版本的分支，叫v2.0，另外还有个开发版本的分支v3.0，我们不能直接把两个分支合并，这样会导致稳定版本混乱，但是又想增加一个v3.0中的功能到v2.0中，这里就可以使用cherry-pick了。就是对已经存在的commit 进行 再次提交；

简单用法：


```
git cherry-pick <commit id>
```
当然，如果使用gerrit的话，也可以直接cherry-pick

注意：当执行完 cherry-pick 以后，将会生成一个新的提交；这个新的提交的commit id和原来的不同，但标识名 一样；


---
### **5、git merge 和 git rebase**
---

[闲谈 git merge 与 git rebase 的区别](https://www.jianshu.com/p/c17472d704a0)

git merge deve 操作：Git 会自动根据两个分支最开始出现分叉的commit-id和两个分支的最新提交即commit-id进行一个三方合并，然后将合并中修改的内容生成一个新的 commit-id

---
### **6、git reset**
---

reset是用来修改提交历史的


```
git reset --hard baeertasdasdvf   // 本地仓库回退到某个版本   
```

git 版本出错了，把本地回滚到之前某个版本，然后把远程分支删除，然后以本地的回滚版本重新建立分支。


```
repo upload -re = "chrou,lqlin"   // 添加代码审核的人（相当于替代了push和手动添加代码审核者这两个步骤）
```

**已经push到远程仓库的commit不允许reset**

---
### **7、git tag**
---

```
git tag //查看所有的标签

git tag -d tagName  //删除某一个标签

git tag -a tagName -m "annotate"  //创建带注释的标签 

git tag tagName  // 轻量级标签 

git checkout tagName  //切换到某一个标签 

git show < tagvalue >
```


### **8、git push**

```
$ git push origin master

上面命令表示，将本地的master分支推送到origin主机的master分支。如果master不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

$ git push origin :master

$ git push origin --delete master

上面命令表示删除origin主机的master分支。如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

$ git push origin

上面命令表示，将当前分支推送到origin主机的对应分支。如果当前分支只有一个追踪分支，那么主机名都可以省略。

$ git push

如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机，这样后面就可以不加任何参数使用git push。

$ git push -u origin master

上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。
```

### **9、git log**

```
git log  查看 当前分支的 提交历史

git log  --author=haiwang   --author 仅显示指定作者相关的提交


git log dev ^master   查看 dev 有，而 master 中没有的

git log --graph --pretty=oneline --abbrev-commit   // 查看简化的提交信息

```


如：
```
$  git log --graph --pretty=oneline --abbrev-commit

* 1eeacc3 (HEAD -> master, origin/master, origin/HEAD) [NBF] Add variables to solve the problem of the upper layer loading dependent webrtc code
* 42cf524 [NBF] modified readme descript
* 82922b9 [NBF] Modify configuration files and variables
* 2171b07 (tag: 1.0.0) Switch debug.js & adapter.js compression code to uncompressed code
* 31eb40d replace gsRTC.js with gsRTC.min.js and gsRTC.api.js
* 207cad5 Local environment configuration modification
* 36f24a7 Revert "delete error commit"
* b49e1a8 add
* 45c2333 delete unuseless statement
* 9f42355 [NBF] delete release mode file
* 8af1224 [NBF] Underlying code stripping
* 2fcd5de [NBF] Splited the WebRTC part source code from IPVT_WebRTC_Client REPO.
* 2c97353 Initial empty repository

```

### **10、git revert**

git 丢弃已合入仓库的某次提交


```
git branch branch-name  // 切换到对应的分支

git log   // 查看提交记录，找到想要revert的提交记录的 commit-id

git log --grep="commit-message提交时的说明信息"    //  或准确查找：

git revert commit-id // 丢弃这个修改

git push origin master  // 把本次丢弃提交到git，master为分支名，按需修改

```




[廖雪峰廖雪峰git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

[阮一峰git教程](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)





