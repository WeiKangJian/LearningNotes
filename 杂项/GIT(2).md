# git 进阶（分支）
## 一些命令记录在下面：

Git clone ssh …..克隆

Git —amend 让上一次提交被覆盖，代码被打回时候使用的参数

Git push origin HEAD:refs/for/[branch]   提交到临时分区待评审

Git checkout -b [本次分支] == git branch branchName.+Git checkout branchName

Git revert head  撤销上一次提交

Git reset —hard HEAD^  工作区和暂存区全恢复到上一次状态

Git Reset - -soft 保留工作目录，并把差异放进暂存区，即回退commit状态，不清空暂存区

Git reset HEAD. 清空暂存区，工作目录不更改

***git push的一般形式为 git push <远程主机名> <本地分支名>  <远程分支名>***
例如 git push origin master：refs/for/master ， origin 是远程主机名， 第一个master：本地分支名，第二个master:远程分支名。

(1):git push origin master
   如果远程分支被省略，如上则表示将本地分支推送到与之存在追踪关系的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。
(2): git push origin ：refs/for/master 
   如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支，等同于 git push origin --delete master
(3): git push origin
   如果当前分支与远程分支存在追踪关系，则本地分支和远程分支都可以省略，将当前分支推送到origin主机的对应分支 

## 关于HEAD^ 和HEAD~
1.建立如图所示分支
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229234716267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)

2.将HEAD指针向后移动一位到原分支git checkout HEAD^
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229234749886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
3.将HEAD指针向后移动一位到merge分支git checkout HEAD^2
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229234821585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
4.将HEAD指针向后移动两位到原分支git checkout HEAD^~
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229234852698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
5.将HEAD指针向后移动一位到merge分支git checkout HEAD^2~
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191229234915648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
