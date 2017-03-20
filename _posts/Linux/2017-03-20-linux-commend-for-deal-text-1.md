
---
layout: post
category: Linux
title: 《Linux常用文本处理命令整理（一）》
tagline: by Vv
tags: Linux 
---


## 目录

> 1.find
>
> 2.grep
>
> 3.xargs
>
> 4.sort

## 内容

### 1.find

#### 1.1 find命令的一般形式

- path :find命令所查找的目录路径
- expression：expression可以分为```-options [-print -exec -ok ...]```
- -options，指定find命令的常用选项，下节详细介绍
- -print，find命令将匹配的文件输出到标准输出
- -exec，find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {} \;，注意{}和\;之间的空格 
```
find ./ -size 0 -exec rm {} \; #删除文件大小为零的文件 
```
（还可以以这样做：``` rm -i `find ./ -size 0` ```或``` find ./ -size 0 | xargs rm -f & ```）

为了用ls -l命令列出所匹配到的文件，可以把ls -l命令放在find命令的-exec选项中：\
``` 
find . -type f -exec ls -l {} \;
```

在/logs目录中查找更改时间在5日以前的文件并删除它们：
```
find /logs -type f -mtime +5 -exec rm {} \;
```

- -ok，和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。 

在当前目录中查找所有文件名以.conf结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示
```
find . -name "*.conf"  -mtime +5 -ok rm {} \; 
```

> 也有人这样总结find命令的结构：
> 

```
find start_directory test 
      options 
      criteria_to_match 
      action_to_perform_on_results
```
#### 1.2 find命令的一般形式

- -name 按照文件名查找文件。 

在/dir目录及其子目录下面查找名字为filename的文件:
```
find /dir -name filename 
```
在当前目录及其子目录（用“.”表示）中查找任何扩展名为“c”的文件:
```
find . -name "*.c" 
```
- -perm 按照文件权限来查找文件。 

在当前目录下查找文件权限位为755的文件，即文件属主可以读、写、执行，其他用户可以读、执行的文件:
```
find . -perm 755 –print
```
- -prune 使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略。 

在/apps目录下查找文件，但不希望在/apps/bin目录下查找:
```
find /apps -path "/apps/bin" -prune -o –print 
```
在/usr/sam目录下查找不在dir1子目录之内的所有文件:
```
find /usr/sam -path "/usr/sam/dir1" -prune -o –print 
```

- -user 按照文件属主来查找文件。 

在$HOME目录中查找文件属主为sam的文件:
```
find ~ -user sam –print 
```

- -group 按照文件所属的组来查找文件。 
在/apps目录下查找属于gem用户组的文件:
```
find /apps -group gem –print
```
- -mtime -n +n 按照文件的更改时间来查找文件， - n表示文件更改时间距现在n天以内，+ n表示文件更改时间距现在n天以前。 

在系统根目录下查找更改时间在5日以内的文件:
```
find / -mtime -5 –print 
```
在/var/adm目录下查找更改时间在3日以前的文件:
```
find /var/adm -mtime +3 –print
```

- -nogroup 查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在。

```
find / –nogroup -print
```

- -nouser 查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在。
```
find /home -nouser –print
```
- -newer file1 ! file2 查找更改时间比文件file1新但比文件file2旧的文件。

- -type 查找某一类型的文件，诸如： 
    - b - 块设备文件。 
    - d - 目录。 
    - c - 字符设备文件。 
    - p - 管道文件。 
    - l - 符号链接文件。 
     -f - 普通文件。

在/etc目录下查找所有的目录:
```
find /etc -type d –print 
```
在当前目录下查找除目录以外的所有类型的文件:
```
find . ! -type d –print
```
在/etc目录下查找所有的符号链接文件:
```
find /etc -type l –print 
```

- -size n：[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。 

在当前目录下查找文件长度大于1 M字节的文件:
```
find . -size +1000000c –print 
```
在/home/apache目录下查找文件长度恰好为100字节的文件 
```
find /home/apache -size 100c –print
```
在当前目录下查找长度超过10块的文件（一块等于512字节:
```
find . -size +10 –print
```

- -depth：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。 

它将首先匹配所有的文件然后再进入子目录中查找：
```
find / -name "CON.FILE" -depth –print
```
- -mount：在查找文件时不跨越文件系统mount点。

从当前目录开始查找位于本文件系统中文件名以XC结尾的文件（不进入其他文件系统）:
```
find . -name "*.XC" -mount –print 
```

- -follow：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。

#### 1.3 find与xargs

> 在使用find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。

> find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。

> 在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高；

> 而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。

来看看xargs命令是如何同find命令一起使用的，并给出一些例子。

查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件:
```
find . -type f -print | xargs file 
```
在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中：
```
find / -name "core" -print | xargs echo "" >/tmp/core.log 
```
用grep命令在所有的普通文件中搜索hostname这个词:
```
find . -type f -print | xargs grep "hostname" 
```
删除3天以前的所有东西 （```find . -ctime +3 -exec rm -rf {} \;```）:
```
find ./ -mtime +3 -print|xargs rm -f –r 
```
删除文件大小为零的文件:
```
find ./ -size 0 | xargs rm -f & 
```

*find命令配合使用exec和xargs可以使用户对所匹配到的文件执行几乎所有的命令。*


### 2.grep

>grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

#### 2.1 grep命令的一般选项及实例

grep [OPTIONS] PATTERN [FILE...] 

grep [OPTIONS] [-e PATTERN | -f FILE] [FILE...]

> grep命令用于搜索由Pattern参数指定的模式，并将每个匹配的行写入标准输出中。这些模式是具有限定的正则表达式，它们使用ed或egrep命令样式。如果在File参数中指定了多个名称，grep命令将显示包含匹配行的文件的名称。
>
> 对 shell 有特殊含义的字符 ($, *, [, |, ^, (, ), \ ) 出现在 Pattern参数中时必须带双引号。如果 Pattern参数不是简单字符串，通常必须用单引号将整个模式括起来。在诸如 [a-z], 之类的表达式中，-（减号）cml 可根据当前正在整理的序列来指定一个范围。整理序列可以定义等价的类以供在字符范围中使用。如果未指定任何文件，grep会假定为标准输入。

#### 2.2 grep正则表达式元字符集(基本集)

字符 | 描述
--- | ---
^ | 锚定行的开始 如：'^grep'匹配所有以grep开头的行。
$ | 锚定行的结束 如：'grep$'匹配所有以grep结尾的行。
. | 匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。
* | 匹配零个或多个先前字符 如：' *grep'匹配所有一个或多个空格后紧跟grep的行。 .*一起用代表任意字符。
[] | 匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。
[^] | 匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-F和H-Z的一个字母开头，紧跟rep的行。
\(..\) | 标记匹配字符，如：'\(love\)'，love被标记为1。
\< | 锚定单词的开始，如：'\<grep'匹配包含以grep开头的单词的行。
\> | 锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。
x\{m\} | 连续重复字符x，m次，如：'o\{5\}'匹配包含连续5个o的行。
x\{m,\} | 连续重复字符x,至少m次，如：'o\{5,\}'匹配至少连续有5个o的行。
x\{m,n\} | 连续重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配连续5--10个o的行。
\w | 匹配一个文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。
\W | w的反置形式，匹配一个非单词字符，如点号句号等。\W*则可匹配多个。
\b | 单词锁定符，如: '\bgrep\b'只匹配grep，即只能是grep这个单词，两边均为空格。

#### 2.3 grep命令的常用选项及实例

字符 | 描述
--- | ---
-? | 同时显示匹配行上下的？行，如：grep -2 pattern filename同时显示匹配行的上下2行。
-b，--byte-offset | 打印匹配行前面打印该行所在的块号码。
-c,--count | 只打印匹配的行数，不显示匹配的内容。
-f File，--file=File | 从文件中提取模板。空文件中包含0个模板，所以什么都不匹配。
-h，--no-filename | 当搜索多个文件时，不显示匹配文件名前缀。
-i，--ignore-case | 忽略大小写差别。
-q，--quiet | 取消显示，只返回退出状态。0则表示找到了匹配的行。
-l，--files-with-matches | 打印匹配模板的文件清单。
-L，--files-without-match | 打印不匹配模板的文件清单。
-n，--line-number | 在匹配的行前面打印行号。
-s，--silent | 不显示关于不存在或者无法读取文件的错误信息。
-v，--revert-match | 反检索，只显示不匹配的行。
-w，--word-regexp | 如果被\<和\>引用，就把表达式做为一个单词搜索。
-V，--version | 显示软件版本信息。

---
事例：

```
ls -l | grep '^a' 
```
通过管道过滤ls -l输出的内容，只显示以a开头的行。

```
grep 'test' d* 
```
显示所有以d开头的文件中包含test的行。

```
grep 'test' aa bb cc 
```
显示在aa，bb，cc文件中匹配test的行。

```
grep '[a-z]' aa
```
显示所有包含每个字符串至少有5个连续小写字符的字符串的行。

```
grep 'w(es)t.*' aa 
```
如果west被匹配，则es就被存储到内存中，并标记为1，然后搜索任意个字符(.*)，这些字符后面紧跟着另外一个es()，找到就显示该行。如果用egrep或grep -E，就不用""号进行转义，直接写成'w(es)t.*'就可以了。

```
grep -i pattern files 
```
不区分大小写地搜索。默认情况区分大小写

```
grep -l pattern files 
```
只列出匹配的文件名，

```
grep -L pattern files 
```
列出不匹配的文件名，

```
grep -w pattern files
```
只匹配整个单词，而不是字符串的一部分(如匹配‘magic’，而不是‘magical’)，

```
grep -C number pattern files 
```
匹配的上下文分别显示[number]行，

```
grep pattern1 | pattern2 files 
```
显示匹配 pattern1 或 pattern2 的行，

```
grep pattern1 files | grep pattern2 
```
显示既匹配 pattern1 又匹配 pattern2 的行。

### 3.xargs

> 在使用 find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。  
>
> find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。  
>
> 在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高； 而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。

#### 3.1 xargs 命令的一般选项

```
xargs [-0prtx] 
    [-E eof-str] [-e[eof-str]] [--eof[=eof-str]] [--null]
    [-d delimiter] [--delimiter delimiter]
    [-I replace-str] [-i[replace-str]] [--replace[=replace-str]] 
    [-l[max-lines]] [-L max-lines][--max-lines[=max-lines]] 
    [-n max-args] [--max-args=max-args] 
    [-s max-chars] [--max-chars=max-chars] 
    [-P max-procs] [--max-procs=max-procs] [--process-slot-var=name] [--interactive] 
    [--verbose] [--exit][--no-run-if-empty] 
    [--arg-file=file] [--show-limits] [--version][--help] 
    [command [initial-arguments]]
```
- -0 作为分隔符，配合find命令的-print0参数使用
- -E 设置命令行结束字符
- -e 同-E
- -I 设置替换。默认为{}
- -i 同-I
- -L 最大行数。默认为1
- -l 同-L
- -n 每行允许的最大参数个数
- -s 每行允许的最大字符数
- -P 每次允许执行的最大进程数
- -p 采用交互式来执行每个命令行
- -a 从文件读取数据。

#### 3.2 xargs命令使用实例：

查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件:
```
find . -type f -print | xargs file
```
在整个系统中查找内存信息转储文件(core dump)，然后把结果保存到/tmp/core.log 文件中:
```
 find / -name "core" -print | xargs echo "" >/tmp/core.log
```

在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限:
```
find . -perm -7 -print | xargs chmod o-w
```

用grep命令在所有的普通文件中搜索hostname这个词:
```
find . -type f -print | xargs grep "hostname"
```

用grep命令在当前目录下的所有普通文件中搜索hostnames这个词:
```
find . -name \* -type f -print | xargs grep "hostnames"
```
说明：
注意，在上面的例子中， \用来取消find命令中的*在shell中的特殊含义。  


使用xargs执行mv:
```
find . -name "*.log" | xargs -i mv {} test4
```

find后执行xargs提示xargs: argument line too long解决方法：
```
find . -type f -atime +0 -print0 | xargs -0 -l1 -t rm -f
```
说明：
-l1是一次处理一个；-t是处理之前打印出命令
 
使用-i参数默认的前面输出用{}代替，-I参数可以指定其他代替字符，如例子中的[]: 
```
find . -name "file" | xargs -I [] cp [] ..
```

xargs的-p参数的使用:
```
find . -name "*.log" | xargs -p -i mv {} ..
```
说明：
-p参数会提示让你确认是否执行后面的命令,y执行，n不执行。


xargs用作替换工具，读取输入数据重新格式化后输出。

定义一个测试文件，内有多行文本数据： 
```
cat test.txt 
    a b c d e f g
    h i j k l m n o p q 
    r s t u v w x y z 
```
多行输入单行输出： 
```
cat test.txt | xargs

a b c d e f g h i j k l m n o p q r s t u v w x y z 
```
-n选项多行输出： 
```
cat test.txt | xargs -n3

a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z 
```
-d选项可以自定义一个定界符： 
```
echo "nameXnameXnameXname" | xargs -dX 

name name name name
```

结合-n选项使用： 

```
echo "nameXnameXnameXname" | xargs -dX -n2 

name name
name name
```

### 4.sort
sort命令用来对文本进行安行排序
#### 4.1 sort 命令的一般选项
    sort [OPTION]... [FILE]...
option
- -b：忽略每行前面开始出的空格字符；
- -c：检查文件是否已经按照顺序排序；
- -d：排序时，处理英文字母、数字及空格字符外，忽略其他的字符；
- -f：排序时，将小写字母视为大写字母； 
- -i：排序时，除了040至176之间的ASCII字符外，忽略其他的字符；
- -m：将几个排序号的文件进行合并； 
- -M：将前面3个字母依照月份的缩写进行排序； 
- -n：依照数值的大小排序； 
- -o<输出文件>：将排序后的结果存入制定的文件； 
- -r：以相反的顺序来排序； 
- -R：随机排序
- -t<分隔字符>：指定排序时所用的栏位分隔字符； 
- -u：忽略相同行
- +<起始栏位>-<结束栏位>：以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。

#### 4.2 sort命令常用事例

事例文本内容：
```
cat sort.txt 

aaa:10:1.1
ccc:30:3.3 
ddd:40:4.4
bbb:20:2.2 
eee:50:5.5
eee:50:5.5 
```
默认排序：

```
sort sort.txt 

aaa:10:1.1
bbb:20:2.2 
ccc:30:3.3 
ddd:40:4.4
eee:50:5.5 
eee:50:5.5
```
忽略相同行使用-u选项或者uniq：
```
sort -u sort.txt 

aaa:10:1.1 
bbb:20:2.2
ccc:30:3.3
ddd:40:4.4 
eee:50:5.5
```
或者
```
uniq sort.txt

aaa:10:1.1
ccc:30:3.3 
ddd:40:4.4
bbb:20:2.2 
eee:50:5.5
```
sort的-n、-r、-k、-t选项的使用： 
```
cat sort.txt 

AAA:BB:CC
aaa:30:1.6 
ccc:50:3.3
ddd:20:4.2 
bbb:10:2.5
eee:40:5.4
eee:60:5.1
```
将BB列按照数字从小到大顺序排列：
```
sort -nk 2 -t: sort.txt 

AAA:BB:CC
bbb:10:2.5
ddd:20:4.2
aaa:30:1.6
eee:40:5.4
ccc:50:3.3
eee:60:5.1 
```

将CC列数字从大到小顺序排列： 
```
sort -nrk 3 -t: sort.txt 

eee:40:5.4 
eee:60:5.1
ddd:20:4.2
ccc:50:3.3 
bbb:10:2.5
aaa:30:1.6
AAA:BB:CC 
```
> -n是按照数字大小排序，-r是以相反顺序，-k是指定需要爱排序的栏位，-t指定栏位分隔符为冒号 

-k选项的具体语法格式：

-k选项的语法格式：
```
FStart.CStart Modifie,FEnd.CEnd Modifier
-------Start--------,-------End-------- 
FStart.CStart 选项 , FEnd.CEnd 选项
```
> 这个语法格式可以被其中的逗号,分为两大部分，Start部分和End部分。Start部分也由三部分组成，其中的Modifier部分就是我们之前说过的类似n和r的选项部分。

> 我们重点说说Start部分的FStart和CStart。CStart也是可以省略的，省略的话就表示从本域的开头部分开始。
FStart.CStart，其中FStart就是表示使用的域，而CStart则表示在FStart域中从第几个字符开始算“排序首字符”。

> 同理，在End部分中，你可以设定FEnd.CEnd，如果你省略.CEnd，则表示结尾到“域尾”，即本域的最后一个字符。或者，如果你将CEnd设定为0(零)，也是表示结尾到“域尾”。

从公司英文名称的第二个字母开始进行排序： 
```
$ sort -t ' ' -k 1.2 facebook.txt 

baidu 100 5000 
sohu 100 4500 
google 110 5000 
guge 50 3000 
```
> 使用了-k 1.2，表示对第一个域的第二个字符开始到本域的最后一个字符为止的字符串进行排序。你会发现baidu因为第二个字母是a而名列榜首。sohu和google第二个字符都是o，但sohu的h在google的o前面，所以两者分别排在第二和第三。guge只能屈居第四了。

只针对公司英文名称的第二个字母进行排序，如果相同的按照员工工资进行降序排序： 
```
$ sort -t ' ' -k 1.2,1.2 -nrk 3,3 facebook.txt 

baidu 100 5000 
google 110 5000 
sohu 100 4500 
guge 50 3000 
```
> 由于只对第二个字母进行排序，所以我们使用了-k 1.2,1.2的表示方式，表示我们“只”对第二个字母进行排序。（如果你问“我使用-k 1.2怎么不行？”，当然不行，因为你省略了End部分，这就意味着你将对从第二个字母起到本域最后一个字符为止的字符串进行排序）。对于员工工资进行排 序，我们也使用了-k 3,3，这是最准确的表述，表示我们“只”对本域进行排序，因为如果你省略了后面的3，就变成了我们“对第3个域开始到最后一个域位置的内容进行排序” 了。
