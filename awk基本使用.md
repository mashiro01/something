---
note:
  createdAt: 2020-06-19T03:26:27.410Z
  modifiedAt: 2020-06-21T06:59:12.684Z
  tags: []
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->

<!-- code_chunk_output -->

- [AWK 基础学习](#awk-基础学习)
  - [介绍](#介绍)
  - [基本使用](#基本使用)

<!-- /code_chunk_output -->

# AWK 基础学习

## 介绍

**AWK** 是一种处理文本文件的语言，是一个强大的文本分析工具。

> 之所以叫 AWK 是因为其取了三位创始人 Alfred Aho，Peter Weinberger, 和 Brian Kernighan 的 Family Name 的首字符。

## 基本使用

现在有一段文本 **coin.txt**

```text
gold 1 1986 USA American Eagle
gold 1 1908 Austria-Hungary Franz josef 100 Korona
silver 10 1981 USA ingot
gold 1 1984 Switzerland ingot
gold 1 1979 RSA Krugerrand
gold 0.5 1981 RSA Krugerrand
gold 0.1 1986 PRC Panda
silver 1 1986 USA Liberty dollar
gold 0.25 1986 USA Liberty 5-dollar piece
silver 1 1986 USA Liberty 50-cent piece
silver 1 1987 USA Constitution dollar
gold 0.25 1987 USA Constitution 5-dollar piece
gold 1 1988 Canada Maple leaf
```

最基础的，要把整段文本输出

```bash
awk '{print}' coin.txt
```

awk 前置，中段 `{print}` 为 awk 的默认语句，用于控制输出，其后为输入流

再次，要把此段文本中的某一列进行输出，可以执行

```bash
awk '{print $1}' coin.txt
```

**\$1**表示此段文本的第一列（awk 中以`空格`为分隔符），以此类推，**\$n**表示文本的第 n 列。`（$0代表整段文本）`

可以看到输出

```text
gold
gold
silver
gold
gold
gold
gold
silver
gold
silver
silver
gold
gold
```

此外，在 awk 中，**","** 代表空格，且同时支持 **"\n"** 等符号的使用

```bash
awk '{print $1, $2, $3}' coins.txt
```

```text
gold 1 1986
gold 1 1908
silver 10 1981
gold 1 1984
gold 1 1979
gold 0.5 1981
gold 0.1 1986
silver 1 1986
gold 0.25 1986
silver 1 1986
silver 1 1987
gold 0.25 1987
gold 1 1988
```

```bash
awk '{print NR "\t" $1 "\t" $2 "\t" $3}' coins.txt
```

```text
1       gold    1       1986
2       gold    1       1908
3       silver  10      1981
4       gold    1       1984
5       gold    1       1979
6       gold    0.5     1981
7       gold    0.1     1986
8       silver  1       1986
9       gold    0.25    1986
10      silver  1       1986
11      silver  1       1987
12      gold    0.25    1987
13      gold    1       1988
```

不仅如此，awk 内置了`NF`，`NR`等统计用的符号

`NR`可以用来统计行号

```bash
awk '{print NR "\t" $1 "\t" $2 "\t" $3}' coins.txt
```

```text
1       gold    1       1986
2       gold    1       1908
3       silver  10      1981
4       gold    1       1984
5       gold    1       1979
6       gold    0.5     1981
7       gold    0.1     1986
8       silver  1       1986
9       gold    0.25    1986
10      silver  1       1986
11      silver  1       1987
12      gold    0.25    1987
13      gold    1       1988
```

可以看到每一行的前边都有对应的行号

`NF`可以用来统计列数

```bash
awk '{print NF , $0}' coins.txt
```

```text
6 gold 1 1986 USA American Eagle
8 gold 1 1908 Austria-Hungary Franz josef 100 Korona
5 silver 10 1981 USA ingot
5 gold 1 1984 Switzerland ingot
5 gold 1 1979 RSA Krugerrand
5 gold 0.5 1981 RSA Krugerrand
5 gold 0.1 1986 PRC Panda
6 silver 1 1986 USA Liberty dollar
7 gold 0.25 1986 USA Liberty 5-dollar piece
7 silver 1 1986 USA Liberty 50-cent piece
6 silver 1 1987 USA Constitution dollar
7 gold 0.25 1987 USA Constitution 5-dollar piece
6 gold 1 1988 Canada Maple leaf
```

此外，在进行输出时，awk 还支持条件判断输出

```bash
awk '$3 == 1986 {print $0}' coins.txt
```

```text
gold 1 1986 USA American Eagle
gold 0.1 1986 PRC Panda
silver 1 1986 USA Liberty dollar
gold 0.25 1986 USA Liberty 5-dollar piece
silver 1 1986 USA Liberty 50-cent piece
```

```bash
awk '$1 == "gold" {print $0}' coins.txt
```

```text
gold 1 1986 USA American Eagle
gold 1 1908 Austria-Hungary Franz josef 100 Korona
gold 1 1984 Switzerland ingot
gold 1 1979 RSA Krugerrand
gold 0.5 1981 RSA Krugerrand
gold 0.1 1986 PRC Panda
gold 0.25 1986 USA Liberty 5-dollar piece
gold 0.25 1987 USA Constitution 5-dollar piece
gold 1 1988 Canada Maple leaf
```

可以方便地进行数据统计

不仅如此，利用 `BEGIN{}`关键字，我们还能实现自定义分隔符以及需要划分的符号

```bash
awk 'BEGIN{FS=":"; OFS=" "} {print $1 "\t" $2 "\t" $3}' /etc/passwd
```

```text
root    x       0
daemon  x       1
bin     x       2
sys     x       3
sync    x       4
games   x       5
man     x       6
lp      x       7
mail    x       8
news    x       9
uucp    x       10
proxy   x       13
www-data        x       33
backup  x       34
list    x       38
irc     x       39
gnats   x       41
nobody  x       65534
systemd-network x       100
systemd-resolve x       101
systemd-timesync        x       102
messagebus      x       103
syslog  x       104
_apt    x       105
tss     x       106
uuidd   x       107
tcpdump x       108
sshd    x       109
landscape       x       110
pollinate       x       111
```

非常容易地就完成了文本的划分以及重排版

同时 awk 不仅能够操纵一个文件，同时也能操作多个文件，文件内容依次拼接到上一段的末尾

```bash
awk '{print NR, FILENANE, $0}' coins.txt data.txt
```

```text
1  gold 1 1986 USA American Eagle
2  gold 1 1908 Austria-Hungary Franz josef 100 Korona
3  silver 10 1981 USA ingot
4  gold 1 1984 Switzerland ingot
5  gold 1 1979 RSA Krugerrand
6  gold 0.5 1981 RSA Krugerrand
7  gold 0.1 1986 PRC Panda
8  silver 1 1986 USA Liberty dollar
9  gold 0.25 1986 USA Liberty 5-dollar piece
10  silver 1 1986 USA Liberty 50-cent piece
11  silver 1 1987 USA Constitution dollar
12  gold 0.25 1987 USA Constitution 5-dollar piece
13  gold 1 1988 Canada Maple leaf
14
15  hello world
16  apple banana orange
17  abcd efgh
```

`NR`，`NF`作为内置的可变变量，其也能作为固定变量被使用，只要在之前加上 **\$** 修饰即可

再之，作为一门脚本向的语言，其也能进行正常的数学运算

```bash
awk '{a=1;b=1; print (a + b)}'
awk '{a=1;b=1; print (a - b)}'
awk '{a=1;b=1; print (a * b)}'
awk '{a=1;b=1; print (a / b)}'

awk '{a=1;b=1; print (a % b)}'
```

进行数学运算或者字符串的拼接均可使用（注意若一为字符串，一为数字，其会寻找字符串中的数字进行类型转换，否则当作 0 来处理）

最后需要了解的就是 awk 中利用 **正则表达式** 进行文本的寻找

<!--
#TODO: 学习正则表达式相关内容，并将剩下的文本进行补充，修订，保存
-->
