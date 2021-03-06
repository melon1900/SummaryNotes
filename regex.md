现如今正则表达式已然成了程序员的标配，不会用正则都不好意思说自己是程序员，对于linux/unix环境下工作的程序员更是如此，在linux环境下工作的时间越长越离不开正则表达式。最近两年用正则用的偏多一些，踩过一些坑，今天给大家分享一下，希望对大家有所帮助。

**我的正则表达式在这个工具语言下好使，为什么换个工具/语言同样的正则表达式就不好使了呢？**

比如我们知道'\d{3}'是匹配3个连续数字的正则表达式
```
[root@bogon ~]# echo '123' | grep '\d{3}'           
```
但在linu下通过grep使用该正则表达式时却并不能匹配3个连续数字，为什么呢？

这主要是正则表达式有多个流派，每个工具或语言所选用的流派不同造成的。正则表达主要有三个流派：基本正则表达式，扩展正则表达式，perl兼容正则表达式。基本正则和扩展正则是POSIX标准制定的，所有POSIX程序可以选择其中的一种，所以说linux下的支持正则表达式的命令要么支持基本正则，要么支持扩展正则，或者两者都支持，像我们常用的grep和sed命令，基本正则和扩展正则都是支持的（grep通过-E选项支持扩展正则，sed通过-r选项支持扩展正则），awk支持扩展正则。但大部分高级语言或脚本语言中以perl兼容正则为主，比如Perl，Java，C#，JS，Python，Ruby，php（扩展正则在php5.3中已废弃）等，而且每种语言还会有一些具体的细微差别。值得一提的是有的语言中的正则这三个流派都不属于，使用的是自己语言特有，比如lua，当你使用这类语言的时候就要特别注意了。

三个主要流派基本规范对比如下：

正则表达式特性 | 基本正则 | 扩展正则 | Perl兼容正则
---|---|---|---
点，\^，$，[...]，[\^...]，* | 支持 | 支持 | 支持
+和? 量词 | POSIX标准不支持，但GNU Linux下作了扩展，可以通过\\+和\\?的方式使用 | 支持 | 支持
区间量词 | \\{min,max\\} | {min, max}| {min, max}
分组 | \\(...\\) | (...)| (...)
量词可否作用于分组 | 可以 | 可以| 可以
分组反向引用 | \\1到\\9 | \\1到\\9| \\1到\\9
选择 | POSIX标准不支持，但GNU Linux下作了扩展，可以通过 ```\|```的方式使用 | ```|``` | ```|```
命名分组 | 不支持 | 不支持 | (?<name>...)
命名分组反向引用 | 不支持 | 不支持 | \k<name> 例如acac会匹配```(?<xx>[ab]c)\k<xx>'```
lazy匹配模式（和贪婪模式对应，默认都是贪婪模式） | 不支持 | 在量词后面加?，即在*，+，?，{min,max}，{n}后面跟上? | 同扩展正则
断言子组 | 不支持 | 不支持 | (?=exp)，(?!exp)，（?<=exp)，(?<!exp) 
[入门教程](http://deerchao.net/tutorials/regex/regex.htm)

字符组对比

该图截取自维基百科，[详情请看](https://en.wikipedia.org/wiki/Regular_expression#Character_classes)

还有一点需要特别注意的是：同一正则表达式，由于所用语言或工具的版本不同也可能会得到不一样的结果，比如对于命名分组grep 2.5.1版本就不支持，但2.6.3版本就已支持了。

知道了这些差异，在不同语言或工具中切换使用正则表达式时就可以得心应手了。在可以使用perl兼容正则的情况下，尽量使用perl兼容正则，因为你可读性更好，功能更完善。

**转义，转迷糊了**

如果你要匹配的字符是正则表达式的元字符，那么就需要转义，正则表达式和字符串一样也使用反斜线来表示转义序列。我们看一个例子：

```
[root@bogon ~]# grep "^a\\\\\\*b"
a\*b
a\*b

[root@bogon ~]# grep '^a\\\*b'               
a\*b
a\*b
```
也就是说
```
"^a\\\\\\*b"
```
和

```
'^a\\\*b'
```
是等效的，为什么呢？

首先就要说一下shell中双引号和单引号的区别：单引号和双引号都能关闭shell对特殊字符的处理。不同的是，双引号没有单引号严格，单引号关闭所有有特殊作用的字符，而双引号只要求shell忽略大多数，具体 的说，就是$,`(反引号),\，这3种特殊字符不被忽略。也就是在shell中被单引号引起来的字符串任何转义序列都是无效的，它所表示的字符串就是它本身，而被双引号引起来的字符串表示的并不是它本身是需要做转义翻译的。

所以上面两条命令等效的原因就是
```
"^a\\\\\\*b"
```
先在shell层面被转义为
```
^a\\\*b
```
，最后在正则表达式层面被转义为
```
a\*b
```
正式因为这一点，在shell中使用正则表达式时，一般都使用单引号，但有时你还必须要使用双引号引用的字符串，比如当你需要匹配单引号时，所以清楚的了解这一点还是很有必要的。

其它可以既可以使用单引号，也可以使用双引号引用字符串的语言，处理方式和shell的这种处理方式基本一致，比如php。


练习工具

如果你的工作环境是linux，可以使用grep工具通过命令行交互的方式来练习正则表达式，如果你输入的字符串和你标示的正则表达式匹配的话，它会将你输入的字符串重复打印出来（见下面例子）。你还可以通过命令行选项来控制使用哪个流派的正则表达式语法，默认使用基本正则的语法，加上-E选项使用扩展正则语法（等同于直接使用egrep），加上-P选项使用Perl兼容正则。

```
[root@bogon ~]# grep -P '\d{3,5}'
aa
123
123
123456
123456
12
```
该例是要匹配含有3-5个连续数字的字符串，所以aa和12不匹配，123和123456匹配。

