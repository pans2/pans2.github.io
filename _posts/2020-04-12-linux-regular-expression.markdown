---
layout:     post
title:      "Linux上的正则表达式"
subtitle:   "第一篇博客致敬linux启蒙教育"
date:       2020-04-12 
author:     "peter"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - 笔记
---

> This document is not completed and will be updated anytime.
<br> 建议阅读时间： 20分钟


## Linux 


> 纪念走进IT世界的第一扇门
<br> Linux is a Unix-like and mostly POSIX-compliant computer OS.

当年刚刚走出大学校门踏入社会，接触到的第一份工作就是IT届存储大厂的technical support。从未接触过操作系统的我开启了这扇通往技术道路的大门，也是为了纪念最初，把这篇开山博客献给我热爱的**linux系统**以及曾经工作中经常用到的工具，**正则表达式**。


## 正则表达式

既然是聊到这个话题，当然要说一下正则表达式的意义和用途。简单来讲，正则表达式就是用来用处字符串的方法，可以通过一些特殊符号的运用来快速的对文本文件进行增删改查的操作。一般来讲，任何一个有经验的运维人员或者是IT从业者，都会熟练的掌握正则表达式。

当初我从事第一份工作技术支持的时候，常常会和企业运维人员进行一些远程桌面解决一些程序错误问题。就在这一次次的远程环境中，我总能看到我的客户熟练的操作着他们的命令行，尤其是当我还停留在**ls**，**cat**，**df -h**这些基本操作的时候，我的客户却在熟练的运用正则表达式提取各种需要的文本字符串。以至于我心虚的不敢在客户桌面提取我想要的特殊字段，只能让客户把一些配置文件日志文件传给我，我偷偷的线下打开notepad++然后用原始的**CTRL + F**，**CTRL + C**以及**CTRL + V**的方法来拿到我想要的字符串。

这样的悲惨经历让我意识到了文本过滤的重要性，也正是如此让我对正则表达式产生了强烈的好奇心和求知欲。正则表达式实际上是一种“表示法”，只要某个工具程序支持这种表示法，就可以用正则表达式来进行字符串的处理。linux上最常见的就是grep，vi，sed，awk等处理文本的工具，其他一些比如cp，mv，ls之类的当然就是无法支持的。

- #### grep
grep是一个非常常用也非常简单的工具，最基础的就是从文本中抓取带有特殊字符串的某一行或几行。 
所以最基础的正则表达式练习可以结合grep开始，先随便找了一台ubuntu的虚机然后获取/etc/hosts配置文件，显示出具体的行数和断行字符。
```html
ubuntu@ip-10-0-0-144:~$ cat -An /etc/hosts
     1  127.0.0.1 localhost$
     2  $
     3  # The following lines are desirable for IPv6 capable hosts$
     4  ::1 ip6-localhost ip6-loopback$
     5  fe00::0 ip6-localnet$
     6  ff00::0 ip6-mcastprefix$
     7  ff02::1 ip6-allnodes$
     8  ff02::2 ip6-allrouters$
     9  ff02::3 ip6-allhosts$
```
然后执行几个命令看看能不能达到我们想要的效果获取特定的行。
```html
grep -n 'localhost' /etc/hosts     打印有localhost字符串的行
grep -n 'ff0[02]' /etc/hosts       打印有ff00或者ff02的行
grep -n 'f[^e]' /etc/hosts         打印有f后面不是e的字符串的行
grep -n '[^[:lower:]]' /etc/hosts  打印含有不是小写字母的字符串的行
grep -n '^$' /etc/hosts            打印出所有空白行
grep -n '^[^$]' /etc/hosts         打印出所有非空白行
```
需要注意的是* 的用法，代表的是重复0个或多个前面的RE字符。因此如果是“f* ”，那么有没有字符其实是都可以显示出来，如果后面再接一个字符的话，则只要含有后面那个字符就可以打印出来。看下面的例子可以作为证明。
```html
ubuntu@ip-10-0-0-144:~$ grep -n 'f*t' /etc/hosts
1:127.0.0.1 localhost
3:# The following lines are desirable for IPv6 capable hosts
4:::1 ip6-localhost ip6-loopback
5:fe00::0 ip6-localnet
6:ff00::0 ip6-mcastprefix
8:ff02::2 ip6-allrouters
9:ff02::3 ip6-allhosts
```

- #### sed工具
一旦开始使用了sed命令之后，我就再也不愿意使用cut这样的初级命令，因为cut的限制太多用起来太吃力，渐渐发现sed相比之下足够的强大并且最重要的是可以支持正则表达式的使用。
<br> 先来看一下sed工具的基本用法： sed [-nefri] [动作]
<br> -n 使用安静模式。正常模式中，所有数据都会被列出来，而加上-n之后就只会打印出经过sed处理过的那一行。
<br> -e 直接在命令行上进行文件编辑并打印出来。
<br> -f 可以将sed的动作写在一个文件内，后面跟这个文件就可以执行这个动作。
<br> -r 后面可以支持扩展的正则表达式，不仅限于基础正则表达式。
<br> -i 直接修改后面接的文件的内容而不只是由屏幕输出。（对于配置文件要慎用，建议还是先确认输出正确再做修改）
<br> 动作说明： '[n1[,n2]]function'
<br> 动作参数： a：新增到下一行 c：替换整行 d：删除 i:插入到上一行 p:打印，一般配合sed -n打印选择的数据 s：替换，配合正则表达式。

举例说明上述基本用法，可以自己编辑一个简单的文本然后执行下面的语句看看是否打印出正确的效果。
```html
$ cat test.test
test
123
tete
erererere
eeeeeeeee
fefsfddfd
xcvxvcvcxvxcv
fdsfsefefefsfefsefef
```
假设我们有上述一个名字叫test.test的文件，然后分别执行下面的语句。为了能打印出行号，可以使用nl代替cat。下面的sed直接接动作省略了-e。
```html
nl test.test | sed '2a test'           第二行下面一行新增一行test
nl test.test | sed '6,7c test'         第六第七行替换成一行test
nl test.test | sed '2,5d'              删除第二行到第五行，d后面如果接test会报错
nl test.test | sed '6,7i test'         第六行和第七行上面分别插入一行test
nl test.test | sed -n '6,7p'           只打印出6-7行，如果这时候不用-n的话，6-7行会反复输出，这就是安静模式的作用配合p的动作执行
```

- #### sed正则表达
为了演示一下sed上的正则表达式的用法，我特地找来了做support工作时候的一个日志文件，选取了一行比较复杂的日志来进行sed处理。
```html
$ cat test.pid
2019-10-24 08:23:22.562866 AEDT,"sys_gis","iadpprod",p766178,th-859531392,"xx.xxx.xxx.xxx","30904",2019-10-24 08:04:53 AEDT,0,con9683124,cmd16,seg-1,,dx23356519,,sx1,"ERROR","58M01","could not execute command on QE","FTS detected connection lost during dispatch to seg29 xxx.xxx.xx.xx:50005 pid=152199:","command: 'set gp_write_shared_snapshot=true'",,,,"SELECT  COUNT(*) FROM  PM_RECOVERY9 WHERE  REP_GID = $1  AND WFLOW_ID = $2  AND SUBJ_ID = $3  AND TASK_INST_ID = $4  AND TGT_INST_ID = $5",0,,"cdbdisp_query.c",550,
```
我如果想要提取FTS以后的字符串一直到包含pid=xxxxxx为止，并且希望整个日志文件其他每一行都要能提取出这个特殊字段，用sed是最佳的方法。如果用cut，我们无法把FTS作为cut的分隔符，也没有办法保证截取到pid=xxxxxx的字符串，因为其他行的pid会是别的号，所以我们就要用sed结合正则表达式来提取。
思路就是把FTS前面的文字全部替换成空格，再把pid=xxxxxx以后的文字也全部替换成空格，效果就出来了。
```html
$ cat test.pid | sed -r 's/.*FTS(.*)pid([^:]*).*/FTS\1pid\2/g'
FTS detected connection lost during dispatch to seg29 xxx.xxx.xx.xx:50005 pid=152199
```

- #### 正则表达式字符
通过上述的命令行来详细介绍一下几个特殊字符的作用。
```html
<br> ^word   待查找的字符串word在行首
<br> word$   待查找的字符串word在行尾
<br> .       代表一定有一个任意字符的字符
<br> *       重复0-无穷多个前一个字符 
<br> \       转义字符
<br> [abc]   字符集合中找出含a或b或c的字符串
<br> [n1-n2] 字符集合中找出想要选区的字符范围，连续性根据ASCII编码而定
<br> [^list] 字符集合中找出不是list的字符串
<br> \{n,m\} 连续n到m个前一个字符
```

所以在上述命令中 .* 代表任意字符重复多次，（.* ）代表第一个分组是FTS到pid字符串中的字符，([^:]* )代表第二个分组是pid之后，第一个：之前的字符串。然后我们讲整行内容替换成FTS+第一个分组内容+pid+第二个分组内容。显示出来的就是我们要的字段了。

其实上面的过滤方式还有另一种方法也可以做到
```html
$ cat test.pid | sed -r 's/^.*FTS/FTS/g' | sed -r 's/pid([^:]*).*$/pid\1/g'
FTS detected connection lost during dispatch to seg29 xxx.xxx.xx.xx:50005 pid=152199
```
通过两个管道分别替换两次，第一次替换FTS前面的部分变成FTS，第二次替换pid到：以后的部分变成pid+分组变量。


#### awk以及更多
其实linux还有更常用更实用的awk命令，awk工具在我之前的support生涯中为我带去很多的便利，利用awk我还写了一个小工具来快速查询我想要的特殊字段。这个就留着作为下一篇博客的题材，总结一下awk的使用方式和我的使用场景以及如何结合shell script写出一个简单实用的脚本供自己反复使用。





