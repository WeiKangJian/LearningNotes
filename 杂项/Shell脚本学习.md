# Shell脚本学习
shell用来做自动化测试，检查部署等任务都十分的方便，一些部署在开发机上的程序有很多步骤都要用到shell来完成，对于shell的学习记录如下：
第一行：
***#!/bin/bash*** ：用来指定脚本用那个解释器来执行

***echo***: 向窗口输出文本

## 变量
在shell中，变量的定义和使用如下：

```
your_name="runoob.com"
```
*注意！变量名和等号之间不能有空格 不能使用bash里的关键字（可用help命令查看保留关键字）*

有时候我们想要对一个变量赋予一个语句的执行结果，而不是单纯的指定字符串，这时候有两种实现方式：

```
for file in `ls /etc`
或
for file in $(ls /etc)
```
变量赋予值之后，取值只需要在变量之前加一个$即可

### 单双引号

在shell中，并不指定变量的类型，即类似于都是字符串的形式，值得注意的是，在shell中，字符串的定义分为单引号和双引号两种类型，
* 单引号：单引号中$变量取值无效
* 单引号：字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。

* 双引号：双引号里可以有变量
* 双引号：里双引号里可以出现转义字符

例：

```
#!/bin/bash
str='this is test'
str2="$str"
echo $str2
```
输出 this is a test
```
#!/bin/bash
str='this is test'
str2='$str'
echo $str2
```
输出$str

```
#-e 激活转义字符
str="Hello, I know you are \"$your_name\"! \n"
```
转义，输出：Hello, I know you are "runoob"! 

### 取字符串长度

```
string="abcd"
echo ${#string} #输出 4
```
### 提取子字符串

```
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```
###  Shell数组
用到再说

##  传递参数
## 运算符
##  流程控制
## 函数
