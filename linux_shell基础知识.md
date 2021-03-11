
##### #!/bin/bash
```bash
#!告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序
#!/bin/bash
#!/bin/sh
```

##### 运行Shell脚本方法
```bash
bash example.sh
source example.sh
./example.sh
```

##### 定义变量
```bash
# 变量和等于号之间不能有空格
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

```bash
str1="hello"
str2="world"
# =
if [ $str1 = $str2 ]; then
    echo "str1 == str2"
else
    echo "str1 != str2"
fi

# !=
if [ $str1 != $str2 ]; then
    echo "str1 != str2"
else
    echo "str1 == str2"
fi

# -z
if [ -z $str1 ]; then
    echo "str1 == null"
else
    echo "str is not null"
fi

# -n
if [ -n $str1 ]; then
    echo "str1 is not null"
else
    echo "str is null"
fi

# $
if [ $str1 ]; then
    echo "str1 is not null"
else
   echo "str1 is null"
fi

```

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

##### 关联数组
```bash
declare -A ass_array
ass_array['A']="AAA"
ass_array['B']="BBB"

# 列出数组索引
echo ${!ass_array[*]}

# 获取数组长度
echo ${#array_tmp[*]}

# 通过key获取value
echo ${array_tmp['A']}

# 获取所有的key
echo ${!array_tmp[*]}
```

##### shell中的数学运算
| 运算符 | 说明 |
| ------| ---- |
| +	加法 |	`expr $a + $b` |
| -	减法 | `expr $a - $b` |
| *	乘法 |	`expr $a \* $b` |
| /	除法 |	`expr $b / $a` |
| %	取余 |	`expr $b % $a` |
| =	赋值 |	a=$b 将把变量 b 的值赋给 a |
| == 相等 | 用于比较两个数字，相同则返回 true |
| != 不相等 | 用于比较两个数字，不相同则返回 true |
```bash
number1=10
number2=5
# let
let result1=number1+number2
echo $result1

# (( ))
result2=$(( number1 + number2 ))
echo $result2

# []
result3=$[ number1 + number2 ]
echo $result3

# expr
result4=`expr $number1 + $number2`
echo $result4
```




##### 函数的定义和使用

##### Shell传递参数
在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...
| 运算符 | 说明 |
| ------| ---- |
| $# |	传递到脚本或函数的参数个数 |
| $* |	传递给脚本或函数的所有参数 |
| $@ |	与$*相同，但是使用时加引号，并在引号中返回每个参数, "$\*" 会将所有的参数作为一个整体 |
| $? |	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误 |

```bash
function hello()
{
    echo "hello -->"
    echo $#
    for arg in $*
    do
        echo $arg
    done
}
# 调用函数
hello test1
hello test1 test2

function getInt()
{
    echo "getInt"
    return 1;
}
# 调用函数
getInt
# 获取返回值
result=$?
echo $result
```

##### 关系运算符
关系运算符只支持数字，不支持字符串，除非字符串的值是数字。
| 运算符 | 说明 | 举例 |
| ------| ---- |---- |
| -eq	| 检测两个数是否相等，相等返回 true |	[ $a -eq $b ] |
| -ne	| 检测两个数是否不相等，不相等返回 true |	[ $a -ne $b ] |
| -gt	| 检测左边的数是否大于右边的，如果是，则返回 true |	[ $a -gt $b ] |
| -lt	| 检测左边的数是否小于右边的，如果是，则返回 true |	[ $a -lt $b ] |
| -ge	| 检测左边的数是否大于等于右边的，如果是，则返回 true |	[ $a -ge $b ] |
| -le	| 检测左边的数是否小于等于右边的，如果是，则返回 true | [ $a -le $b ] |

```bash
a=10
b=20
if [ $a -eq $b ]
then
   echo "$a -eq $b : a == b"
else
   echo "$a -eq $b: a != b"
fi

if [ $a -ne $b ]
then
   echo "$a -ne $b: a != b"
else
   echo "$a -ne $b : a == b"
fi
```

##### 布尔运算符
| 运算符 | 说明 |
| ------| ---- |
| !	非运算 | 表达式为 true 则返回 false，否则返回 true |
| -o	或运算 | 有一个表达式为 true 则返回 true |
| -a	与运算 | 两个表达式都为 true 才返回 true |
```bash
a=10
b=20
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
```

#### Shell test 命令
Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。
##### 数值测试
| 参数 | 说明 |
| ------| ---- |
| -eq	| 等于则为真 |
| -ne	| 不等于则为真 |
| -gt	| 大于则为真 |
| -ge	| 大于等于则为真 |
| -lt	| 小于则为真 |
| -le	| 小于等于则为真 |
```bash
a=10
b=20
if test $a -eq $b ; then
    echo "a == b"
else
    echo "a != b"
fi
```


##### 字符串测试
| 参数 | 说明 |
| ------| ---- |
| =	| 等于则为真 |
| != | 不相等则为真 |
| -z 字符串 | 	字符串的长度为零则为真 |
| -n 字符串 | 	字符串的长度不为零则为真 |
```bash
str1='hello'
str2='hello'
if test $str1 = $str2 ; then
    echo "str1 == str2"
else
    echo "str1 != str2"
fi

```

##### 文件测试运算符
文件测试运算符用于检测 Unix 文件的各种属性。
| 操作符 | 说明 | 举例 |
| ------| ---- | ---- |
| -d file	| 检测文件是否是目录，如果是，则返回 true |	[ -d $file ] |
| -f file	| 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true |	[ -f $file ] |
| -s file	| 检测文件是否为空（文件大小是否大于0），不为空返回 true |	[ -s $file ] |
| -e file	| 检测文件（包括目录）是否存在，如果是，则返回 true |	[ -e $file ] |

```bash
file="hello.txt"
if test -e $file ; then
    echo "$file is exist"
else
    echo "$file is not exist"
fi
```

##### Shell流程控制
```bash
# if else
file="hello.txt"
if test -e $file ; then
    echo "$file is exist"
else
    echo "$file is not exist"
fi

# if else if
score=85
if [ $score -gt 80 ]; then
  echo "A"
elif [ $score -gt 60 ]; then
  echo "B"
else
  echo "C"
fi

# case
score=100
case $score in
1) echo "1";;
2) echo "2";;
3) echo "3";;
*) echo "default";;
esac
```

##### 循环
```bash
# for
for i in 1 2 3 4 5
do
    echo $i
done

# for
for((i=0; i<10 ; i++))
do
    echo $i
done

# while
count=0
while(( $count < 10 ))
do
    echo $count
    let count=count+1
done
```

##### Shell 输入/输出重定向：
| 命令 | 说明 |
| ------| ---- |
| command > file	| 将输出重定向到 file |
| command < file	| 将输入重定向到 file |
| command >> file	| 将输出以追加的方式重定向到 file |
| n >& m	| 将输出文件 m 和 n 合并 |
| n <& m	| 将输入文件 m 和 n 合并 |

```bash
# 使用大于号将文本保存到文件
echo "this is a sample" > temp.txt

# 使用双大于号将文本追加到文件中
echo "this is a sample" >> temp.txt
```

##### sed简单使用
sed的命令格式: sed [options] 'command' file(s)
- -i 直接修改文件内容
- d 删除，删除选择的行
- s 替换指定字符
- g 表示可以使sed执行全局替换
```bash
# 1 替换
sed -i 's/pattern/replace/' file

# 2 替换
sed -i 's/pattern/replace/g' file

# 3 sed命令会将s之后的字符视为命令分隔符
sed 's:text:replace:g'

# 删除空行
sed -i '/^$/d' file
```


##### awk简单使用
awk以每行的形式处理文件
| 内部变量 | 说明 |
| ------| ---- |
| NF    |              浏览记录的域的个数 |
| NR    |              已读的记录数 |
| OFS   |              输出域分隔符 |
| ORS   |              输出记录分隔符 |
| RS    |             控制记录分隔符 |
| $0变量是指整条记录 | $1表示当前行的第一个域,$2表示当前行的第二个域,......以此类推|
```bash
# 1 awk结构
awk 'BEGIN{ print "start"} pattern { command }  END{ print "end"}' file

# 2默认的字段分割符是空格
awk '{print $1, $2}' file

# 3打印行号
awk 'END { print NR}'


# 4 默认的字段分割符是空格，也可以用选项-F指定不同的分隔符
awk -F: '{ print $NF}' file
```
