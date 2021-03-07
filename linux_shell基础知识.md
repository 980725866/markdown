
##### #!/bin/bash
```bash
#!告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序
#!/bin/bash
#!/bin/sh
```

运行Shell脚本方法
```bash
bash example.sh
source example.sh
./example.sh
```

##### 定义变量
```bash
pwd="pwd"
echo $pwd
```

##### 环境变量
使用export导出环境变量


##### Shell字符串
双引号允许shell解释字符串中出现的特殊字符。单引号不会对其做任何解释

```bash
var="hello world"
# 获取变量值的长度
length=${#var}
echo $length

# 拼接字符串
string1="world"
string2="hello, "$string1" !"
string3="hello, ${string1} !"
echo $string2
echo $string3
```

##### 字符串运算符
| 运算符 | 说明 | 举例 |
| ------| ---- | ---- |
| =     | 检测两个字符串是否相等，相等返回 true    |    [ $a = $b ]  |
| !=    | 检测两个字符串是否不相等，不相等返回 true   |   [ $a != $b ]   |
| -z    | 检测字符串长度是否为0，为0返回 true	| [ -z $a ] |
| -n    | 检测字符串长度是否不为 0，不为 0 返回 true |	[ -n "$a" ] |
| $	    | 检测字符串是否为空，不为空返回 true	| [ $a ] |


##### 数组
```bash
# 1 定义一个数组
array_var1=(test1 test2 test3 test4)
# 或者
array_var2=(
value0
value1
value2
value3
)

# 赋值
array_var1[0]="tmp1"
array_var1[1]="tmp2"
array_var1[2]="tmp3"
array_var1[3]="tmp4"

# 2 打印数组元素内容
echo ${array_var1[0]}

# 3 打印数组中的所有值
echo ${array_var1[*]}
echo ${array_var1[@]}

# 4 打印数组长度
echo ${#array_var1[*]}

```

关联数组
declare -A ass_array
ass_array['A']="AAA"
ass_array['B']="BBB"
echo $ass_array['A']

列出数组索引
echo ${!ass_array[*]}


##### shell中的数学运算
1, let
2, (( ))
3, []
4, expr
number1=1
number2=2

let result1=number1+number2
echo $result1

result2=$(( number1 + number2 ))

result3=$[ number1 + number2 ]

result4=`expr $number1 + $number2`





##### 函数的定义和使用





Shell 传递参数
我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：$n。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……
参数处理	说明
$#	传递到脚本的参数个数
$*	以一个单字符串显示所有向脚本传递的参数。
如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$$	脚本运行的当前进程ID号
$!	后台运行的最后一个进程的ID号
$@	与$*相同，但是使用时加引号，并在引号中返回每个参数。
如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
$-	显示Shell使用的当前选项，与set命令功能相同。
$?	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。


运算符	说明	举例
+	加法	`expr $a + $b` 结果为 30。
-	减法	`expr $a - $b` 结果为 -10。
*	乘法	`expr $a \* $b` 结果为  200。
/	除法	`expr $b / $a` 结果为 2。
%	取余	`expr $b % $a` 结果为 0。
=	赋值	a=$b 将把变量 b 的值赋给 a。
==	相等。用于比较两个数字，相同则返回 true。	[ $a == $b ] 返回 false。
!=	不相等。用于比较两个数字，不相同则返回 true。	[ $a != $b ] 返回 true。

关系运算符
关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

运算符	说明	举例
-eq	检测两个数是否相等，相等返回 true。	[ $a -eq $b ] 返回 false。
-ne	检测两个数是否不相等，不相等返回 true。	[ $a -ne $b ] 返回 true。
-gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ] 返回 false。
-lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ] 返回 true。
-ge	检测左边的数是否大于等于右边的，如果是，则返回 true。	[ $a -ge $b ] 返回 false。
-le	检测左边的数是否小于等于右边的，如果是，则返回 true。	[ $a -le $b ] 返回 true。

布尔运算符
下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

运算符	说明	举例
!	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ] 返回 true。
-o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ] 返回 false。

文件测试运算符
文件测试运算符用于检测 Unix 文件的各种属性。

属性检测描述如下：

操作符	说明	举例
-b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。
-c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -c $file ] 返回 false。
-d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ] 返回 false。
-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。
-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] 返回 false。
-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。
-p file	检测文件是否是有名管道，如果是，则返回 true。	[ -p $file ] 返回 false。
-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。
-r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。
-w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。
-x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。
-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。
-e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。



Shell test 命令
Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

数值测试
参数	说明
-eq	等于则为真
-ne	不等于则为真
-gt	大于则为真
-ge	大于等于则为真
-lt	小于则为真
-le	小于等于则为真

字符串测试
参数	说明
=	等于则为真
!=	不相等则为真
-z 字符串	字符串的长度为零则为真
-n 字符串	字符串的长度不为零则为真

if else

if else-if else

case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac

for 循环
与其他编程语言类似，Shell支持for循环。


Shell 函数


##### Shell传递参数
在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

| 运算符 | 说明 |
| ------| ---- |
| $# |	传递到脚本或函数的参数个数 |
| $* |	以一个单字符串显示所有向脚本传递的参数 |
| $@ |	与$*相同，但是使用时加引号，并在引号中返回每个参数 |
| $? |	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误 |


Shell 输入/输出重定向

1使用大于号将文本保存到文件
echo "this is a sample" > temp.txt

2 使用双大于号将文本追加到文件中
echo "this is a sample" >> temp.txt

alias创建别名
alias new command='orgin cmd

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回​​到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

重定向命令列表如下：

命令	说明
command > file	将输出重定向到 file。
command < file	将输入重定向到 file。
command >> file	将输出以追加的方式重定向到 file。
n > file	将文件描述符为 n 的文件重定向到 file。
n >> file	将文件描述符为 n 的文件以追加的方式重定向到 file。
n >& m	将输出文件 m 和 n 合并。
n <& m	将输入文件 m 和 n 合并。
<< tag	将开始标记 tag 和结束标记 tag 之间的内容作为输入。

sed

awk
