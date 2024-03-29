[TOC]







# 第十一单元-awk文本处理工具

## 11.1 awk高级应用

### 11.1.1 awk命令详解

#### 1. 简介

awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，**以空格为默认分隔符将每行切片**，切开的部分再进行各种分析处理。

awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： **AWK 程序设计语言** ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。



快速掌握awk的技巧：只要记住awk是**以行为单位读入和输出的**。

**使用方法**

```
awk '{pattern + action}' filename
```



#### 2. awk常用选项和命令

| 参数  | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| -F    | 指定字段一个或多个分割符 例如:-F'[:#/]'   定义三个分隔符     |
| -v    | 定义或修改一个awk内部的变量                                  |
| NR    | 行号                                                         |
| FS    | 字段的分隔符，默认为空格，跟-F选择一样                       |
| OFS   | 输出的字段分隔符，默认为空格(即把空格替换成指定的字符串)     |
| RS    | 输入记录的分割，以分割符分割之后，使之成为新的行（即读入行的时候遇到指定分隔符，就把分割替换成\n）。 |
| ORS   | 输出的记录分隔符，默认为新行。（即读入行的时候遇到\n之后把\n用指定的分隔符代替，然后读入一行，并合并为同一行） |
| $NF   | 表示最后一列                                                 |
| $0    | 显示当前一整行                                               |
| $N    | N为数字（N>0），表示第几列。例如：$1表示用-F指定分隔符分隔后的第一列，$2...$N以此类推 |
| {}    | 命令代码块，包含一条或多条命令                               |
| ;     | 多条命令使用分号分隔                                         |
| ~     | 匹配字段,与==相比不是精确比较                                |
| !~    | 不匹配，不精确比较                                           |
| ==    | 等于，必须全部相等，精确比较                                 |
| >     | 大于                                                         |
| <     | 小于                                                         |
| >=    | 大于等于                                                     |
| <=    | 小于等于                                                     |
| !=    | 不等于，精确比较                                             |
| &&    | 逻辑与                                                       |
| \|\|  | 逻辑或                                                       |
| +     | 匹配1个或1个以上                                             |
| //    | 正则匹配符                                                   |
| print | 输出、打印                                                   |



#### 3. awk的选项

| 选项                                     | 描述                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| -f progfile     --file=progfile          | 从脚本文件中读取awk命令                                      |
| -F fs             --field-separator=fs   | 指定字段一个或多个分割符 例如:-F'[:#/]'   定义三个分隔符     |
| -v var=val     --assign var=val          | 定义或修改一个awk内部的变量                                  |
| -b                 --characters-as-bytes | 将所有输入数据视为单字节字符。posix选项或覆盖这个选项        |
| -c       --traditional                   | 在兼容模式下运行。在兼容模式下,gawk的行为与UNIX awk相同;没有一个可以识别特定于gn的扩展 |
| -C      --copyright                      | 在标准输出和退出中成功打印GNU版权信息消息的简短版本          |
| -d[file]--dump-variables[=file]          | 将全局变量的排序列表、它们的类型和最终值打印到文件中。如果没有提供文件，gawk使用一个名为awkvars的文件 |
| -e 'program-text'--source='program-text' | 使用程序文本作为AWK程序源代码。这个选项允许轻松地将库函数(通过-f和-file选项使用)与在命令行中输入的源代码混合使用。它主要用于shell脚本中使用的中型到大型AWK程序 |
| -E file    --exec=file                   | 与-f类似，这是最后一个处理的选项。这应该与#一起使用!脚本，特别是为CGI应用程序，以避免从URL向命令行传递选项或源代码(!)。这个选项禁止命令行变量赋值 |
| -g                 --gen-pot             | 扫描和解析AWK程序，并在标准输出上生成GNU .pot(可移植对象模板)格式文件，其中包含程序中所有可本地化字符串的条目。程序本身没有执行。 |
| -h                 --help                | 简短的打印帮助                                               |
| -L [fatal]       --lint[=fatal]          | 提供关于可疑或不可移植到其他AWK实现的构造的警告              |
| -n           --non-decimal-data          | 识别输入数据中的八进制和十六进制值                           |
| -N         --use-lc-numeric              | 使用句点作为小数点                                           |
| -O                --optimize             | 在程序的内部表示上启用优化。目前，这包括简单的常数合并       |
| -p[file]           --profile[=file]      | 将分析数据发送到PROFIX文件。默认值是DouthPo.OUT              |
| -P   --posix                             | 启动兼容模式;将有如下限制： <1>无法识别\x <2>当FS被设置为单个空间时，只有空格和Tab充当字段分隔符，换行符不被设置为分隔符 <3>之后行不能有?和: <4>关键字函数的同义词FUNC不被识别 <5>运算符**和**=不能代替^和^= |
| -r     --re-interval                     | 允许在正则表达式匹配中使用区间表达式                         |
| -R    --command file                     | 只有DGAWK。从文件读取存储的调试器命令                        |
| -S    --sandbox                          | 在沙盒模式下运行GOWK，禁用Stand（）函数，用GETLIN输入重定向，输出Read打印和打印的方向，并加载动态扩展。命令执行（通过管道)也被禁用。这有效地阻止了脚本访问本地资源（除了 命令行中指定的文件） |
| -t     --lint-old                        | 提供对UNIX AWK原始版本不可移植的结构的警告                   |
| -V    --version                          | 打印AWK的版本信息                                            |



#### 4. awk的环境变量

| 变量        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| **$N**      | **N为数字(N>0)，表示第几列。例如：$1表示用-F指定分隔符分隔后的第一列，$2...$N以此类推** |
| **$0**      | **显示当前一整行(有多少行显示多少行)**                       |
| **$NF**     | **表示最后一列**    **$(NF-1)是倒数第二列**                  |
| ARGC        | 命令行参数的数目                                             |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                            |
| ARGV        | 包含命令行参数的数组                                         |
| CONVFMT     | 数字转换格式(默认值为%.6g)                                   |
| ENVIRON     | 环境变量关联数组                                             |
| ERRNO       | 最后一个系统错误的描述                                       |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                                   |
| FILENAME    | 当前文件名                                                   |
| **NR**      | **行号(当前记录数)**                                         |
| **FNR**     | **同NR，但相对于当前文件**                                   |
| **FS**      | **字段的分隔符，默认为空格，跟-F选择一样**                   |
| **OFS**     | **输出的字段分隔符，默认为空格(即把空格替换成指定的字符串)** |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配                             |
| **NF**      | **当前记录中的字段数**                                       |
| OFMT        | 数字的输出格式(默认值是%.6g)                                 |
| RLENGTH     | 由match函数所匹配的字符串的长度                              |
| RS          | 输入记录的分割，以分割符分割之后，使之成为新的行（即读入行的时候遇到指定分隔符，就把分割替换成\n(默认是\n)） |
| ORS         | 输出的记录分隔符，默认为新行。（即读入行的时候遇到\n之后把\n用指定的分隔符代替，然后读入一行，并合并为同一行）(默认值是一个换行符) |
| RSTART      | 由match函数所匹配的字符串的第一个位置                        |
| SUBSEP      | 数组下标分隔符(默认值是\034)                                 |



#### 5. awk的运算符

| 运算符                  | 描述                             |
| ----------------------- | -------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| 条件表达式?值1:值2      | 三目运算符                       |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ ~!                    | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接符                           |
| + -                     | 加，减                           |
| * / %                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^   ***                 | 幂运算符                         |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |



#### 6. awk的内置的字符串函数

| 函数名称        | 描述                                  |
| --------------- | ------------------------------------- |
| gsub(r,s)       | 在整个$0中用s代替r                    |
| gsub(r,s,t)     | 在整个t中用s替代r                     |
| index(s,t)      | 返回s中字符串t的第一位置              |
| length(s)       | 返回s长度                             |
| match(s,r)      | 测试s是否包含匹配r的字符串            |
| split(s,a,fs)   | 在fs上将s分成序列a                    |
| sprint(fmt,exp) | 返回经fmt格式化后的exp                |
| sub(r,s)        | 用$0中最左边最长的子串代替s           |
| substr(s,p)     | 返回字符串s中从p开始的后缀部分        |
| substr(s,p,n)   | 返回字符串s中从p开始长度为n的后缀部分 |



### 11.1.2 awk中各种模式详解



#### **1. awk 脚本拥有的形式**

```shell 
awk  '/pattern/ { actions }' filename
```

　　你通常会发现脚本中的模式(/pattern/)是一个正则表达式，此外，你也可以在这里用特殊模式 BEGIN 和 END。因此，我们也能按照下面的形式编写一条 awk 命令：

```shell
awk 'BEGIN { actions } /pattern/ { actions } /pattern/ { actions }...END { actions }' filenames
```

假如你在 awk 脚本中使用了特殊模式：BEGIN 和 END，以下则是它们对应的含义：

BEGIN 模式：是指 awk 将在读取任何输入行之前立即执行 BEGIN 中指定的动作。

END 模式：是指 awk 将在它正式退出前执行 END 中指定的动作。

awk通过Pattern（模式）来控制是否处理当前记录，如果当前记录和Pattern匹配，则执行Action（操作）。在awk中，有下列几种模式：

```shell
1、正则表达式
2、关系表达式
3、组合的Pattern
4、Pattern1,Pattern2
5、BEGIN
6、END
```

为了说明以上各种模式，我们这里准备一个文件score.txt，以实例的方式一一进行说明，score.txt文件内容如下：

```shell
zhaosan 85 92 78
lisheng 89 90 75
zhaoyun 84 88 80
guanyu 83 78 90
liubei 86 88 79
```

这是一个比较简单的学生成绩表，共有5个学生的成绩，每条记录包含名字，其后跟的是三门课的成绩，分别算是语文、数学和英语的成绩吧。



#### 2. 正则表达式

```shell
awk '/9[0-9]/ {print $0}' score.txt

zhaosan 85 92 78
lisheng 89 90 75
guanyu 83 78 90
```

　　以上指令查询有一门课成绩在`[90-99]`区间的学生的成绩信息，`/9[0-9]/`部分即为**awk**程序指令中的**Pattern**，这里Pattern的类型为正则表达式。 

```shell
awk '$3 ~ /9[0-9]/ {print $0}' score.txt

zhaosan 85 92 78
lisheng 89 90 75
```

　　这条指令在上一条指令的基础上增加了限制，需要第二门课（数学）成绩在`[90-99]`区间才可与模式匹配。这里的 **~** 操作符用来表示变量是否与正则表达式匹配，如果要判断不匹配，可以使用 **!~** 操作符。



#### **3. 关系表达式**

```shell
awk '$3 >= 90 {print $0}' score.txt

zhaosan 85 92 78
lisheng 89 90 75
```

　　可用来形成模式关系运算符包括： <（小于）、>（大于）、<=（小于或等于）、>=（大于或等于）、= =（等于）和 ! =（不等于）。

　　这条指令的作用也是查询数学成绩在90分以上的学生成绩信息，不过比正则表达式中的范围要大一点，这里100分也是符合模式的。



#### 4. 组合的Pattern（模式）

```shell
awk '$3 >= 90 && $3 < 100 {print $0}' score.txt

zhaosan 85 92 78
lisheng 89 90 75
```

　　布尔运算符 ||（或）&&（和）以及 !（不）将模式组合，组合后如果求值为真则模式匹配，否则不匹配。这里就解决了关系表达式示例中包含了100的问题。



#### 5. Pattern1,Pattern2

```shell
awk 'FNR == 2 , FNR == 4 {print $0}' score.txt

lisheng 89 90 75
zhaoyun 84 88 80
guanyu 83 78 90
```

　　其实这个也可以归为组合的模式中，只是这种模式比较特殊，故单独列出。以,（逗号）隔开的两个Pattern指定一个范围，对从匹配第一个Pattern的记录开始，到匹配第二个Pattern结束的所有记录执行Action。



#### 6. BEGIN

​	BEGIN模块在awk读取文件之前就执行，一般用来定义我们的内置变量（预定义变量，eg：FS,RS）可以输出表头（excel表格名称）

BEGIN模式之前在实例中提到，自定义变量，给内容变量赋值等，都是使用过。需要注意的是BEGIN模式后面要结合一个**action操作块，包含在大括号内**。

awk必须在对输入文件进行任何处理前先执行BEGIN定义的action操作块。我们可以不要任何输入文件，就可以对BEGIN模块进行测试，因为awk需要先执行完BEGIN模式，才对输入文件做处理。BEGIN模式常常被用来修改内置变量ORS，RS，FS，OFS，等的值。

**示例一：给文件开头添加信息**

　　假如我们要将学生成绩表打印出来，那总得加点表头什么的吧，就可以放到BEGIN中了。

```shell
awk 'BEGIN { print "Print student score table"} {print $0}' score.txt

Print student score table
zhaosan 85 92 78
lisheng 89 90 75
zhaoyun 84 88 80
guanyu 83 78 90
liubei 86 88 79
```

**示例二：取eth0的ip地址**

```shell
ifconfig eth0|awk -F '(addr:)|(Bcast:)' 'NR==2{print $2}'
ifconfig eth0|awk -F '[: ]+' 'NR==2{print $4}'
ifconfig eth0|awk -F '[^0-9.]+' 'NR==2{print $2}'
```

也可以写成：

```shell
ifconfig eth0|awk  'BEGIN{FS="(addr:)|(Bcast:)"} NR==2{print $2}'
ifconfig eth0|awk  'BEGIN{FS="[^0-9.]+"} NR==2{print $2}'
ifconfig eth0|awk  'BEGIN{FS="[: ]+"} NR==2{print $4}'
```

注意：命令行 -F 本质就是修改FS的变量。



#### 7. END

​	END 在awk读取完所有的文件的时候，再执行END模块，一般用来输出一个结果（累加，数组的结果）也可以是和BEGIN模块类似的结尾标识信息。

**示例一：给文件结尾添加信息**

```shell
awk 'END { print "Work done"} {print $0}' score.txt

zhaosan 85 92 78
lisheng 89 90 75
zhaoyun 84 88 80
guanyu 83 78 90
liubei 86 88 79
Work done
```



**示例二：统计/etc/services文件中的空行数量**

统计数量:  grep  -c  或 awk

```shell
[root@mysql-master ~]# awk '/^$/{print $0}' /etc/services |wc -l
16

[root@mysql-master ~]# grep -c '^$' /etc/services
16

[root@mysql-master ~]# awk '/^$/{i++}END{print i}' /etc/services
16
[root@mysql-master ~]# awk '/^$/{i=i+1}END{print i}' /etc/services
16
```

**示例三：显示用户信息配置文件中uid大于500的用户名及uid信息，并在开头显示“用户名 UID”字样，在结尾显示“the over”**

```shell
awk -F: 'BEGIN { print "用户名 UID"} END { print "the over"} $3>500{print $1,$3}' /etc/passwd
```

**补充：awk中变量使用**

直接定义，直接使用即可。

awk中**字母会被认为是变量**，如果真的要给一个变量赋值使用双引号

```
[root@mysql-master ~]# awk 'BEGIN{ a=123asdf;print a}'    #awk中字母会被认为是变量
123
[root@mysql-master ~]# awk 'BEGIN{ a="123asdf";print a}' #awk中给变量赋值要加双引号；使用变量直接使用即可
123asdf
```





## 11.2 awk应用举例

创建练习文件：

```shell
[root@ mysql-master ~]# vim data.txt
Beth    4.00    0
Dan     3.75    0
kathy   4.00    10
Mark    5.00    20
Mary    5.50    22
Susie   4.25    18
```



### 11.2.1 简单输出

#### **1. 打印每一行**

如果一个动作没有任何模式, 这个动作会对所有输入的行进行操作. print 语句用来打印(输出)当前输入的行, 所以程序

```shell
[root@ mysql-master ~]# awk '{print}' data.txt
Beth	4.00	0
Dan	3.75	0
kathy	4.00	10
Mark	5.00	20
Mary	5.50	22
Susie	4.25	18
```

会输出所有输入的内容到标准输出. 由于 $0 表示整行,

```shell
[root@ mysql-master ~]# awk '{print $0}' data.txt
Beth	4.00	0
Dan	    3.75	0
kathy	4.00	10
Mark	5.00	20
Mary	5.50	22
Susie	4.25	18
```



#### **2. 打印特定字段**

```shell
[root@ mysql-master ~]#  awk '{print $1,$3}' data.txt
Beth 0
Dan 0
kathy 10
Mark 20
Mary 22
Susie 18
```

在 print 语句中被逗号分割的表达式, 在默认情况下他们将会用一个空格分割 来输出. 每一行 print 生成的内容都会以一个换行符作为结束. 但这些默认行 为都可以自定义。



#### **3. NF, 字段数量**

依次打印出每一行的字段数量, 第一个字段的值, 最后一个字段的值：

```shell
[root@ mysql-master ~]# awk '{print NF, $1, $NF}' data.txt
3 Beth 0
3 Dan 0
3 kathy 10
3 Mark 20
3 Mary 22
3 Susie 18
```



#### 4. 计算和打印

```shell
[root@ mysql-master ~]# awk '{print $1, $2 * $3}' data.txt
Beth 0
Dan 0
kathy 40
Mark 100
Mary 121
Susie 76.5
```





```
cat score.txt
zhaosan 85 92 78
lisheng 89 90 75
zhaoyun 84 88 80
guanyu 83 78 90
liubei 86 88 79
```

第二列求和：

```
cat score.txt|awk '{sum+=$2} END {print "Sum = ", sum}'
```

第二列求平均：

```
cat score.txt|awk '{sum+=$2} END {print "Average = ", sum/NR}'
```

求第二列最大值

```
cat score.txt|awk 'BEGIN {max = 0} {if ($2>max) max=$2 fi} END {print "Max=", max}'
```

求第二列最小值（min的初始值设置一个超大数即可）

```
awk 'BEGIN {min = 1999999} {if ($2<min) min=$2 fi} END {print "Min=", min}'
```





#### 5. 打印行号

Awk提供了另一个内建变量, 叫做 NR, 它会存储当前已经读取了多少行的计数. 我们可以使用 NR 和 $0 给 emp.data 的每一行加上行号:

```shell
[root@ mysql-master ~]# awk '{print NR,$0}' data.txt
1 Beth	4.00	0
2 Dan	3.75	0
3 kathy	4.00	10
4 Mark	5.00	20
5 Mary	5.50	22
6 Susie	4.25	18
```



#### 6. 在输出中添加内容

当然也可以在字段中间或者计算的值中间打印输出想要的内容:

```shell
[root@ mysql-master ~]# awk '{print "total pay for", $1, "is", $2 * $3}' data.txt
total pay for Beth is 0
total pay for Dan is 0
total pay for kathy is 40
total pay for Mark is 100
total pay for Mary is 121
total pay for Susie is 76.5
```







## 11.3 awk流程控制语句

Awk为选择提供了一个 if-else 语句，以及为循环提供了几个语句，所以都效仿C语言中对应的控制语句。**它们仅可以在动作中使用。**

### 11.3.1 if-else语句

格式：

if（条件）{语句；语句} else {语句1；语句2}

```shell
awk 'BEGIN{
test=100;
if(test>90)
{
    print "very good";
}
else if(test>60)
{
    print "good";
}
else
{
    print "no pass";
}
}'

输出：

very good
```

如果statement只有一条语句，{}可以不写；每条命令语句后面可以用“；”号结尾。

以冒号为分隔符，判断第一个字段，如果为root，则显示用户为administrator，否则显示用户问common user

```shell
awk -F: '{if($3==0){print $1,"is administrator."} else {print $1,"is common user"}}' /etc/passwd
```



**1. 编写uid大于500的用户个数**

```shell
awk -F: '{if($3>500){i++}} END {print "uid大于500的用户数量：",i}' /etc/passwd
或者
awk -F: '$3>500{print $0}' /etc/passwd|wc -l

输出：

uid大于500的用户数量： 20
```



**2. 判断系统的bash用户和nologin用户**

```shell
awk -v num1=0 -v num2=0 -F: '/bash$/ || /nologin$/{if($7=="/bin/bash"){num1++} else {num2++}}END{print "bash用户数量：",num1,"nologin用户数量：",num2}' /etc/passwd

输出：

bash用户数量： 23 nologin用户数量： 21
```



### 11.3.2 while语句

格式：

while(条件) {语句1;语句2;.....}

**1. passwd前3行进行输出，输出3次**

```shell
[root@ mysql-master ~]# head -3 /etc/passwd | awk '{i=1;while(i<=3){print $0; i++}}'

root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```



**2.以冒号为分割符，判断每一行的每一个字段的长度如果大于4，则输出**

```shell
head -3 /etc/passwd | awk -F: '{i=1;while(i<=7){if(length($i)>4){print $i};i++}}'

输出：

/root
/bin/bash
/sbin/nologin
daemon
daemon
/sbin
/sbin/nologin
```



**3.统计test.txt的文件中单词长度大于5**

```
awk '{i=1;while(i<NF) {if(length($i)>5){print $i};i++}}' test.txt 
```



### 11.3.3 for循环语句

格式：

for（变量定义；循环终止的条件；改变循环条件的语句） {语句；语句...}

    for(i=1;i<=4;i++) {......}

**1.以冒号为分隔符，显示/etc/passwd每一行的前3个字段**

```
awk -F: '{for (i=1;i<=3;i++) { print $i} }' /etc/passwd
```

**2.将数字1-100着个累加，计算**

```
awk 'BEGIN{ total=0; for(i=0;i<=100;i++) {total+=i}; print total}'

输出：

5050
```



**shell的for循环与awk的for循环的性能比较：**

```shell
[root@ c6m01 ~]# time (awk 'BEGIN{ total=0;for(i=0;i<=10000;i++){total+=i;}print total;}')
50005000

real	0m0.002s
user	0m0.000s
sys		0m0.001s

[root@ c6m01 ~]# time(total=0;for i in $(seq 10000);do total=$(($total+i));done;echo $total;)
50005000

real	0m0.075s
user	0m0.071s
sys		0m0.001s
```