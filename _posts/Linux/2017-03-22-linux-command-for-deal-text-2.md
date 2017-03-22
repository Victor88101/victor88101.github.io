---
layout: post
category: Linux
title: 《Linux常用文本处理命令（二）》
tagline: by Vv
tags: Linux 
---

## 目录
1.uniq

2.tr

3.cut

4.paste

5.wc

6.sed

## 内容

### 1.uniq
uniq命令用来过滤输入的相同邻近的数据行，并进行输出。

#### 1.1 uniq 命令常用参数项
```
uniq [OPTION]... [INPUT [OUTPUT]]
```

- -c, --count 在每行的最前列显示重复的次数
- -d, --repeated 只显示重复
- 行列
- -D, --all-repeated[=delimit-method] 打印所有重复的行
-   -f, --skip-fields=N  忽略比较第N列
-   -i, --ignore-case    忽略大小写
-   -s, --skip-chars=N   忽略比较第N个字符
-   -u, --unique         相同的列只打印一次
-   -z, --zero-terminated  end lines with 0 byte, not newline
-   -w, --check-chars=N   指定比较位置的字符
#### 1.2 uniq 命令常用事例

删除重复行： 
```
uniq file.txt
sort file.txt | uniq
sort -u file.txt
```

只显示单一行：
```
uniq -u file.txt 
sort file.txt | uniq -u 
```

统计各行在文件中出现的次数： 
```
sort file.txt | uniq -c 
```
在文件中找出重复的行： 
```
sort file.txt | uniq -d
```

### 2.tr

tr命令可以对来自标准输入的字符进行替换、压缩和删除。它可以将一组字符变成另一组字符，经常用来编写优美的单行命令，作用很强大。

#### 2.1 tr 常用语法项

```
tr [OPTION]... SET1 [SET2]
```

- -c --complerment：取第一字符集的补集； 
- -d --delete：删除所有属于第一字符集的字符
- -s --squeeze-repeats：把连续重复的字符以单独一个字符表示；
- -t --truncate-set1：删除第一字符集较第二字符集多出的字符。

#### 2.2 tr命令常用事例

将输入字符由大写转换为小写：
```
echo "HELLO WORLD" | tr 'A-Z' 'a-z' 

hello world 
```
>'A-Z' 和 'a-z'都是集合，集合是可以自己制定的，例如：'ABD-}'、'bB.,'、'a-de-h'、'a-c0-9'都属于集合，集合里可以使用'\n'、'\t'，可以可以使用其他ASCII字符。

使用tr删除字符： 
```
echo "hello 123 world 456" | tr -d '0-9'

hello world 
```

将制表符转换为空格： 
```
cat text | tr '\t' ' '
```

字符集补集，从输入文本中将不在补集中的所有字符删除： 
```
echo 'aa.,a 1 b#$bb 2 c*/cc 3 ddd 4' | tr -d -c '0-9 \n'

1 2 3 4 
```
> 此例中，补集中包含了数字0~9、空格和换行符\n，所以没有被删除，其他字符全部被删除了。 

用tr压缩字符，可以压缩输入中重复的字符： 
```
echo "thissss is a text linnnnnnne." | tr -s ' sn' 

this is a text line. 
```

巧妙使用tr做数字相加操作：
```
echo 1 2 3 4 5 6 7 8 9 | xargs -n1 | echo $[ $(tr '\n' '+') 0 ] 
```

删除Windows文件“造成”的'^M'字符： 
```
cat file | tr -s "\r" "\n" > new_file 
```
或 
```
cat file | tr -d "\r" > new_file
```


tr可以使用的字符类：

```
[:alnum:]：字母和数字 
[:alpha:]：字母 
[:cntrl:]：控制（非打印）字符
[:digit:]：数字 
[:graph:]：图形字符 
[:lower:]：小写字母 
[:print:]：可打印字符 
[:punct:]：标点符号 
[:space:]：空白字符 
[:upper:]：大写字母 
[:xdigit:]：十六进制字符 
```

使用方式： 
```
tr '[:lower:]' '[:upper:]'
```
### 3.cut
cut命令用来显示行中的指定部分，删除文件中指定字段。

#### 3.1 cut命令常用语法项
```
cut OPTION... [FILE]...
```
options:
- -b --bytes=LIST :仅显示行中指定直接范围的内容； 
- -c --characters=LIST :仅显示行中指定范围的字符； 
- -d  --delimiter=DELIM：指定字段的分隔符，默认的字段分隔符为“TAB”； 
- -f --fields=LIST：显示指定字段的内容； 
- -n：与“-b”选项连用，不分割多字节字符； 
- --complement：取被选择的字节、字符或字段的补集；
- -s, --only-delimited 不显示包含指定分隔符的行
- --out-delimiter=<字段分隔符>：指定输出内容是的字段分割符；

#### 3.2 cut命令常用实例

例如有一个学生报表信息，包含No、Name、Mark、Percent： 
```
cat test.txt
No Name Mark Percent 
01 tom 69 91
02 jack 71 87 
03 alex 68 98 
```

使用 -f 选项提取指定字段： 
```
cut -f 1 test.txt 

No 01 02 03 
```
```
cut -f2,3 test.txt

Name Mark 
tom 69 
jack 71 
alex 68 
```

--complement选项提取指定字段之外的列（打印除了第二列之外的列）：
```
cut -f2 --complement test.txt 

No Mark Percent
01 69 91
02 71 87
03 68 98 
```
使用 -d 选项指定字段分隔符： 
```
cat test2.txt

No;Name;Mark;Percent 
01;tom;69;91 
02;jack;71;87 
03;alex;68;98 
```
```
cut -f2 -d";" test2.txt 

Name
tom 
jack 
alex 
```
指定字段的字符或者字节范围 cut命令可以将一串字符作为列来显示，字符字段的记法：
```
N-：从第N个字节、字符、字段到结尾； 
N-M：从第N个字节、字符、字段到第M个（包括M在内）字节、字符、字段； 
-M：从第1个字节、字符、字段到第M个（包括M在内）字节、字符、字段。 
```
上面是记法，结合下面选项将摸个范围的字节、字符指定为字段： 
```
-b 表示字节； 
-c 表示字符； 
-f 表示定义字段。 
```
事例：
```
cat test.txt 
 
abcdefghijklmnopqrstuvwxyz 
abcdefghijklmnopqrstuvwxyz 
abcdefghijklmnopqrstuvwxyz 
abcdefghijklmnopqrstuvwxyz 
abcdefghijklmnopqrstuvwxyz 
```
 
 打印第1个到第3个字符： 
```
cut -c1-3 test.txt 
 
abc 
abc 
abc 
abc 
abc 
```
 打印前2个字符： 
```
cut -c-2 test.txt 
 
ab
ab
ab
ab 
ab 
```
 打印从第5个字符开始到结尾：
```
cut -c5- test.txt

efghijklmnopqrstuvwxyz 
efghijklmnopqrstuvwxyz 
efghijklmnopqrstuvwxyz 
efghijklmnopqrstuvwxyz 
efghijklmnopqrstuvwxyz

```
### 4.paste
paste命令用于将多个文件按照列队列进行合并。

#### 4.1 paste命令常用语法项

```
paste [OPTION]... [FILE]...
```

- -d --delimiters=LIST：用指定的字符取代TABS； 
- -s --serial:串列进行而非平行处理。

#### 4.2 paste命令常用事例

文件a.txt和b.txt 内容如下：

```
cat a.txt 

aaaa
bbbb
cccc

cat b.txt 

dddd
eeee
ffff
gggg
```
合并a.txt、b.txt：

```
paste a.txt b.txt 

aaaa    dddd
bbbb    eeee
cccc    ffff
        gggg
```

使用#替换TABS合并a.txt、b.txt：
```
paste -d '#' a.txt b.txt

aaaa#dddd
bbbb#eeee
cccc#ffff
#gggg
```
使用串行方式合并a.txt、b.txt：
```
paste -s a.txt b.txt 
aaaa    bbbb    cccc
dddd    eeee    ffff    gggg
```
使用#替换TABS,并使用串行方式合并a.txt、b.txt：
```
paste -s -d '#' a.txt b.txt 

aaaa#bbbb#cccc
dddd#eeee#ffff#gggg
```
### 5.wc
wc命令用来计算文件的行数、字节数、字符数
#### 5.1 wc命令的语法项：
```
wc [OPTION]... [FILE]...
```
OPTION：

- -c, --bytes 打印字节数
- -m, --chars 打印字符数
- -l, --lines 打印行数
- -L, --max-line-length 打印最长行的字符数
- -w, --words 打印字数

### 6.sed

#### 6.1 sed命令选项格式
```
sed [OPTION] [ACTION] file
```

选项与参数：
- -n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
- -e ：直接在命令列模式上进行 sed 的动作编辑；
- -f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
- -r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
- -i ：直接修改读取的文件内容，而不是输出到终端。

动作说明： [n1[,n2]]function

n1, n2 ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』

function：
- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个s的动作可以搭配正规表示法

#### 6.2 sed命令常用实例

##### 6.2.1 地址定界常规方法

1.空地址：即对全文进行处理
```
sed 's/root/ROOT/' /etc/passwd
```
2. 单地址：

> #: 指定行
```
sed -n '1,5{/^#/p}' fstab
sed '1,+5{/^#/d}' fstab
/pattern/ : 被模式匹配到的每一行
sed '/^root/p' /etc/passwd
```
3.地址范围

> #,# : 从#号行到#号行
```
sed '2,3d' /etc/fstab : 显示除2到3行的所有行
```
> #,+# : 从#号行向下#行
```
sed '2,+5d' /etc/fstab : 删除2到5行
```
> #,/pattern/ : 从#号行到被模式匹配到的行
```
sed '1,/^UUID/d' fstab1 :删除从第1行到被模式匹配到的第一个行的位置，删除
```
> /pattern1/,/pattern2/ : 从模式1匹配到的行到被模式2匹配到的行
```
sed -n '/^[/]/p' fstab1 : 显示为/开始的行
sed '/^#/d' fstab1 : 显示开始为#号的行
```
> $ : 表示最后一行
```
sed '$d' fstab1 : 删除最后一行
```
4.步进地址表示法：

> 1~2: 所有奇数行
```
sed -n '1~2p' fstab1
```
> 2~2: 所有偶数行
```
sed -n '2~2p' fatab
```
##### 6.2.2 sed编辑命令

> d : 删除模式空间中的内容

示例:
```
sed '1,5d' FILE : 删除1到5行的内容
sed '1~2d' FILE : 删除奇数行，只显示偶数行
```
> p : 显示被模式框定的内容

示例：
```
sed '1~2p' FILE : 显示奇数行，如果只需要显示一次，需要使用-n关闭默认模式空间的内容
sed -n '/./p' a.sh : 显示非空行，但对制表符无效
```
> P : 只显示模式空间中的第一行

示例：
```
seq 5 | sed -n 'N;P' : 显示结果为1、3两行
```
> a \line : 追加line行至匹配到行的后面，如果是多行可使用\n实现多行追加

示例：
```
sed '/^UUID/ a\line1\nline2' fstab :查找匹配到UUID开始的行，并在后面添加line1,line2两行内容
```
> i \line : 添加line行到匹配行的前面，如果是多行可使用\n实现多行添加

示例：
```
sed '/^UUID/i \line1\nline2' fstab :查找匹配到UUID开始的行，并在其前面增加line1,line2两行内容
```
> c \line : 把匹配到的行替换为line行

示例
```
sed '/^UUID/c \newline' fstab1 : 匹配以UUID开始的行，并把其替换为newline行
```
> w /PATH : 将模式空间匹配到的行，写入指定文件中

示例
```
sed '/#/!w ./w.txt' fstab : 匹配非#开始的行，并写入当前目录下的w.txt文件中
```
> r /PATH : 将PATH中指定的文件写入匹配到的行下方，多用于文件合并。

示例：
```
sed '/^UUID/r ./w.txt' fstab :把当前目录下的w.txt文件写入到以UUID开头的行下
```
> q : 退出sed,一般用于打印到第几行即退出

示例：
```
sed '10q' FILE : 只打印文件中的前10行，等同于sed -n '1,10p' FILE
```
> y : 完成大小写替换（等同于s///,基本不用）
```
sed '1,15y/UUID/uuid/' fstab1 :替换1到15行的内容UUID为uuid
sed 'y/UUID/uuid/' fstab1 : 替换全文每行中的第一个匹配到的
```
> = : 匹配到的行，显示一个行号，默认在其匹配到的行上方显示对应的行号，如果需要只显示行号，需要加-n参数，把模式空间中的内容关闭显示。

示例
```
sed '/^UUID/=' fstab : 在匹配到UUID开头的行上一行打印其行号
sed -n '$=' fstab : 显示最后一行的行号，一般可用于显示文本的总行数。
sed '/./=' File : 显示所有行的行号，但空行不显示行号
```
> ! : 条件取反，一般用于模式之后，命令之前

示例
```
sed '/^#/!d' FILE : 只显示非注释的行
```
> s/// : 字符替换查找，其分隔符可自动指定，常用的有,s@@@、s###。

替换标记操作符
> g : 全局替换，不加g只能每行开始的第一个匹配操作
如果只想从第几次开始替换，可使用3g即Ng(N代表一个数值)
> w /PATH: 将替换成功的结果保存至指定文件中
> p : 显示替换成功的行
示例
```
sed 's/UUID/uuid/' fstab : 将UUID替换为uuid
sed 's/love/& you' FILE: 将love替换为love you,&`表示对前面模式的引用
sed 's/^\(UUID\).*/\1 Hello/' fstab1 : 将UUID开头的行替换为UUID Hello的内容
sed -n '1,15s/^UUID/uuid/gp' fstab : 查找1到15行以UUID开始的行，并将其替换为uuid，并且只显示被替换过的行
sed 's/.$//' File 将每行中最后一个字符删除，.$代表每行的最后一个字符
```
> h : 把模式空间中的内容覆盖至保持空间中

> H : 把模式空间中的内容追加至保持空间中

> g : 把保持空间中的内容覆盖至模式空间中

> G : 把保持空间中的内容追加至模式空间中

> x : 把模式空间中的内容到保持空间中的内容互换，初始保持空间中为空

> n : 读取下一行覆盖模式空间中的行
```
seq 11 | sed 'n;d' : 显示结果为1、3、5、7、9、11 ，默认动作先输出模式空间中的行，再覆盖读取下一行，再执行d命令
seq 10 | sed 'n;d' : 显示结果为1、3、5、7、9
```
> N : 读取下一行并追加到模式空间中的行后面，使用\n分隔
```
seq 11 | sed 'N;d' : 显示结果为11，默认动作先读取两行，然后执行d操作
seq 10 | sed 'N;d' :显示结果为空
```
> D : 删除模式空间中的多行
```
seq 11 | sed 'N;D' : 显示结果为11
```
> {} : 多命令同时执行时，需要使用{}括起来
```
sed -n '/^UUID/{N;p}' fstab1 : 读取UUID开始的行，再读取下一行并打印模式空间的内容。
```