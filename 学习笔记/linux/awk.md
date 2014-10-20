AWK常用命令
=================
###文本操作
* 打包当前目录下的所有文件:

`ls | awk '{ print "tar zcvf "$0".tar.gz " $0|"/bin/bash" }'`

* 指定分隔符

`echo "abc#1233+232@jjjj?===" |awk -F '[#@]' '{print $2}'`

`1233+232`

`echo "abc#1233+232@jjjj?===" |awk -F '[#?]' '{print $2}'`

`1233+232@jjjj`

* 匹配

匹配非空行：

`awk '/^[^$]/ {print $0}' test.txt`

匹配非包含zh：

`awk '/^[^zh]/ {print $0}' test.txt` 

* 替换(将:替换为#)

`echo "x:y:1:2:3" | awk 'gsub(/:/,"#"){print $0}'`

`x#y#1#2#3`

###数值运算
* 列运算

>test.txt文档内容:

>1

>2

>3

列求和： `cat test.txt |awk '{a+=$1}END{print a}'`

列求平均值：`cat test.txt |awk '{a+=$1}END{print a/NR}'`

列求最大值：`cat test.txt |awk 'BEGIN{a=0}{if ($1>a) a=$1 fi}END{print a}'`

(**BEGIN用于初始化。设定一个变量开始为0，遇到比该数大的值，就赋值给该变量，直到结束**)

列求最小值：`cat test.txt |awk 'BEGIN{a=11111}{if ($1<a) a=$1 fi}END{print a}'`


* 全文最值

例1：求test.txt的最值

>12 34 56 78

>24 65 87 90

>76 11 67 87

>100 89 78 99

求最大值及位置：`for i in 'cat test.txt';do echo $i; done |sort |sed -n '1p;2p'`  

例2：同样是test.txt

求总和：`for i in 'cat test.txt';do echo $i ;done |awk '{a+=$1}END{print a}'`

例3：

>A     88

>B     78

>B     89

>C     44

>A     98

>C     433

要求输出：

>A：88；98

>B：78；89
          
>C：44；433
          
`awk '{a[$1]=a[$1]" "$2}END{for(i in a)print i,a[i]}' test.txt |awk '{print $1":",$2";",$3}'`
