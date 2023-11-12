---
title: Base 中的特殊符号
tags: Base
article_header:
  type: cover
  image:
    src: /screenshot.jpg
---

A Post with Header Image, See [Page layout](https://tianqi.name/jekyll-TeXt-theme/samples.html#page-layout) for more examples.

<!--more-->

## 0x01. `#` 井号 - 注释

注释符号(Hashmark[Comments])

在 shell 文件的行首，作为 shebang 标记， `#!/bin/bash`;

其他地方作为注释使用，在一行中，#后面的内容并不会被执行，除非(what else)；

用单/双引号包围时，`#` 作为 `#` 号字符本身，不具有注释作用。

## 0x02. `;` 分号 - 语句分隔

作为多语句的分隔符(Command separator [semicolon])。

多个语句要放在同一行的时候，可以使用分号分隔。注意，有时候分号需要转义。

## 0x03. `;;` 双分号 - `case` 语句

连续分号(Terminator [double semicolon])。

在使用 case 选项的时候，作为每个选项的终结符。

在 Bash version 4+ 的时候，还可以使用[ `;;&`], [ `;&`]

## 0x04. `.` 点号 - `source`/正则

点号(dot command [period])。

### \1. 

相当于 bash 内建命令source，如：

```bash
    #!/bin/bash
    . data-file
    #包含data-file;
```

\2. 作为文件名的一部分，在文件名的开头，表示该文件为隐藏文件，ls一般不显示出来（ls -a 可以显示）；

\3. 作为目录名，一个点代表当前目录，两个点号代表上层目录（当前目录的父目录）。注意，两个以上的点不出现，除非你用引号（单/双）包围作为点号字符本身；

\4. 正则表达式中，点号表示任意一个字符。

## 0x05. `"` 双引号 - 允许变量扩展

双引号（partial quoting [double quote]）。

部分引用。

双引号包围的内容可以允许变量扩展，也允许转义字符的存在。

如果字符串内出现双引号本身，需要转义，因此双引号不一定是成对的。
6. ' 单引号-禁止变量扩展

单引号(full quoting [single quote])。

单引号括住的内容，被视为单一字符串，引号内的禁止变量扩展，所有字符均作为字符本身处理（除单引号本身之外），单引号必须成对出现。
7. , 逗号-数学表达式中

逗号(comma operator [comma])。

\1. 用在连接一连串的数学表达式中，这串数学表达式均被求值，但只有最后一个求值结果被返回。如：

#!/bin/bash
let t1=((a=5+1, b=7+2))
echo t1=$t1, a=$a, b=$b
## 这个$t1=$b；

\2. 用于参数替代中，表示首字母小写，如果是两个逗号，则表示全部小写，注意，这个特性在bash version 4的时候被添加的。

a="ATest"
echo ${a,}
echo ${a,,}
## 前面输出aTest，后面输出的是atest。

8. / 斜线-目录/除法

斜线，斜杆（Filename path separator [forward slash]）。

1.作为路径的分隔符，路径中仅有一个斜杆表示根目录，以斜杆开头的路径表示从根目录开始的路径；

2.在作为运算符的时候，表示除法符号。如：a=4/2
9. \ 反斜线-转义

反斜线，反斜杆(escape [backslash])。

\1. 放在特殊符号之前，转义特殊符号的作用，仅表示特殊符号本身，这在字符串中常用；

\2. 放在一行指令的最末端，表示紧接着的回车无效（其实也就是转义了Enter），后继新行的输入仍然作为当前指令的一部分。
10. ` 反引号/后引号-命令替换

反引号，后引号（Command substitution[backquotes])。

命令替换。这个引号包围的部分为命令，可以执行包围的命令，并将执行的结果赋值给变量。

如：a=dirname '/tmp/x.log' 。

返回的结果会赋值给a，注意，此处特地使用了反引号和单引号，注意区别。
11. : 冒号-空命令

冒号(null command [colon])。

空命令，这个命令什么都不做，但是有返回值，返回值为0（即：true）。这个命令的作用非常奇妙。

\1. 可做while死循环的条件；

\2. 在if分支中作为占位符（即某一分支什么都不做的时候）；

\3. 放在必须要有两元操作的地方作为分隔符，如：: ${username=whoami}

\4. 在参数替换中为字符串变量赋值，在重定向操作(>)中，把一个文件长度截断为0（:>>这样用的时候，目标存在则什么都不做），这个只能在普通文件中使用，不能在管道，符号链接和其他特殊文件中使用；

\5. 甚至你可以用来注释（#后的内容不会被检查，但:后的内容会被检查，如果有语句如果出现语法错误，则会报错）；

\6. 你也可以作为域分隔符，比如环境变量$PATH中，或者passwd中，都有冒号的作为域分隔符的存在；

\7. 你也可以将冒号作为函数名，不过这个会将冒号的本来意义转变（如果你不小心作为函数名，你可以使用unset -f : 来取消function的定义）。
12. ! 感叹号-取反

感叹号（reverse (or negate) [bang],[exclamation mark])。

取反一个测试结果或退出状态。

\1. 表示反逻辑，比如后面的!=,这个是表示不等于；

\2. 表示取反，如：ls a[!0-9] #表示a后面不是紧接一个数字的文件；

\3. 在不同的环境里面，感叹号也可以出现在间接变量引用里面；

\4. 在命令行中，可以用于历史命令机制的调用，你可以试试!$,!#，或者!-3看看，不过要注意，这点特性不能在脚本文件里面使用（被禁用）。

感叹号的使用(转义) 
#!/bin/bash 
test='hello!@#$%-test'    # 注意,等号两边不能有空格,否则会将叹号当做历史命令调用来处理 
echo  $test 
echo "you !!!" 
test2="test2!" 
echo $test2 
注意:    shell编程里,等号两边不应该有空格(跟其他任何语言都不同,这是bash/sh等的特例). 

13. * 星号-通配符/乘法

星号（wildcard/arithmetic operator[asterisk])。

\1. 作为匹配文件名扩展的一个通配符，能自动匹配给定目录下的每一个文件；

\2. 正则表达式中可以作为字符限定符，表示其前面的匹配规则匹配任意次；

\3. 算术运算中表示乘法。
14. ** 双星号-求幂

双星号(double asterisk)。算术运算中表示求幂运算。
15. ? 问号-条件测试/通配符

问号（test operator/wildcard[Question mark])。

\1. 表示条件测试；

\2. 在双括号内表示C风格的三元操作符((condition?true-result:false-result))；

\3. 参数替换表达式中用来测试一个变量是否设置了值；

\4. 作为通配符，用于匹配文件名扩展特性中，用于匹配单个字符；

\5. 正则表达式中，表示匹配其前面规则0次或者1次。
16. $ 美元符-变量/行末

美元符号(Variable substitution[Dollar sign])。

\1. 作为变量的前导符，用作变量替换，即引用一个变量的内容，比如：echo $PATH；

\2. 在正则表达式中被定义为行末（End of line）。
17. ${}-变量

参数替换(Variable substitution)。

用于在字符串中表示变量。
17. ${!#} ${@🅰️b} 参数的操作

./example.sh -a 1 -b 2 -c 3 -d

参数总个数不确定,从脚本参数的第n个位置开始取参数

参数位置 获取技巧 ${@🅰️b}

$# 表示参数个数,如 echo $#

1.格式

${!#}   取最后一个参数 
${@:a:b}  从第a个参数开始取,合计取b个. 如取到最后一个,则":b"可以不写.有些类似python的切片方式. 

2.常见示例 ${!#} ${@:1:$#-1}

./t.sh 1001 1002 1003 1004 1005 目的要赋值给变量a=1005 , b="1001 1002 1003 1004",则脚本内容为

a=${!#}        
#取最后一个参数 

b=${@:1:$#-1}     
#从第1个参数开始,合计取$#-1个.
其中$@是列表形式列出所有的传入参数;
然后:1是从第一个参数开始，后面不加任何东西的话是一直到结尾;
若加:$#-1是"$#参数总个数-1"，即从第1个参数开始,合计取$#-1个参数. 

若要取倒数第二个参数即c=1004： c=${@😒#-1:1} #@:开始位置(倒数第2个):合计取1个

从第3个参数位置获取: directory=${@:3:$#} #从第3个参数开始,取所有(取$#个,显然取不到这么多个,但取到最后的时候,shell自动会判断结束)

其他示例说明:

a=${@:2:$#-2} 
从第2个开始,合计取参数个数减2个

b=${!#} 
取最后一个

c=${@:$#-3:$#} 
取倒数第3个位置开始,取参数总个数这么多.
(能够取到的参数肯定比参数总个数少,因为是从倒数第三个开始取的.除非从第一个开始取);

d=${@:$#-3} 
取倒数第3个位置开始,全部取完.同上;

18. ${var:offset:number}-字符串切片

扩展 字符串切片

参考: https://www.cnblogs.com/alongdidi/p/bash_parameter_expansion.html

这块在上一篇讲解数组的文章中，也大概提到了Shell Parameter Expansion除了可以对数组（array）切片以外，还可以对变量切片。

格式:

${var:offset:number}

${var: length}

[root@c7-server ~]# name="zhangwenlong"
[root@c7-server ~]# echo ${name}
zhangwenlong

[root@c7-server ~]# echo ${name:2:5}
angwe

[root@c7-server ~]# echo ${name: -4}
long

19. ${var^^} ${var,,}-字符大小写转换

字符大小写转换

格式:

${var^^}：将变量var中的所有小写字符转换成大写。

${var,,}：将变量var中的所有大写字符转换成小写。

[root@c7-server ~]# name=RenDanChaoXian[root@c7-server ~]# echo ${name^^}RENDANCHAOXIAN[root@c7-server ~]# echo ${name,,}rendanchaoxian

20. $‘...’-引用内容展开(未找到示例)

$‘...’ 引用内容展开，执行单引号内的转义内容（单引号原本是原样引用的），这种方式会将引号内的一个或者多个[]转义后的八进制，十六进制值展开到ASCII或Unicode字符。
21. $*或$@-位置参数

位置参数(Positional Parameters)。

这个在使用脚本文件的时候，在传递参数的时候会用到。

两者都能返回调用脚本文件的所有参数.

但$*是将所有参数作为一个整体返回（字符串）;

而$@是将每个参数作为单元返回一个参数列表。

注意，在使用的时候需要用双引号将$*,$@括住。

这两个变量受到$IFS的影响，如果在实际应用中，要考虑其中的一些细节。

补充:

shell特殊 参数 变量 $#,$@,$*,$$,$?,$0,$1的含义解释 整理时间: 20180718 Chenxin shell的特殊字符  $#     是传给脚本的参数个数(不含脚本文件自身) $0     是脚本本身的名字 $1～$n 是传递给该shell脚本的第1-n个参数 $@     所有参数列表.以"$1" "$2" … "$n" 的形式输出所有参数。 $*       所有参数列表.以"$1 $2 … $n"的形式输出所有参数。是以一个单字符串显示所有向脚本传递的参数. $$     是脚本运行的当前进程ID号 $?     是显示最后命令的退出状态，0表示没有错误，其他表示有错误 ${!#}    输出最后一个参数 $!    Shell最后运行的后台Process的PID 完整的特殊字符含义: https://linux.cn/article-5657-1.html  

21. $#-参数数量

表示传递给脚本的参数数量。
22. $?-返回值

此变量值在使用的时候，返回的是最后一个命令、函数、或脚本的退出状态码值，如果没有错误则是0，如果为非0，则表示在此之前的最后一次执行有错误。
23. $$-进程PID

进程ID变量，这个变量保存了运行当前脚本的进程ID值。
24. ()-命令组subshell/数组初始化

圆括号(parentheses)。

1， 命令组（Command group）。由一组圆括号括起来的命令是命令组，命令组中的命令是在子shell（subshell）中执行。因为是在子shell内运行，因此在括号外面是没有办法获取括号内变量的值，但反过来，命令组内是可以获取到外面的值，这点有点像局部变量和全局变量的关系，在实际中，如果碰到要cd到子目录操作，并在操作完成后要返回到当前目录的时候，可以考虑使用subshell来处理；

\2. 用于数组的初始化。
25. {x,y,z,...} 花括号扩展

花括号扩展(Brace Expansion)。

在命令中可以用这种扩展来扩展参数列表，命令将会依照列表中的括号分隔开的模式进行匹配扩展。注意的一点是，这花括号扩展中不能有空格存在，如果确实有必要空格，则必须被转义或者使用引号来引用。

例子：

chenxin@yunwei-01:~$ echo {a,b,c}-{\ d," e",' f'}a- d a- e a- f b- d b- e b- f c- d c- e c- f

26. {a..z} 花括号扩展

在Bash version 3时添加了这种花括号扩展的扩展，可以使用{A..Z}表示A-Z的所有字符列表，这种方式的扩展测试了一下，好像仅适用于A-Z，a-z，还有数字{最小..最大}的这种方式扩展。
27. {} 代码块

代码块(curly brackets)。

这个是匿名函数，但是又与函数不同，在代码块里面的变量在代码块后面仍能访问。

注意：

花括号内侧需要有空格与语句分隔。

另外，在xargs -i中的话，还可以作为文本的占位符，用以标记输出文本的位置。

{ cmd1;cmd2; } :指令以当前进程pid执行;

(cmd1;cmd2) :指令以子进程pid执行;
28. {} ; 貌似只在find里用

这个{}是表示路径名，这个并不是shell内建的，现在接触到的情况看，好像只用在find命令里。

注意后面的分号，这个是结束find命令中-exec选项的命令序列，在实际使用的时候，要转义一下,以免被shell理解错误。
29. [] 中括号-测试/数组元素/正则

中括号（brackets）。

\1. 测试的表示，Shell会测试在[]内的表达式，需要注意的是，[]是Shell内建的测试的一部分，而非使用外部命令/usr/bin/test的链接；

\2. 在数组的上下文中，表示数组元素，方括号内填上数组元素的位置就能获得对应位置的内容，如：

chenxin@yunwei-01:~$ Array[1]=xxxchenxin@yunwei-01:~$ echo ${Array[1]};xxx

\3. 正则中,表示字符集的范围，方括号表示该位置可以匹配的字符集范围。
30. [[]] 双中括号

双中括号(double brackets)。

这个结构也是测试，测试[[]]之中的表达式(Shell的关键字)。

这个比单中括号更能防止脚本里面的逻辑错误，比如：&&,||,<,>操作符能在一个[[]]里面测试通过，但是在[]却不能通过。[[]]里面没有文件名扩展(filename expansion）或是词分隔符(Word splitting)，但是可以用参数扩展(Parameter expansion)和命令替换(command substitution)。不用文件名通配符和像空白这样的分隔符。注意，这里面如果出现了八进制，十六进制等，shell会自动执行转换比较。
31. $[...] 整数扩展,类似$(())

词表达表示整数扩展(integer expansion)。

在方括号里面执行整数表达式。例：

chenxin@yunwei-01:~$ a=3chenxin@yunwei-01:~$ b=7chenxin@yunwei-01:~$ echo $[$a+$b]10chenxin@yunwei-01:~$ echo $[$a*$b]21##返回是10和21##这种很类似$(())的方式chenxin@yunwei-01:~$ echo $((a+b))10chenxin@yunwei-01:~$ echo $((a*b))21chenxin@yunwei-01:~$ echo $(( a * b ))21

32. (()) 双小括号,整数扩展

双括号(double parentheses)。

表示整数扩展（integer expansion）。

功能和上面的$[]差不多，但是需要注意的是，$[]是会返回里面表达式的值的，而(())只是执行，并不会返回值。

两者执行后如果变量值发生变化，都会影响到后继代码的运行。可对变量赋值，可以对变量进行一目操作符操作，也可以是二目，三目操作符。
33. > ， &< ，>&，>>，<，<> 重定向

重定向(redirection)。

scriptname >filename 重定向scriptname的输出到文件filename中去，如果文件存在则覆盖；

command &>filename 重定向command的标准输出(stdout)和标准错误(stderr)到文件filename中；

command >&2 把command的标准输出(stdout)重定向到标准错误(stderr)中；

scriptname >>filename 把scriptname的输出（同>)追加到文件filenmae中，如果文件不存在则创建。

[i]<>filename 打开filename这个文件用来读或者写，并且给文件指定i为它的文件描述符(file descriptor)，文件不存在就会创建。
34. (command)>，<(command) 进程替换

这是进程替换(Process Substitution)。

使用的时候注意，括号和<,>之间是不能有空格的，否则报错。

其作用有点类似管道，但和管道在用法上又有些不同，管道是作为子进程的方式来运行的，这个命令会在/dev/fd/下面产生类似/dev/fd/63,/dev/fd/62这类临时文件，用来传递数据。

Mitchell个人猜测之所以用这种方法来传递，是因为前后两个不属于同一个进程，因此需要用共享文件的方式来传递资料(这么说其实管道也应该有同样的文件?)。网上有人说这个只是共享文件而已，但是经过测试，发现虽然有/dev/fd/63这样的文件产生，但是这个文件其实是指向pipe:[43434]这样的通道的链接。
35. << 双小于号 后继内容重定向到左侧命令的输入

双小于号(here-document [double less then marks] 双小于标记 )。

Here Document 说明

Here Document也被称为here-document/here-text/heredoc/hereis/here-string/here-script，在Linux/Unix中的shell中被广泛地应用，尤其在于用于传入多行分割参数给执行命令。

除了shell（包含sh/csh/tcsh/ksh/bash/zsh等），这种方式的功能也影响和很多其他语言诸如Perl，PHP以及Ruby等。

这个也被称为Here-document，用来将后继的内容重定向到左侧命令的stdin中。

<<可以节省格式化时间，别且使命令执行的处理更容易。

在实作的时候只需要输入<<和终止标志符，而后（一般是回车后）你就可以输入任何内容，只要在最后的新行中输入终止标志符，即可完成数据的导入。

Here Document 使用限制

1.分割串常见的为EOF，但不一定固定为EOF，可以使用开发者自行定义的，比如EAOF;2.缺省方式下第一个分割串（EOF）前后均可有空格或者tab，运行时会自动剔除，不会造成影响;(未必,曾经遇到过起始EOF后不可以有空格问题);3.缺省方式下第二个分割串（EOF）必须顶格写，前后均不可有空格或者tab;

使用here-document的时候，你可以保留空格，换行等。

<<- 与 << 的区别

如果要让shell脚本更整洁一点，可以在<<和终止符之间放上一个连字符(-);

这样,

1.结束行的EOF字符前,就可以有空格或者tab键了(这里有疑问,这条解释经验证可能是错的).

2.使用<<-代替<<唯一的作用在与分割串所扩起来的内容，顶格的tab会被删除，用于ident。

示例1:

chenxin@yunwei-01:~/1tmp$ cat -A 1.sh #!/bin/bash$$date_tmp=`date +%s`$$cat <<- 'EOF'                     $ echo hello1$  echo hello2$    echo hello$    echo $date_tmp$EOF$$echo $date_tmp$chenxin@yunwei-01:~/1tmp$ ./1.sh  echo hello1  echo hello2    echo hello    echo $date_tmp1645672402

示例2:

cat > env-build-2.sh <<- "EOF"#!/bin/bashecho "--- docker tag and push ---"docker_image_id=$( docker images |grep $AIC_BUILD_NUMBER |grep "^android" |awk '{print $3}' ) || ( echo "cant find docker images for android" && exit 1 )docker tag "$docker_image_id" 11.122.132.129:80/aic/android:"$AIC_BUILD_NUMBER"-"$lunch_choice"echo "--- Clean up related containers ---"docker container ps -a|grep -E 'android:eng|aic-manager:eng' |grep 'Created' |awk '{print $1}'|xargs docker container rmEOF说明:上面第一个EOF的双引号,是防止文件中的$符号被转移为变量值后注入到xxx.sh文件中.

36. <<< 3个小于号

三个小于号(here-strings)。

Here-字串和Here-document类似，here-strings语法：

command [args] <<<["]$word["]；$word会展开并作为command的stdin。

<<< 就是将后面的内容作为前面命令的标准输入

grep a <<< "$VARIABLE" 意思就是在VARIABLE这个变量值里查找字符a

示例

chenxin@yunwei-01:~$ aaa='this is bbb'chenxin@yunwei-01:~$ grep bbb <<< $aaathis is bbbchenxin@yunwei-01:~$ grep bbb << $aaa  #两个就不行> ;> <> ^C

37. >, < 什么意思?

小于，大于号(ASCII Comparison)。

ASCII比较，进行的是变量的ASCII比较，字串？数字?呃...这个...不就是ASCII比较么？
38. <...> 用于标记单词的分界

词界符(word boundary)。

这个是用在正则表达式中的一个特殊分隔符，用来标记单词的分界。

比如：

the会匹配there，another，them等等，如果仅仅要匹配the，就可以使用这个词界符，<the>就只能匹配the了。
39. | 管道

管道(pipe)。

管道是Linux，Unix都有的概念，是非常基础，也是非常重要的一个概念。

它的作用是将管道前（左边）的命令产生的输出(stdout)作为管道后（右边）的命令的输入(stdin)。

如：ls | wc l，使用管道就可以将命令连接在一起。

注意：

管道是每一个进程的标准输出都会作为下一个命令的标准输入，期间的标准输出不能跨越管道作为后继命令的标准输入，

如： cat filename | ls -al | sort 。

想想这个的输出? 同时，管道是以子进程来运行的，所以管道并不能引起变量改变。
40. >| 强制重定向

强制重定向(force redirection)。

这会强制重写已经存在的文件。
41. & 与

与号(Run job in background[ampersand])。

如果命令后面跟上一个&符号，这个命令将会在后台运行。

有的时候，脚本中在一条在后台运行的命令可能会引起脚本挂起，等待输入，出现这种情况可以在原有的脚本后面使用wait命令来修复。
42. &&，|| 逻辑操作符

逻辑操作符(logical operator)。

在测试结构中，可以用这两个操作符来进行连接两个逻辑值。

||是当测试条件有一个为真时返回0（真），全假为假；

&&是当测试条件两个都为真时返回真(0)，有假为假。
43. - 减号

减号，连字符(Hyphen/minus/dash)。

\1. 作为选项，前缀[option, prefix]使用。用于命令或者过滤器的选项标志；操作符的前缀。如：

## COMMAND -[选项列表]ls -alsort -dfu $fileset -- $variable if [ $file -ot $file2 ]then    echo "$file is older than $file2."fi

\2. 用于stdin或者stdout的重定向的源或目的[dash].

在tar没有bunzip2的程序补丁时，我们可以这样： bunzip2 linux-2.6.13.tar.bz2 | tar xvf - 。将前面解压的数据作为tar的标准输入（这里使用一个-表示）

注意：在实作的时候，如果文件名是以[-]开头的，那么在加上这个作为定向操作符的时候，可能会出错，此时应该为文件加上合适的前缀路径，以避免这种情况发生，同样的，在echo变量的时候，如果变量是以[-]开始，那么可能也会产生意想不到的结果，为了保险起见，可以使用双引号引用标量：

var="-n"echo $var## 试试看有什么输出？

还有，这种表示方法不是Bash内建的，要达到此点的这种效果，需要看你使用的软件是否支持这种操作；

\3. 表示先前的工作目录(previous working directory)，因此，如果你cd到其他目录下要放回前一个路径的时候，可以使用cd -来达到目的，其实，这里的[-]使用的是环境变量的$OLDPWD，注意：这里的[-]和前一点是不同的；

\4. 减号或者负号，用在算术操作中。
43. -- 双减号

shell中的 -- 含义 (两个横线的含义)

两个横线代表选项的结束，两个横线后面的部分都会被认为是参数了，而不再是前面的命令的选项了。

[admin@ip-10-0-1-23 ~]$ echo -- -e hello

-- -e hello

[admin@ip-10-0-1-23 ~]$ echo -e hello

hello

echo -- -e hello和echo -e hello是不一样的，前者-e是一个普通参数，后者-e则是echo的一个选项.

在你的这个场合下，set -- mysqld表示重设脚本的参数为mysqld，会影响到$argv变量和$1，$#等和参数有关的变量(-- 表示将任何剩余的参数分配给位置参数, 如果没有剩余的参数, 就会将位置参数复位).

-- 代表后面的字符不是set命令的选项( 比如 set -- "$@" "-h",表示用set指令将参数追加一个-h).

Chanix-LGdeMacBook-Pro:shell_test chanix$ echo $@

Chanix-LGdeMacBook-Pro:shell_test chanix$ set -- "$@" "-h"

Chanix-LGdeMacBook-Pro:shell_test chanix$ echo $@

-h
44. = 等号

等号(Equals)。

\1. 赋值操作，给变量赋值，在等号两侧禁止有空格；

\2. 在比较测试中作为比较符出现，这里要注意，如果在[]中括号中作为比较出现，需要有空格符在等号左右两侧,如 [ $a==$b ]。
45. + 加号

加号(Plus)。

\1. 算术操作符，表示加法；

\2. 在正则表达式中，表示的是其前的这个匹配规则匹配最少一次;

3.在命令或过滤器中作为选项标记，在某些命令或者内置命令中使用+来启用某些选项，使用-来禁止；

\4. 在参数替换(parameter substitution)中，+前缀表示替代值(当变量为空的时候，使用+后面的值)
56. % 百分号-求模/模式匹配

百分号(modulo[percent sign])。

1.在算术运算中，这个是求模操作符，即两个数进行除法运算后的余数；

2.在参数替换(parameter substitution)中，可以作为模式匹配。例子：

p=b*9var="abcd12345abc479"echo ${var%p}, ${var%%p}##从右边开始查找(想想从左是那个符号?)##任何在b和9之间的内容（含）##第一个是找到最短的符合匹配项##后一个是找最大符合的匹配项（贪婪匹配?)#经测试,以上说法不对呢...chenxin@yunwei-01:~$ p=b*9chenxin@yunwei-01:~$ var="abcd12345abc479"chenxin@yunwei-01:~$ echo ${var%p}, ${var%%p}abcd12345abc479, abcd12345abc479

57. ~ 波浪号

波浪号(Home directory[tilde])。

这个和内部变量$HOME是一样的。

默认表示当前用户的家目录（主目录），这个和~/效果一致，如果波浪号后面跟用户名，表示是该用户的家目录。
58. ~+ 同$PWD

当前的工作目录(current working directory)。

这个和内置变量$PWD一样。

chenxin@yunwei-01:~$ echo ~+/home/chenxinchenxin@yunwei-01:~$ pwd/home/chenxinchenxin@yunwei-01:~$ echo $pwdchenxin@yunwei-01:~$ echo $PWD/home/chenxin

59. ~-

前一个工作目录(previous working directory)。

这个和内部变量$OLDPWD一致，之前的[-]也一样。

经测试,以上说法并不符...

chenxin@yunwei-01:~$ echo ~-
~-
chenxin@yunwei-01:~$ echo $OLDPWD

chenxin@yunwei-01:~$ 

60. =~ 在[[]]中匹配

Bash 版本3中有介绍，这个是正则表达式匹配。可用在[[]]测试中，比如：

var="this is a test message."
[[ "$var" =~ tf*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
##你可以修改中间的正则表达式匹配项，正则表达式可以但不一定需要使用双引号括起来。


chenxin@yunwei-01:~$ var="this is a test message."
chenxin@yunwei-01:~$ [[ "$var" =~ tf*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
Sorry Sir. No match be found.

chenxin@yunwei-01:~$ [[ "$var" =~ t*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
Sir. Found that.

chenxin@yunwei-01:~$ [[ "$var" =~ th*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
Sorry Sir. No match be found.

chenxin@yunwei-01:~$ [[ "$var" =~ th*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
Sorry Sir. No match be found.

chenxin@yunwei-01:~$ [[ "$var" =~ th.*message ]] && echo "Sir. Found that." || echo "Sorry Sir. No match be found."
Sir. Found that.

61. ^ 脱字符

脱字符(caret)。

\1. 在正则表达式中，作为一行的行首(beginning-of-line)位置标志符；

\2. 在参数替换(Parameter substitution)中，这个用法有两种，

一个脱字符(${var^})，表示第一个字母大写.

两个脱字符(${var^^})，表示全部大写的意思(Bash version >=4)。

示例

chenxin@yunwei-01:~$ var="this is a test message."

chenxin@yunwei-01:~$ echo ${var^}
This is a test message.

chenxin@yunwei-01:~$ echo ${var^^}
THIS IS A TEST MESSAGE.

chenxin@yunwei-01:~$ echo ${var}
this is a test message.

62. 空白-空格/tab/空行

空白符(Whitespace)。

空白符不仅仅是指空格(spaces)，还包括制表符(tabs)，空行(blank lines)，或者这几种的组合。

可用做函数的分隔符,分隔命令或变量.

空行不会影响脚本的行为，因此可以用它来规划脚本代码，以增加可读性，在内置的特殊变量$IFS可以用来针对某些命令进行输入的参数进行分割，其默认就是空白符。

在字符串或变量中如果有空白符，可以使用引号来规避可能的错误。

## appdedix

参考和抄袭:

- <https://www.cnblogs.com/chanix/p/15940444.html>