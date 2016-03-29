---
title: Linux_命令-awk
date: 2016-01-17 15:31:00
categories: 技术
tags: [linux]
description: awk - 操作详细说明
---

ref: http://man.linuxde.net/awk
ref: http://rachzhang.iteye.com/blog/900811

## 常规操作

### 命令基本结构

```sh
awk 'BEGIN{ print "start" } pattern{ commands } END{ print "end" }' file
```

一个awk脚本通常由：BEGIN语句块、能够使用模式匹配的通用语句块、END语句块3部分组成，这三个部分是可选的，任意一个部分都可以不出现在脚本中。

- 第一步：执行BEGIN{ commands }语句块中的语句；
- 第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ commands }语句块，它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。
- 第三步：当读至输入流末尾时，执行END{ commands }语句块。

BEGIN语句块在awk开始从输入流中读取行之前被执行，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中。

END语句块在awk从输入流中读取完所有的行之后即被执行，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。

pattern语句块中的通用命令是最重要的部分，它也是可选的。如果没有提供pattern语句块，则默认执行{ print }，即打印每一个读取到的行，awk读取的每一行都会执行该语句块。

### 内置变量


| 变量名 | 作用 |
|-------|------|
| \$n | 当前记录的第n个字段，比如n为1表示第一个字段，n为2表示第二个字段。|
| \$0 | 这个变量包含执行过程中当前行的文本内容。|
|NF | 表示字段数，在执行过程中对应于当前的字段数。|
|NR | 表示记录数，在执行过程中对应于当前的行号。|
|FS | 字段分隔符（默认是任何空格）|
|RS | 记录分隔符（默认是一个换行符) |
|OFS | 输出字段分隔符（默认值是一个空格） |
|ORS | 输出记录分隔符（默认值是一个换行符） |

### 高级操作 - next

在循环逐行匹配，如果遇到next，就会跳过当前行，直接忽略下面语句。而进行下一行匹配。net语句一般用于多行合并：

```sh
$ cat text.txt
web01[192.168.2.100]
httpd ok
tomcat ok sendmail ok
web02[192.168.2.101]
httpd ok
postfix ok
web03[192.168.2.102]
mysqld ok
httpd ok 0
$ awk '/^web/{T=$0;next;}{print T":t"$0;}' test.txt
web01[192.168.2.100]: httpd ok
web01[192.168.2.100]: tomcat ok
web01[192.168.2.100]: sendmail ok
web02[192.168.2.101]: httpd ok
web02[192.168.2.101]: postfix ok
web03[192.168.2.102]: mysqld ok
web03[192.168.2.102]: httpd ok
```



## 数组应用

### 数组索引（下标）可以是数字和字符串

**数组下标是从1开始，与C数组不一样。**

读取数组的值
```
 { for(item in array) {print array[item]}; } #输出的顺序是随机的 
 { for(i=1;i<=length(array);i++) {print array[i]}; } #Len是数组的长度
```

### 判断键值存在以及删除键值

错误的判断方法 - 坑：

```sh
$ awk 'BEGIN{tB["a"]="a1";tB["b"]="b1";if(tB["c"]!="1"){print "no found";};for(k in tB){print k,tB[k];}}'
no found
a a1
b b1
c
```
以上出现奇怪问题，tB[“c”]没有定义，但是循环时候，发现已经存在该键值，它的值为空，这里需要注意，awk数组是关联数组，只要通过数组引用它的key，就会自动创建改序列。

正确判断方法： 

```sh
$ awk 'BEGIN{tB["a"]="a1";tB["b"]="b1";if( "c" in tB){print "ok";};for(k in tB){print k,tB[k];}}'
a a1
b b1
```

if(key in array)通过这种方法判断数组中是否包含key键值。

### 二维数组


## 内置函数
awk内置函数，主要分以下3种类似：算数函数、字符串函数、其它一般函数、时间函数。

### 算数函数

| 格式 | 描述 |
|----|----|
| atan2(y, x)|返回 y/x 的反正切。|
| cos(x) | 返回 x 的余弦；x 是弧度。|
| sin(x) | 返回 x 的正弦；x 是弧度。|
| exp(x) | 返回 x 幂函数。| 
| log(x) | 返回 x 的自然对数。|
| sqrt(x) | 返回 x 平方根。|
| int(x) | 返回 x 的截断至整数的值。|
| rand() | 返回任意数字 n，其中 0 <= n < 1。|
| srand([expr]) | 将 rand 函数的种子值设置为 Expr 参数的值，或如果省略 Expr 参数则使用某天的时间。返回先前的种子值。 |

### 字符串函数

### 一般函数

### 时间函数

## 案例

### 输出最后一列

```sh
$ cat tmp.txt | awk '{print $NF}'
```

### 统计某行平均值

```sh
$ awk '{sum+=$1}END{print sum/NR}' test.txt
```

### 指定输出列分隔符
```sh
$ awk '{print}' OFS="|" test.txt
```

### 指定输入内容分隔符

```sh
$ awk 'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd
$ awk -F':' '{print $1,$3,$6}' /etc/passwd
$ ## 多个分隔符
$ awk -F'[:;]' '{print $1,$3,$6}' /etc/passwd
```

### 去重计数

```sh
$ awk '{a[$2]++}END{for(i in a) print i,a[i]}' | sort -
```

### 文件合并

```
🍺 /Users/baidu/work/tmp ]$cat tmp.txt
1 aaaaa
2 bbbbb
3 ccccc
4 ddddd
5 eeeee
🍺 /Users/baidu/work/tmp ]$cat tmp2.txt
11 test2 2aaaaa
3 test2 2ccccc
2 test2 2bbbbb
14 test2 2ddddd
15 test2 2eeeee
🍺 /Users/baidu/work/tmp ]$awk '{if(NR==FNR){a[$1]=$2}else{a[$1]=a[$1]" "$0}}END{for(i in a)print i,a[i];}' tmp.txt tmp2.txt
 2 test2 2bbbbb
 3 test2 2ccccc
4 ddddd
5 eeeee
11  11 test2 2aaaaa
14  14 test2 2ddddd
15  15 test2 2eeeee
1 aaaaa
```
