---
title: Linux-Shell
layout: post
tags: Linux基础
categories: 'Linux'
---

### 1.shell简介

#### 1.简介

Linux系统可以划分为四个部分：

1. 应用软件
2. 窗口管理软件
3. GNU系统工具链
4. Linux内核

<img src="../../assets/images/20210811Linux-shell/image-20210822152235927.png" alt="image-20210822152235927" style="zoom:50%;" />

Linux shell 是一种特殊的交互式工具，它提供了文件管理、运行进程的的途径。

Shell的核心是命令提示符，允许用户输入命令，然后解释命令，并在内核中执行。

用户可以编写脚本文件，将多个shell命令以某种形式组织起来，作为程序一起执行

#### 2.常见的shell

1. shell有很多种，不同的shell有不同的特性

   | shell | 描述                                                         |
   | ----- | ------------------------------------------------------------ |
   | bsh   | 一种最早出现在Unix系统上的标准shell，全程叫Bourne shell      |
   | bash  | 一种对bsh在功能上进行扩展的shell，几乎可以涵盖shell所需要的所有功能 |
   | ksh   | 一种与bsh兼容的编程shell，增加了很多特性，常见于Unix操作系统 |
   | tcsh  | 一种具有C语言风格语法结构的shell，常见于嵌入式开发           |
   | zsh   | 一种结合了bash、ksh和tsh的特点，同时提供了高级编程特性的高级shell |
   | sh    | 绝大部分Linux发行版中，作为软链接指向其他shell（默认是bash） |

2. 几乎所有的Linux发行版默认shell是bash shell

3. 有些发行版默认系统shell和默认交互shell并不相同

4. 查看系统支持的shell类型

```sh
root@luckydog:~# cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash

root@luckydog:~# cat /etc/shells |xargs ls -al
ls: cannot access '#': No such file or directory
ls: cannot access '/etc/shells:': No such file or directory
ls: cannot access 'valid': No such file or directory
ls: cannot access 'login': No such file or directory
ls: cannot access 'shells': No such file or directory
-rwxr-xr-x 1 root root 1183448 Jun 18  2020  /bin/bash
-rwxr-xr-x 1 root root  129816 Jul 19  2019  /bin/dash
lrwxrwxrwx 1 root root       4 Jun 18  2020  /bin/rbash -> bash
lrwxrwxrwx 1 root root       4 Jun 23 11:28  /bin/sh -> dash
-rwxr-xr-x 1 root root 1183448 Jun 18  2020  /usr/bin/bash
-rwxr-xr-x 1 root root  129816 Jul 19  2019  /usr/bin/dash
lrwxrwxrwx 1 root root       4 Jun 18  2020  /usr/bin/rbash -> bash

# 查看当前交互shell类型是 bash
root@luckydog:~# echo $SHELL
/bin/bash
```

### 2.shell常用命令

#### 1.常见命令

1. 管理文件和目录：

   ```sh
    cd 		pwd		ls		touch	cp		 mv	rm	mkdir	rmdir	file	cat   more      less      tail      head
   ```

2. 管理系统进程

   ```sh
   ps      top       kill      killall
   ```

3. 管理磁盘空间

   ```sh
   mount    umount   df   du 
   ```

4. 处理数据文件

   ```sh
   sort    grep     gzip     tar 
   ```

shell命令帮助手册： **man** [command]

#### 2.外部命令

1. 外部命令也成为文件系统命令，通常位于/bin、/sbin、/usr/bin、/usr/sbin等目录中

   ```sh
   # 查看命令位置
   root@luckydog:~# which ls
   /usr/bin/ls
   ```

2. fork：外部命令执行时，会创建一个子进程。

   ```sh
   root@luckydog:~# ps -f
   UID          PID    PPID  C STIME TTY          TIME CMD
   root       74229   74187  0 15:32 pts/0    00:00:00 -bash
   root       74259   74229  0 15:33 pts/0    00:00:00 /bin/sh
   root       74260   74259  0 15:33 pts/0    00:00:00 /bin/bash
   root       74269   74260  0 15:34 pts/0    00:00:00 /bin/bash
   root       74306   74269  0 15:53 pts/0    00:00:00 ps -f
   ```

   可以看到，执行`ps -f`命令时，会在父进程（74269）下创建一个子进程（74306）

#### 3.内建命令

1. 作为shell工具的组成部分，内建命令不需要使用子进程来执行

2. 对于有些命令，有多种实现，既有外部命令，也有内建命令。

3. 了解某个命令的类型

   ```sh
   type -a pwd
   
   # pwd既是外部命令，也是内建命令
   root@luckydog:~# type -a pwd
   pwd is a shell builtin
   pwd is /usr/bin/pwd
   pwd is /bin/pwd
   
   ```

4. 了解所有内建命令

   ```sh
   man builtin
   ```

### 3.shell脚本基础

#### 1.编写

1. 脚本创建、执行与退出状态码

   第一行需要指定shell类型shebang

    script.sh:

   ```shell
   #!/bin/bash
   date
   who
   ```

2. 执行脚本

   * 增加脚本可执行权限 （chmod）
   * 使用绝对/相对路径执行shell脚本

   ```sh
   chmod +x script.sh
   ./script.sh
   ```

3. 脚本的退出状态码

   ```sh
   # 通过 $?查看脚本执行状态
   # 查看状态： 0 - 执行成功；其他数字 - 执行失败
   $?
   ```

#### 2.变量定义与使用

1. Linux系统的环境变量

   通过`set`查看：

   * 局部环境变量

     通常是小写字母，只在其定义的进程中可见

   * 全局环境变量

     通常为大写字母，全局可见

2. 用户自定义变量

   用户可以自定义由字母、数字和下划线组成，长度不超过20个字符，区分大小写

3. 变量的定义和赋值

   **等号两边不能有空格**

4. 使用美元符$对变量进行引用

   建议使用**`${variable_name}`**

5. 命令替换：将命令的输出赋值给变量

   * 反引号`command`
   * 推荐使用 **$(command)**

```sh
#!/bin/bash

date
echo "current user is ${USER}"
echo "current user UID is ${UID}"
echo "current user HOME path is ${HOME}"
```

```sh
root@luckydog:~# chmod +x script
root@luckydog:~# ./script
Sun 22 Aug 2021 04:19:48 PM CST
current user is root
current user UID is 0
current user HOME path is /root
```

TIP: echo语句或解释语句内容（不包括通配符），所以 如果语句不包含对通配符的解释，一律用双引号括起来。

#### 3.数学运算

略

### 4.shell脚本条件控制

#### 1. if-then语句

```sh
if command
then 
	commands
fi

--------------------------

if command; then
	commands
fi
```

#### 2.if-elif-then

```sh
if command
then
	commands
elif command
then
	commands
else 
	commands
fi
```

#### 3.条件测试test命令

```sh
if test condition
then 
	commands
fi

---------------------
# 方括号两边必须加上空格
if [ condition ]
then
	commands
fi
```

##### 1.数值比较

|      -eq      |         -ne         |        -gt         |           -ge           |       -lt        |         -le          |
| :-----------: | :-----------------: | :----------------: | :---------------------: | :--------------: | :------------------: |
| 等于（equal） | 不等于（not equal） | 大于（great than） | 大于等于（great equal） | 小于（less than) | 小于等于(less equal) |



```sh
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```

##### 2.字符串比较

| =    | ！=    | -z 字符串     | -n 字符串       |
| ---- | ------ | ------------- | --------------- |
| 等于 | 不等于 | 字符串长度为0 | 字符串长度不为0 |

```sh
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
```

##### 3.文件比较

| 参数      | 说明                                 |
| :-------- | :----------------------------------- |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |
| -a        | 与运算                               |
| -o        | 或运算                               |
| ！        | 非运算                               |

```sh
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
```

另外，Shell 还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来，其优先级为： **!** 最高， **-a** 次之， **-o** 最低。例如：

```sh
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

#### 4.if-then高级特性-双括号

* 支持高级**数据表达式**计算
* 命令格式：((expression))
* expression可以是数据赋值或者比较表达式

script01.sh

```sh
# 赋值等号没有空格，必须连着写
#!/bin/bash
value=10
if (( ${value} ** 2 >90))
then
	((answer = ${value} ** 2))
	echo "the answer of ${value} ** 2 is ${answer}."
fi
```

```sh
root@luckydog:~# ./script01.sh
the answer of 10 ** 2 is 100.
```

#### 5.if-then高级特性-双方括号

* 支持针对**字符串**比较的高级特性
* 命令格式 [[ expression ]] 
* 除了标准的字符串比较，还支持模式匹配

script02.sh

```sh
#!/bin/bash
value=10
if [[ ${USER} == r* ]]
then
	echo "the USER name is ${USER}."
fi
```

```sh
root@luckydog:~# ./script02.sh
the USER name is root.
```

#### 6.case语句

```shell
case expression in 
	pattern1)
		commands;;
    pattern2)
    	commands;;
   	pattern3)
   		commands;;
   	*)
   		default commands;;
 esac
```

script03.sh

```sh
case ${USER} in 
	mysql)
		echo "current user is mysql";;
	root)
		echo "current user is root";;
	*)
		echo "unknown user.";;
esac
		
```

```sh
root@luckydog:~# ./script03.sh
current user is root
```

### 5.shell脚本循环控制

#### 1.for语句

1. 用于遍历一个制定的列表，每次迭代使用列表中的一个元素，执行好定义好的一组命令。
2. for语句格式

```sh
for var in list
do 
	commands
done

--------------------
for var in list; do 
	commands
done
```

3. for 命令的使用

script04.sh

```sh
#!/bin/bash

for fruit in apple lemon orange
do
	echo ${fruit}
done
```

```sh
root@luckydog:~# ./script04.sh
apple
lemon
orange
```

4. 从命令中读取数据列表

   将变量写入文本文件`fruitlist.txt`中

```sh
echo -e "apple\nlemon\norange\n" > fruitlist.txt
```

script05.sh

```sh
#!/bin/bash
file='fruitlist.txt'
for fruit in $(cat ${file})
do 
	echo "this is $fruit"
done
```

```sh
root@luckydog:~# ./script05.sh
this is apple
this is lemon
this is orange
```

5. 更改字段分隔符 $IFS

script06.sh

```sh
#!/bin/bash
fruitlist='apple:lemon:orange'
IFS=:
for fruit in $fruitlist
do 
	echo "this is $fruit"
done
```

```sh
root@luckydog:~# ./script06.sh
this is apple
this is lemon
this is orange
```

6. 通配符遍历

script07.sh

```sh
#!/bin/bash
for file in ./*
do 
	if [[ ${file##*.} = sh ]]
	then
		echo "sh文件:$file"
	elif [[ ${file##*.} = txt ]]
	then
		echo "txt文件:$file"
	fi
done
```

```sh
root@luckydog:~# ./script07.sh
txt文件:./fruitlist.txt
sh文件:./get-docker.sh
sh文件:./script01.sh
sh文件:./script02.sh
sh文件:./script03.sh
sh文件:./script04.sh
sh文件:./script05.sh
sh文件:./script06.sh
sh文件:./script07.sh
```

#### 2.while语句

```sh
while [ command ]
do 
	commands
done
```

script08.sh

```sh
#!/bin/bash
i=1
while [ $i -le 10 ]
do
	echo "the number is $i"
	((i = i + 1))
done
```

```sh
root@luckydog:~# ./script08.sh
the number is 1
the number is 2
the number is 3
the number is 4
the number is 5
the number is 6
the number is 7
the number is 8
the number is 9
the number is 10
```

#### 3.循环控制break

break可以跳出任意类型的循环

1. 跳出内部循环, 使用`break`
2. 跳出外层循环，使用`break n`，其中n为整数

script09.sh  [九九乘法表]

```sh
#!/bin/bash
for i in $(seq 1 9)
do 
	for j in $(seq 1 9)
	do
		echo "$i * $j = $(($i*$j))"
	done
done

```

```sh
root@luckydog:~# ./script09.sh
1 * 1 = 1
1 * 2 = 2
1 * 3 = 3
1 * 4 = 4
1 * 5 = 5
1 * 6 = 6
1 * 7 = 7
1 * 8 = 8
1 * 9 = 9
2 * 1 = 2
2 * 2 = 4
2 * 3 = 6
2 * 4 = 8
2 * 5 = 10
2 * 6 = 12
2 * 7 = 14
2 * 8 = 16
2 * 9 = 18
3 * 1 = 3
3 * 2 = 6
3 * 3 = 9
3 * 4 = 12
3 * 5 = 15
3 * 6 = 18
3 * 7 = 21
3 * 8 = 24
3 * 9 = 27
4 * 1 = 4
4 * 2 = 8
4 * 3 = 12
4 * 4 = 16
4 * 5 = 20
4 * 6 = 24
4 * 7 = 28
4 * 8 = 32
4 * 9 = 36
5 * 1 = 5
5 * 2 = 10
5 * 3 = 15
5 * 4 = 20
5 * 5 = 25
5 * 6 = 30
5 * 7 = 35
5 * 8 = 40
5 * 9 = 45
6 * 1 = 6
6 * 2 = 12
6 * 3 = 18
6 * 4 = 24
6 * 5 = 30
6 * 6 = 36
6 * 7 = 42
6 * 8 = 48
6 * 9 = 54
7 * 1 = 7
7 * 2 = 14
7 * 3 = 21
7 * 4 = 28
7 * 5 = 35
7 * 6 = 42
7 * 7 = 49
7 * 8 = 56
7 * 9 = 63
8 * 1 = 8
8 * 2 = 16
8 * 3 = 24
8 * 4 = 32
8 * 5 = 40
8 * 6 = 48
8 * 7 = 56
8 * 8 = 64
8 * 9 = 72
9 * 1 = 9
9 * 2 = 18
9 * 3 = 27
9 * 4 = 36
9 * 5 = 45
9 * 6 = 54
9 * 7 = 63
9 * 8 = 72
9 * 9 = 81
```

在其中加入break，跳出当前循环

```sh
#!/bin/bash
for i in $(seq 1 9)
do 
	for j in $(seq 1 9)
	do
		if [ $j -gt 3 ]
		then
			break
		else
			echo "$i * $j = $(($i*$j))"
		fi
	done
done
```

```sh
root@luckydog:~# ./script10.sh
1 * 1 = 1
1 * 2 = 2
1 * 3 = 3
2 * 1 = 2
2 * 2 = 4
2 * 3 = 6
3 * 1 = 3
3 * 2 = 6
3 * 3 = 9
4 * 1 = 4
4 * 2 = 8
4 * 3 = 12
5 * 1 = 5
5 * 2 = 10
5 * 3 = 15
6 * 1 = 6
6 * 2 = 12
6 * 3 = 18
7 * 1 = 7
7 * 2 = 14
7 * 3 = 21
8 * 1 = 8
8 * 2 = 16
8 * 3 = 24
9 * 1 = 9
9 * 2 = 18
9 * 3 = 27
```

写入break 2 ，跳出两层循环

```sh
#!/bin/bash
for i in $(seq 1 9)
do 
	for j in $(seq 1 9)
	do
		if [ $j -gt 3 ]
		then
			break 2
		else
			echo "$i * $j = $(($i*$j))"
		fi
	done
done
```

```sh
root@luckydog:~# ./script10.sh
1 * 1 = 1
1 * 2 = 2
1 * 3 = 3
```

#### 4.循环控制continue

1. continue可以跳过执行当前循环的命令，但不会终止整个循环
2. 可指定跳过的循环层数 continue n

continue的本质就是跳出本次循环，继续执行下一个循环

### 6.shell处理用户输入

#### 1.命令行输入

1. 命令行参数输入是向shell脚本传递数据的最基本方法
2. 位置参数：`$0` 是脚本名； `$1`到`$9`表示第一到第九个参数，第十个参数可以用`${10}`表示
3. 若参数中存在空格，需要用双引号引用

script11.sh

```sh
#!/bin/bash

echo "hello $1"
echo "hello $2"
```

```sh
root@luckydog:~# ./script11.sh
hello
hello


root@luckydog:~# ./script11.sh alice ted
hello alice
hello ted


root@luckydog:~# ./script11.sh alice "Li Ming"
hello alice
hello Li Ming
```

#### 2.特殊参数变量 $#

`$#`存储了脚本运行时携带的命令行参数的个数

使用变量可以判断传入参数的个数

#### 3.shift命令

1. shift命令可以将每个参数的变量向左移动一个位置，

   如使用一次shift，`$1`的值被删除，此时`$1`的值为`$2`以前的值、`$2`的值为`$3`以前的值；`$0`始终不变，为脚本名

2. 一次移动多个位置，使用`shift N`

#### 4.交互式处理-read命令

script12.sh

```sh
#!/bin/bash

# -p表示输入参数
read -p "enter your name: " first last

echo "your first name is $first"
echo "your last name is $last"
```

```sh
root@luckydog:~# ./script12.sh
enter your name: Li Ming
your first name is Li
your last name is Ming
```

script13.sh

```sh
#!/bin/bash

# -n1 表示输入一个字符，存入到后面的ans中
read -n1 -p "Do you want to continue [Y/n]" ans
echo 
case $ans in 
	Y | y)
		echo "YES";;
	N | n)
		echo "Bye";;
esac
```

```sh
root@luckydog:~# ./script13.sh
Do you want to continue [Y/n]y
YES

root@luckydog:~# ./script13.sh
Do you want to continue [Y/n]n
Bye
```

script14.sh

```sh
#!/bin/bash

# -p表示输入参数,-t 5 表示5秒不输入则返回非0状态码，并退出
if read -t 5 -p "enter your num: " num
then 
	echo "your num is $num"
else 
	echo "Sorry timeout!"
fi
```

script15.sh

读取文件中的参数

```sh
#!/bin/bash

filename='data.txt'
if [ -f ${filename} ];then
	count=1
	cat data.txt |while read line
	do 
		echo "line ${count} :${line}"
		count=$[ $count + 1 ]
    done
    echo "Finished reading the file"
else
	echo "${filename} is not exist!"
fi
```

````sh
root@luckydog:~# ./script15.sh
data.text is not exist!

root@luckydog:~# ./script15.sh
line 1 :this is the test file
line 2 :there are two lines!
Finished reading the file
````

### 7.脚本重定向

#### 1.标准文件描述符

Linux通过标准文件描述符来标识每个文件对象

| 文件描述符 | 缩写   | 描述     |
| ---------- | ------ | -------- |
| 0          | STDIN  | 标准输入 |
| 1          | STDOUT | 标准输入 |
| 2          | STDERR | 标准错误 |

#### 2.重定向普通和错误信息

* 只重定向错误输出

  ```sh
  ls -al badfile 2 > error.log
  ```

* 重定向错误和普通输出

  ```sh
  ls -al badfile testfile 2 > error.log 1 > output.log
  ```

* 重定向到同一个文件

  ```sh
  ls -al badfile testfile &> output.log
  ```


1. 在脚本中重定向输出

   临时重定向：

   * 重定向到文件描述符时，必须在文件描述符数字前加一个`&`
   * `echo "this is a error message" >&2`

   script16.sh

   ```sh
   #!/bin/bash
   
   echo "this is a error message" >&2
   echo "this is a normal message"
   ```

   ```sh
   root@luckydog:~# ./script16.sh
   this is a error message
   this is a normal message
   
   
   root@luckydog:~# ./script16.sh 2>error.log
   this is a normal message
   root@luckydog:~# cat error.log
   this is a error message
   ```

   永久重定向

   * 使用`exec`命令在脚本执行期间重定向某个特定的文件描述符
   * exec命令会启动一个shell来进行数据重定向

   script17.sh

   ```sh
   #!/bin/bash
   
   # 将普通输出定位到output.log文件中
   exec 1> output.log
   echo "this is a normal message"
   ```

   ```sh
   root@luckydog:~# ./script17.sh
   root@luckydog:~# cat output.log
   this is a normal message
   ```

   

2. 创建自己的重定向

   除了上面的0、1、2三个描述符之外，其余4、5、6、7、8、9都可以作为自定义文件描述符

   * 创建文件描述符

     ```sh
     exec 3>output_file   # 输出文件描述符
     
     exec 0<input_file	# 输入文件描述符
     
     exec 6 <>test_file  # 读写文件描述符
     ```

   * 关闭文件描述符

     ```sh
     exec 6>&-
     exec 6<&-
     ```

   * 实现一个简单的线程池

     ```sh
     # 脚本需要并发的执行任务
     # 令牌桶模型控制并发数
     ```

   script18.sh

   ```sh
   #!/bin/bash
   
   n_thread=3
   tmp_fifofile='file.fifo'
   rm -f ${tmp_fifofile}
   
   mkfifo ${tmp_fifofile}
   exec 6<> ${tmp_fifofile}
   
   for (( i=1; i <=${n_thread}; i++));do
   	echo >&6
   done
   
   for i in $(seq 100 200);do
   	read -u6
   	{
   		echo "thread handling number : ${i}"
   		sleep 3
   		echo >&6
   	}&
   done
   
   wait
   
   rm ${tmp_fifofile}
   exec 6>&-
   exec 6<&-
   
   ```

### 8.shell脚本函数

#### 1.基本的脚本函数

 1. 创建函数

    ```sh
    # 第一种方法
    function func_name{...}
    ```

    ```sh
    # 第二种方法
    func_name(){...}
    ```

2. 使用函数

   ```sh
   func_name
   ```

#### 2.在函数中使用变量

​	1.向函数传递参数

​		位置参数变量

​	2.在函数中处理变量

		* 全局变量	
		* 局部变量

#### 3.函数的返回值

1. 默认的退出状态码

   $?

2. 使用return命令

   只能获取 整数 0-255，否则使用下面第三种方法

3. 使用命令执行获取函数的输出

   ans = $(func)

### 9.shell脚本实例

目标： 获取指定目录中占用空间大小前十的子目录，并生成每日报告

核心命令：`du -s [directionary] sort -rn`

核心功能：准备需要检测的目录，得到目录统计数据，将结果生成每日报告

完善脚本: 执行验证，设置计划任务

#### 1.后台开启python服务，并开机自运行

1. 准备shell文件和python文件

   test----|---- start.sh

   ​		   |----back_run.py

   *start.sh*

   ```sh
    #!/bin/bash
    cd /test
    python -u back_run.py > out.log 2>&1 &
   
   ```

   *back_run.py*

   ```sh
    import os,datetime,time
    while True:
       print('{}'.format(datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S'))+' running...')
   	time.sleep(1)
   ```

2. 编写服务文件

   ```sh
   vim /etc/systemd/system/python_test.service
   ```

   ```sh
    1 [Unit]
     2 Description=python-test
     3 After=network.target
     4 StartLimitIntervalSec=0
     5 [Service]
     6 Type=simple
     7 Restart=always
     8 RestartSec=1
     9 User=root
    10 ExecStart=/usr/bin/bash /test/start.sh  // 必须为绝对路径
    11
    12 [Install]
    13 WantedBy=multi-user.target
   ```

   ```sh
   # 修改权限
   chmod 755  /etc/systemd/system/python_test.service
   # 重载守护
   systemctl daemon-reload
   # 开启开机启动服务
   systemctl enable python_test.service
   ```

3. 重启服务：

   ```sh
   sudo reboot
   ```

4. 附录–一些关于systemctl的命令


   ```sh
   # 查看所有服务的状态
   systemctl status
   
   # 查看单个任务
   systemctl status python_test.service
   
   # 停止服务
   systemctl stop python_test
   
   # 手工启动服务
   systemctl start python_test
   
   # 禁用开机启动
   systemctl disable python_test.service
   ```

参考链接

[在Ubuntu上20.04编写一个开机自启动的Python脚本]: https://blog.csdn.net/qq_43030934/article/details/116238278

