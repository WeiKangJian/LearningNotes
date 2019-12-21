
**2019.12.21**

# git总结
之前一直在上传框架语言类的学习笔记，一直比较忙没时间把git这一块的学习体验上传上来，今天总算总结了这一块的内容，只是初步的内容，后续涉及到分支等内容还会再次进行更新。
## 区域概念
***工作区（Working Directory）， 版本库（Repository）/暂存区 ，（中央/远程）服务器.***

**服务器**的概念已经清楚了。叫做 中央服务器/远程服务器都行。一般是 github和SVN这样的公有仓库提供。

**工作区**:就是你电脑的工作目录（未comiit之前的，commit这个命令很重要）

**版本库**:工作区有一个隐藏的 .git文件夹，这个是叫做 版本库(有些文章也叫 暂存区，不管叫什么，知道这个意思就好)。.git 是隐藏文件夹。该文件内的内容很重要，因为git的控制配置等信息，都在这个隐藏文件夹里。电脑如果设置不显示隐藏文件夹，那么就会看不到。（commit之后即使将工作区提交到版本库当中）

## 一些常用命令
在git 的命令行的方式中一些命令和linux十分的相似，如cd..，ls,vi等，都是相似的命令命名方式，对于一个完整的创建仓库，上传代码且涉及到多人合作的项目来说，以下几个命令是快速入门的：

```
git init                                                 //初始化git仓库
```

```
git remote add origin （远程仓库地址）//用来链接到远程仓库
```

```
git add README.md                   //添加新增加的东西到暂存区
```

```
git commit -m "first commit"        //将暂存区的东西提交到版本库中并添加注释
```

```
git pull                                      //拉取远程仓库的内容到版本库中，可能有冲突
```

```
git push  origin master           //提交到远程仓库，可能有冲突
```

一般如果一个仓库的贡献者只有自己的话，上述命令已经能解决大部分的需求了，但是有时候还有一些其他的命令十分重要，可以帮助我们对当前的提交步骤进行查看处理：

`git status .：`查看当前路径下的的状态。git下最最常用的一个命令。


**下图展示的是未提交到暂存区时候的状态：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220162818505.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)

`git diff` ：显示当前版本库和工作区到底有哪些不同
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220162957526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
`git add` 之后，`git status`的结果;显示待提交（commit）的状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220163041663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
最后`git commit` 之后的，`git status`状态，显示可以push啦：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191220163348154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
## 冲突解决
为了避免潜在冲突一个好习惯就是 你在修改你的代码之前，先git pull一下，把服务器的最新代码拉下来,这是一个好习惯。因此我们开发时，有时候早晨来了第一件事情，就是先把代码git pull一下，进行更新。这是个好的开发习惯。避免你写了很多代码，你同事也写了很多代码，然后冲突了，你们俩合并的时候，比较浪费时间。
当我们忘记pull后，提交到本地仓库，然后再push的时候，会出现提交失败，这是因为之前有别人提交过自己版本库没有的内容，出现了冲突。可能有以下报错信息：

```
To git@git.oschina.net:yaowen369/ConcurrentDemo.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@git.oschina.net:yaowen369/ConcurrentDemo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
按照这个提示,可以看出，之所以被拒绝是因为 远程分支包含了你本地没有的内容，这通常是因为 另一个库推送了同样的文件(ref是索引的意思，可以翻译成文件)。你可以在推送之前先合并这些远程的变化(比如，试试 git pull)。一般出现问题后，git都会进行下一步操作的提示。一般如果冲突不是很大，比如同事在文件头增加了信息，你在文件尾加了内容，那么git会帮助你进行自动的冲突合并。如果出现复杂的冲突情况，（两人都对同一句话进行了修改）那么需要手动进行冲突的合并。

如果先commit之后，冲突报错，再pull回来的时候，会提示你哪个文件出现了冲突，这时候按照提示进入对应的文件合并冲突（在实际工作环境中这时候往往需要和同事进行讨论，来完成对冲突代码段的处理）。这时候进入文件查看，会有冲突代码段的提示，比如下面这样（讨论后确定是保留哪一个）：

```
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< HEAD
Creating a new branch is quick AND simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```
解决冲突保存后，再commit到版本库，然后在push后即可成功。

上面就是常用的git的一些操作，后续还有分支合并等内容后续会逐步的进行更新。
