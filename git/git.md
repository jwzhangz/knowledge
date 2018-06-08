Git pull 强制覆盖本地文件
```
git fetch --all  
git reset --hard origin/master 
git pull
```
 
撤销 git add  文件
```
git status 先看一下add 中的文件 
git reset HEAD 如果后面什么都不跟的话 就是上一次add 里面的全部撤销了 
git reset HEAD XXX/XXX/XXX.java 就是对某个文件进行撤销了
```
 
撤销 git commit
```
git log 查看节点 
git reset commit_id（回退到上一个 提交的节点 代码还是原来你修改的） 
git reset -hard commit_id （回退到上一个commit节点， 代码也发生了改变，变成上一次的）
```
 
撤销提交过的代码
```
还原已经提交的修改 
此次操作之前和之后的commit和history都会保留，并且把这次撤销作为一次最新的提交 
git revert HEAD 撤销前一次 commit 
git revert HEAD^ 撤销前前一次 commit 
git revert commit-id (撤销指定的版本，撤销也会作为一次提交进行保存） 
git revert是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容。
```

合并commit
```
git log 查看有几个commit
git rebase -i HEAD~2
出现编辑窗口，把后面的pick 改为 s, :wq 保存
出现编辑窗口，保留一条字符串。
同步到远程分支 git push -f
```

保存用户名密码
```
git config --global credential.helper store
```

撤销操作
```
A- =  untracked 未跟踪
A  =  tracked 已跟踪未修改
A+ =  modified - 已修改未暂存
B  =  staged - 已暂存未提交
C  =  committed - 已提交未PUSH


A- -> B :  git add <FILE>
B  -> A- :  git rm --cached <FILE>
B  -> 删除不保留文件 :  git rm -f <FILE>
A  -> A- :  git rm --cached <FILE>
A  -> A+ : 修改文件
A+ -> A :  git checkout -- <FILE>
A+ -> B :  git add <FILE>
B  -> A+ :  git reset HEAD <FILE>
B  -> C :  git commit
C  -> B :  git reset --soft HEAD^
修改最后一次提交: git commit --amend

```


[使用git pull文件时和本地文件冲突怎么办？](http://www.01happy.com/git-resolve-conflicts/)  


linux git自动补全
```
获取 git-completion.bash
git源码 
https://github.com/git/git
也可以
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash 

复制 git-completion.bash 到用户目录
cp git/contrib/completion/git-completion.bash ~/.git-completion.bash

修改 .bashrc，添加
source ~/.git-completion.bash

复制 git-prompt.sh
cp git/contrib/completion/git-prompt.sh ~/.git-prompt.sh

修改 .bashrc，添加
source ~/.git-prompt.sh

.bashrc 中添加
export GIT_PS1_SHOWDIRTYSTATE=1
export GIT_PS1_SHOWSTASHSTATE=1
export GIT_PS1_SHOWUNTRACKEDFILES=1
export GIT_PS1_SHOWUPSTREAM="verbose git svn"
PS1='\[\033[1;32m\]\u@\h \[\033[1;34m\]\W\[\033[1;31m\]$(__git_ps1 " (%s)")\[\033[1;35m\] $ \[\033[0m\]'

```
