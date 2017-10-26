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
