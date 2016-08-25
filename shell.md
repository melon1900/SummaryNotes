1. ```<<和<<- ```

这种重定向形式被称为here document 或者 here script，语法如下：

```
command << token
text
token
```
**command** 是一个可以接受标准输入的命令名，**token**是一个用来指示嵌入文本结束的字符串。注意这个token必须在一行中单独出现，并且后面不允许有其它字符包括空格。<<-和<<的区别是<<-会忽略**text**中每行行首的tab 字符，便于排版。**text**中的"，'均不再有特殊意义，**text**可以通过$使用变量来生成动态内容。
2. 用命令的执行结果为变量赋值

```
variable_name=`ls -la | tail -2`
```
或

```
variable_name=$(ls -la | tail -2)
```
3. 函数定义

```
function name {
    commands
    return
}
或
name () {
    commands
    return
}
```
函数定义必须出现在调用它的代码之前。函数名的命名和变量的命名一样，只能由数字、字母和下划线组成且不能以数字开头。
4. 包装新命令
在.bashrc文件中通过
```
alias newcmd='df -h'
或
newcmd() {
    echo 'this is a new command'
    df -h
}
```
5. 条件判断

```
test expression
或
[ expression ]
```
后一种格式更常用，注意expression和[、]之间的空格不能省。expression 是一个表达式，其执行结果是true或者是false。当表达式为真时，这个test命令返回一个零退出状态，当表达式为假时，test命令退出状态为 1。

**注：表达式中操作符和操作数之间必须有空格**
#### 文件表达式

Expression | Is ture If
---|---
file1 -ef file2 | file1 and file2 have the same inode numbers (the two filenames refer to the same file by hard linking).
file1 -nt file2 | file1 is newer than file2.
file1 -ot file2 | file1 is older than file2.
-b file | file exists and is a block special (device) file.
-c file | file exists and is a character special (device) file.
-d file | file exists and is a directory.
-e file | file exists.
-f file | file exists and is a regular file.
-g file | file exists and is set-group-ID.
-G file | file exists and is owned by the effective group ID.
-k file | file exists and has its “ sticky bit” set.
-L file | file exists and is a symbolic link.
-O file | file exists and is owned by the effective user ID.
-p file | file exists and is a named pipe.
-r file | file exists and is readable (has readable permission for the effective user).
-s file | file exists and has a length greater than zero.
-S file | file exists and is a network socket.
-t fd | fd is a file descriptor directed to/from the terminal. This can be used to determine whether standard input/output/error is being redirected.
-u file | file exists and is setuid.
-w file | file exists and is writable (has write permission for the effective user).
-x file | file exists and is executable (has execute/search permission for the effective user)

#### 字符串表表达式
Expression | Is ture If
---|---
string | string is not null.
-n string | The length of string is greater than zero.
-z string | The length of string is zero.
string1 = string2 or string1 == string2 | string1 and string2 are equal. Single or double equal signs may be used, but the use of double equal signs is greatly preferred.
string1 != string2 | string1 and string2 are not equal.
string1 > string2 | sting1 sorts after string2.
string1 < string2 | string1 sorts before string2.
注意：虽然bash文档声明排序遵从当前语系的排列规则，但并不这样。将来的bash版本，包含4.0，使用 ASCII（ POSIX）排序规则。
#### 整型表达式
Expression | Is ture If
---|---
integer1 -eq integer2 | integer1 is equal to integer2.
integer1 -ne integer2 | integer1 is not equal to integer2.
integer1 -le integer2 | integer1 is less than or equal to integer2.
integer1 -lt integer2 | integer1 is less than integer2.
integer1 -ge integer2 | integer1 is greater than or equal to integer2.
integer1 -gt integer2 | integer1 is greater than integer2.

#### 条件测试增强版

```
[[ expression ]]
```
主要是在[]条件测试基础上给==操作增加了正则匹配和模式匹配，下面分别取个例子。
正则匹配
```
[me@linuxbox ~]$ INT=-3
[me@linuxbox ~]$ if [[ "$INT" =~ ^-?[0-9]+$ ]]; then
> echo "$INT is an integer"
> fi
-3 is an integer
```
模式匹配
```
[me@linuxbox ~]$ FILE=foo.bar
[me@linuxbox ~]$ if [[ $FILE == foo.* ]]; then
> echo "$FILE matches pattern 'foo.*'"
> fi
foo.bar matches pattern 'foo.*'
```
#### 为整数设计的(())
 (( )) 是 shell 语法的一部分，而不是一个普通的命令，而且
它只处理整数，所以它能够通过名字识别出变量。即[ $a == 0] 和 ((a==0))是等价的

### 逻辑操作符
test 和 [[ ]] 使用不同的操作符来表示逻辑操作符
Operation | test | [[ ]] and (( ))
---|---|---
AND | -a | &&
OR | -o | ```||```
NOT | ! | !
我们也可以对表达式使用圆括号，为的是分组。如果不使用括号，那么否定只应用于第一个表达式，而不是两个组合的表达式。用test可以这样来编码：

```
if [ ! \( $INT -ge $MIN_VAL -a $INT -le $MAX_VAL \) ]; then
echo "$INT is outside $MIN_VAL to $MAX_VAL."
else
echo "$INT is in range."
fi
```
**因为test(等同于[])使用的所有的表达式和操作符都被shell看作是命令参数（不像[[ ]]和(())），对于bash有特殊含义的字符，比如说<，>，(，和)，必须引起来或者是转义。**
test 更传统（是 POSIX的一部分），然而[[]]特定于bash。知道怎样使用test很重要，因为它被非常广泛地应用，但是显然[[]]更有助于，并更易于编码。

#### 控制操作符

```
command1 && command2
```
先执行 command1，只有command1执行成功后，才会执行 command2。

```
command1 || command2
```
先执行 command1，只有command1 执行失败后，才会执行 command2。
6. 读键盘输入

```
read [-options] [variable...]
```
variable 是用来存储输入数值的一个或多个变量名。如果没有提供变量名， shell 变量 REPLY 会包含数据行。

```
IFS=":" read user pw uid gid name home shell <<< $(grep "^root:" /etc/passwd)
```
通常，shell 对提供给read的输入按照单词进行分离。这种行为由shell变量IFS配置。IFS的默认值包含一个空格，一个tab，和一个换行符，每一个都会把字段分割开。上面的脚本在read之前把IFS改为了:。<<< 操作符指示一个 here 字符串。一个 here 字符串就像一个 here 文档，只是比较简短，由单个字符串组成。

7. 循环
#### while和until循环
语法如下：
```
while [  ]; do
    if [  ];then
        break
    fi
    
    if [  ];then
        continue
    fi
done
```
until循环

```
until [  ]; do
    if [  ];then
        break
    fi
    
    if [  ];then
        continue
    fi
done
```
until 命令与 while 非常相似，当遇到一个非零退出状态的时候， while 退出循环，当循环条件为假时。而until正好相反，当循环条件为真时退出。

#### for循环
语法如下：
传统shell格式
```
for variable [in words]; do
commands
done
```
例如

```
for i in A B C D; do echo $i; done
```
```
for i in {A..D}; do echo $i; done
```
```
for i in distros*.txt; do echo $i; done
```
```
for i in $(ls -la | tail -2); do echo $i; done
```
```
for i in $(seq 10); do echo $i; done
```
C语言格式

```
for (( expression1; expression2; expression3 )); do
    commands
done
```


8. 调试脚本

在测试包含删除文件操作的脚本时，可以在删除文件操作命令前加上echo，打印删除命令但不实际删除文件，如：
echo rm * 

bash 还提供了一种名为追踪的方法，这种方法可通过 -x 选项和 set 命令加上 -x 选项两种途径实现。set 命令加上 -x 选项来启动追踪， +x 选项关闭追踪

```
#!/bin/bash -x
your shell code
...
```
或

```
#!/bin/bash
set -x
...
set +x

....
....
set -x
...
set +x

...
```
加号是追踪输出的默认字符。它包含在 PS4（提示符 4） shell 变量中。可以调整这个变量值让提示信息更有意义，比如在追踪输出输出当前行号：

```
export PS4='$LINENO + '
```
9. case

```
case word in
[pattern [| pattern]...) commands ;;]...
esac
```
case 命令检查一个变量值，然后试图去匹配其中一个具体的模式。当与之相匹配的模式找到之后，就会执行与该模式相关联的命令。若找到一个模式之后，就不会再继续寻找。case语句使用的模式和路径展开中使用的那些是一样的模式以一个)为终止符。下面是一些有效的模式：
Pattern | Description
---|---
a) | Matches if word equals ”a”.
[[:alpha:]]) | Matches if word is a single alphabetic character.
???) | Matches if word is exactly three characters long.
*.txt) | Matches if word ends with the characters ".txt" .
*) | Matches any value of word. It is good practice to include this as the last pattern in a case command, to catch any values of word that did not match a previous pattern; that is, to catch any possible invalid values.

早于版本号 4.0 的bash，case语法只允许执行与一个成功匹配的模式相关联的动作。匹配成功之后，命令将会终止。

现在的 bash 版本，添加";;&"表达式来终止每个行动，所以现在我们可以做到这一点：

```
read -n 1 -p "Type a character > "
case $REPLY in
[[:upper:]]) echo "'$REPLY' is upper case." ;;&
[[:lower:]]) echo "'$REPLY' is lower case." ;;&
[[:alpha:]]) echo "'$REPLY' is alphabetic." ;;&
[[:digit:]]) echo "'$REPLY' is a digit." ;;&
[[:graph:]]) echo "'$REPLY' is a visible character." ;;&
[[:punct:]]) echo "'$REPLY' is a punctuation symbol." ;;&
[[:space:]]) echo "'$REPLY' is a whitespace character." ;;&
[[:xdigit:]]) echo "'$REPLY' is a hexadecimal digit." ;;&
esac
```
10. 位置参数
shell 提供了一个称为位置参数的变量集合，这个集合包含了命令行中所有独立的单词。这些变量按照从0到9给予命名。通过参数展开方式你可以访问的参数个数多于9个。只要指定一个大于9的数字，用花括号把该数字括起来就可以。例如 ${10},${211}，等等。

参数 | 描述
---|---
$# | 获取命令行参数个数
$0 | 已执行程序的路径名（在函数中使用$0时也代表的是程序名）
$* | 展开成一个从1开始的位置参数列表。当它被用双引号引起来的时候，展开成一个由双引号引起来的字符串，包含了所有的位置参数，每个位置参数由shell变量IFS的第一个字符（默认为一个空格）分隔开。
$@ | 展开成一个从1开始的位置参数列表。当它被用双引号引起来的时候，它把每一个位置参数展开成一个由双引号引起来的分开的字符串。

遍历位置参数
```
count=1
while [[ $# -gt 0 ]]; do
    echo "Argument $count = $1"
    count=$((count + 1))
    shift
done
```
每次 shift 命令执行的时候，变量 $2 的值会移动到变量 $1 中，变量 $3 的值会移动到变量$2 中，依次类推。变量 $# 的值也会相应的减1。

或
```
for i in $@;do
    echo $i
done
```

$*和$@的区别

```
#!/bin/bash
# posit-params3 : script to demonstrate $* and $@
print_params () {
echo "\$1 = $1"
echo "\$2 = $2"
echo "\$3 = $3"
echo "\$4 = $4"
}
pass_params () {
echo -e "\n" '$* :'; print_params $*
echo -e "\n" '"$*" :'; print_params "$*"
echo -e "\n" '$@ :'; print_params $@
echo -e "\n" '"$@" :'; print_params "$@"
}
pass_params "word" "words with spaces"

输入结果：
$* :
$1 = word
$2 = words
$3 = with
$4 = spaces
"$*" :
$1 = word words with spaces
$2 =
$3 =
$4 =
$@ :
$1 = word
$2 = words
$3 = with
$4 = spaces
"$@" :
$1 = word
$2 = words with spaces
$3 =
$4 =
```
11. 字符串和数字
#### 参数展开
形式 | 描述
---|---
${parameter:-word}|若parameter没有设置（例如，不存在）或者为空，展开结果是 word的值。若parameter不为空，则展开结果是parameter 的值。
${parameter:=word}|若parameter没有设置（例如，不存在）或者为空，展开结果是 word的值，并且word的值会赋值给parameter。若parameter不为空，则展开结果是parameter 的值。
${parameter:?word}|若parameter没有设置或为空，这种展开导致脚本带有错误退出，并且 word 的内容会发送到标准错误。若 parameter 不为空，展开结果是 parameter 的值。
${parameter:+word}|若parameter没有设置或为空，展开结果为空。若 parameter 不为空，展开结果是word的值会替换掉parameter的值；然而， parameter 的值不会改变。
${!prefix*}或${!prefix@}|这种展开会返回以prefix开头的已有变量名
${#parameter}|展开成由 parameter 所包含的字符串的长度。通常， parameter 是一个字符串；然而，如果parameter是@或者是*的话，则展开结果是位置参数的个数。
${parameter: offset}或${parameter: offset:length}|从parameter包含的字符串的第offset个字符（从字符串开头算起，0索引）提取length个字符。如果不指定length则默认提取到末尾。offset可以为负数，如果为负数则表示从字符串末尾开始反向提取，不过其前面必须有一个空格，否则会与${parameter:-word}混淆。如果 parameter 是 @，展开结果是 length 个位置参数，从第 offset 个位置参数开始。
${parameter#pattern}和${parameter##pattern}|清除parameter中匹配pattern的字符串，pattern是通配符模式，就如那些用在路径名展开中的模式。# 形式清除最短的匹配结果，而该 ##模式清除最长的匹配结果。
${parameter%pattern}和${parameter%%pattern}|和上面的 # 和 ## 一样，只不过它们清除的文本从parameter所包含字符串的末尾开始，而不是开头。
${parameter/pattern/string} ${parameter//pattern/string} ${parameter/#pattern/string} ${parameter/%pattern/string}| parameter 的内容执行查找和替换操作。如果找到了匹配通配符 pattern的文本，则用string的内容替换它。在正常形式下，只有第一个匹配项会被替换掉。在该// 形式下，所有的匹配项都会被替换掉。该/# 要求匹配项出现在字符串的开头，而/%要求匹配项出现在字符串的末尾。/string 可能会省略掉，这样会导致删除匹配的文本。

${parameter#pattern}和${parameter##pattern}使用示例
``` 
[me@linuxbox ~]$ foo=file.txt.zip
[me@linuxbox ~]$ echo ${foo#*.}
txt.zip
[me@linuxbox ~]$ echo ${foo##*.}
zip
```
格式 | 结果
---|---
${parameter„}|把 parameter 的值全部展开成小写字母。
${parameter,}|仅仅把 parameter 的第一个字符展开成小写字母。
${parameterˆˆ}|把 parameter 的值全部转换成大写字母。
${parameterˆ}|仅仅把parameter的第一个字符转换成大写字母（首字母大写）。
declare 命令可以用来把字符串规范成大写或小写字符。使用 declare 命令，我们能强制一个变量总是包含所需的格式，无论如何赋值给它。
```
declare -u upper
declare -l lower
```
#### 算术运算
表示法 | 描述
---|---
number |默认情况下，没有任何表示法的数字被看做是十进制数（以10 为底）。
0number |在算术表达式中，以零开头的数字被认为是八进制数。
0xnumber |十六进制表示法，如$((0xff))
base#number |number 以 base 为底，如 $((2#11111111))
二元运算符 ** 表示乘方运算

shell 算术只操作整形，所以除法运算的结果总是整数。$(( 5 / 2 ))为2

if (( foo = 5 ))和if (( foo == 5 ))是不一样的，前面=是赋值运算符

支持位运算，除了按位取反运算符之外，其它所有位运算符都有相对应的赋值运算符（例如<<=）。如((foo>>=1))

支持逻辑运算和问号表达式。如((a<1?(a+=1) : (a-=1)))

bc 一个可以进行高精度计算的外部程序，例如浮点运算。

12. 数组
bash只支持一维数组。
#### 创建
非关联数组就像其它bash变量一样命名，当被访问的时候，它们会被自动地创建。也可以通过带有 -a 选项的declare 命令创建

关联数组必须用带有 -A 选项的declare 命令创建。
```
declare -A colors
colors["red"]="#ff0000"
```
#### 数组赋值
```
a[0]=1
bb=(1 2 3)
days=([0]=Sun [1]=Mon [2]=Tue [3]=Wed [4]=Thu [5]=Fri [6]=Sat)
animals=("a dog" "a cat" "a fish")
```
#### 数组操作
下标 * 和 @ 可以被用来访问数组中的每一个元素。与位置参数一样， @ 表示法在两者之中更有用处。
```
[me@linuxbox ~]$ animals=("a dog" "a cat" "a fish")
[me@linuxbox ~]$ for i in ${animals[*]}; do echo $i; done
a
dog
a
cat
a
fish
[me@linuxbox ~]$ for i in ${animals[@]}; do echo $i; done
a
dog
a
cat
a
fish
[me@linuxbox ~]$ for i in "${animals[*]}"; do echo $i; done
a dog a cat a fish
[me@linuxbox ~]$ for i in "${animals[@]}"; do echo $i; done
a dog
a cat
a fish
```
确定数组元素个数
```
[me@linuxbox ~]$ a[100]=foo
[me@linuxbox ~]$ echo ${#a[@]} # number of array elements
1
[me@linuxbox ~]$ echo ${#a[100]} # length of element 100
3
```
bash 允许赋值的数组下标包含“间隔”，有时候确定哪个元素真正存在是很有用的。为做到这一点，可以使用以下形式的参数展开：${!array[*]} ${!array[@]}

```
[me@linuxbox ~]$ foo=([2]=a [4]=b [6]=c)
[me@linuxbox ~]$ for i in "${foo[@]}"; do echo $i; done
a b c
[me@linuxbox ~]$ for i in "${!foo[@]}"; do echo $i; done
2 4 6
```
通过使用 += 赋值运算符，我们能够自动地把值附加到数组末尾

```
[me@linuxbox~]$ foo=(a b c)
[me@linuxbox~]$ echo ${foo[@]}
a b c
[me@linuxbox~]$ foo+=(d e f)
[me@linuxbox~]$ echo ${foo[@]}
a b c d e f
```
删除数组
```
[me@linuxbox~]$ foo=(a b c d e f)
[me@linuxbox~]$ echo ${foo[@]}
a b c d e f
[me@linuxbox~]$ unset 'foo[2]'
[me@linuxbox~]$ echo ${foo[@]}
a b d e f

unset foo #删除整个数组
```
数组元素必须用引号引起来为的是防止 shell 执行路径名展开操作。

任何引用一个不带下标的数组变量，则指的是数组的首元素

13. 重定向

1 表示stdout标准输出，系统默认值是1
2 表示stderr标准错误
重定向标准输入
```
command < file
```

重定向标准输出
```
command > file
```
重定向标准错误
```
command 2> file
```
重定向标准输出和错误到同一文件
```
command > file 2>file
command > file 2>&1
command &> file
```
command > file 2>&1和command &> file是等价的

file为/dev/null表示丢弃输出

& 表示等同于的意思，2>&1，表示2的输出重定向等同于1
   
command > file 2>file的意思是将命令所产生的标准输出信息,和错误的输出信息送到file 中.command > file2 > file 这样的写法,stdout和stderr都直接送到file中,file会被打开两次,这样stdout和stderr会互相覆盖,这样写相当使用了FD1和FD2两个同时去抢占file 的管道.而command >file 2>&1这条命令就将stdout直接送向file, stderr继承了FD1管道后,再被送往file,此时,file只被打开了一次,也只使用了一个管道FD1,它包括了stdout和stderr的内容.从IO效率上,前一条命令的效率要比后面一条的命令效率要低,所以在编写shell脚本的时候,较多的时候我们会用command > file 2>&1 这样的写法.

cat合并文件
```
cat movie.mpeg.0* > movie.mpeg
```
tee －从 Stdin 读取数据，并同时输出到 Stdout 和文件
```
[me@linuxbox ~]$ ls /usr/bin | tee ls.txt | grep zip
bunzip2
bzip2
....
```
14. 进程

保持进程后台运行即使关闭了把该进程放入后台运行的终端窗口

```
nohup command > file 2>&1 &
```

nohup 的用途就是让提交的命令忽略 hangup 信号。

nohup 无疑能通过忽略HUP信号来使我们的进程避免中途被中断，但如果我们换个角度思考，如果我们的进程不属于接受HUP信号的终端的子进程，那么自然也就不会受到HUP信号的影响了。setsid就能帮助我们做到这一点。
```
setsid command
```

进程返回前台(setsid方式添加的后台任务不适用)
```
[me@linuxbox ~]$ jobs
[1]+ Running xlogo &
[me@linuxbox ~]$ fg %1
xlogo
```
使用kill向进程发送信号

```
kill [-signal] PID...
killall [-u user] [-signal] name...
```
编号 | 名字 | 含义
---|---|---
1 | HUP | 挂起。这是美好往昔的痕迹，那时候终端机通过电话线和调制解调器连接到远端的计算机。这个信号被用来告诉程序，控制的终端机已经“挂起”。通过关闭一个终端会话，可以说明这个信号的作用。发送这个信号到终端机上的前台程序，程序会终止。许多守护进程也使用这个信号，来重新初始化。这意味着，当发送这个信号到一个守护进程后，这个进程会重新启动，并且重新读取它的配置文件。Apache 网络服务器守护进程就是一个例子。
2 |INT |中断。实现和 Ctrl-c 一样的功能，由终端发送。通常，它会终止一个程序。
3 |QUIT |退出
9 |KILL |杀死。这个信号很特别。鉴于进程可能会选择不同的方式，来处理发送给它的信号，其中也包含忽略信号，这样呢，从不发送 Kill 信号到目标进程。而是内核立即终止这个进程。当一个进程以这种方式终止的时候，它没有机会去做些“清理”工作，或者是保存劳动成果。因为这个原因，把 KILL 信号看作杀手锏，当其它终止信号失败后，再使用它。
15 |TERM |终止。这是kill命令发送的默认信号。如果程序仍然“活着”，可以接受信号，那么这个信号终止。
18 |CONT |继续。在停止一段时间后，进程恢复运行。
19 |STOP |停止。这个信号导致进程停止运行，而没有终止。像KILL 信号，它不被发送到目标进程，因此它不能被忽略。
20 |TSTP |终端停止。当按下 Ctrl-z 组合键后，终端发送这个信号。不像 STOP 信号， TSTP 信号由目标进程接收，且可能被忽略。
28 |WINCH |改变窗口大小。当改变窗口大小时，系统会发送这个信号。一些程序，像 top 和 less 程序会响应这个信号，按照新窗口的尺寸，刷新显示的内容。

basename /usr/bin/sort 输出sort

basename include/stdio.h .h 输出stdio

15.shell模式匹配

通配符用于模式匹配，如文件名匹配、路径名搜索、字符串查找等。
常用的通配符有四种：

通配符 | 描述
---|---
*| 匹配任意字符0次或多次出现。例如，f*可以匹配以f打头的任意字符串。但应注意，文件名前面的圆点( . ) 和路径名中的斜线( / )必须显式匹配。
？| 匹配任意一个字符，例如，f ?匹配f1、fa、fb等，但不能匹配 f 、fabc、 f12等。
[ ]| 匹配字符组所限定的任何一个字符。该字符组可以由直接给出的字符组成，也可以由表示限定范围的起始字符、终止字符及中间一个连字符（-）组成。例如，f[a-d]与f[abcd]作用相同。
! | 匹配不在字符组中的字符。例如，f[!1—9].c 表示以f打头，后面一个字符不是数字1至9的.c文件名，它匹配fa.c、fb.c、fm.c等。

扩展模式匹配表达式
通配符 | 描述
---|---
*(模式表)| 匹配给定模式表中“模式”的0次或多次出现。例如，file*(.c &#124; .o)将匹配文件file、file.c、file.o、file.c.c、file.0.0、file.c.o、file.o.c等，但不匹配file.h或file.s等。
+(模式表)| 匹配给定模式表中“模式”的1次或多次出现。例如，file+(.c &#124; .o)匹配文件file.c、file.o、file.c.o、file.c.c等，但不匹配file。
?(模式表)| 匹配模式表中任何一种“模式”的0次或1次出现。例如，file?(.c &#124; .o)只匹配file、file.c和file.0，它不匹配多个模式或模式的重复出现，即不匹配file. c. c、file. c. 0等。
@(模式表)|仅匹配模式表中给定“模式”的一次出现。例如，file@(.c &#124; .0)匹配file.c和file.0，但不匹配file、file.c.c、file.c.o等。
!(模式表)| 除给定模式表中的一个“模式”之外，它可以匹配其它任何东西。

模式表中各模式之间以“|”分开