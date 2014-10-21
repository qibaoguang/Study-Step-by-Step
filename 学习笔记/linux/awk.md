AWK常用命令
=================
###文本操作
* 打包当前目录下的所有文件:

`ls | awk '{ print "tar zcvf "$0".tar.gz " $0|"/bin/bash" }'`

* 指定分隔符

`echo "abc#1233+232@jjjj?===" |awk -F '[#@]' '{print $2}'`

输出：`1233+232`

`echo "abc#1233+232@jjjj?===" |awk -F '[#?]' '{print $2}'`

输出：`1233+232@jjjj`

* 匹配

匹配非空行：

`awk '/^[^$]/ {print $0}' test.txt`

匹配非包含zh：

`awk '/^[^zh]/ {print $0}' test.txt` 

* 替换(将:替换为#)

`echo "x:y:1:2:3" | awk 'gsub(/:/,"#"){print $0}'`

输出：`x#y#1#2#3`

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

###日志分析

* 统计Jmeter平均响应时间

`egrep --color 'true' 1.jtl |awk -F\" 'BEGIN{load=0;latency=0;count=0;}{load+=$2;latency+=$4;count+=1}END{print "css:",load/count,latency/count,latency/load}'`

* 统计Tomcat访问日志内的各种访问请求的响应时间并且排序
 
`grep 'PimSearch' logs/localhost_access_log.2011-12-02.txt |awk '{print $7"\t"$13}'|awk '{url=$1;time=$2;split(url,arr,"?");url=arr[1];len=split(url,arr,"/");dict[arr[3]"-"arr[len]]+=time;count[arr[3]"-"arr[len]]+=1;}END{for (url in dict)print url"\t"dict[url]/count[url]}'|sort -k2 -n -r`

* 统计Tomcat访问日志平均请求，响应时间

access_log.2014-10-15.09.log日志内容：

>`127.0.0.1 124.126.148.39 [20/Oct/2014:09:55:30 +0800] HTTP/1.0 - POST /c/pay HTTP/1.0 429 2795 200`
>
>`127.0.0.1 124.193.219.14 [20/Oct/2014:09:58:19 +0800] HTTP/1.0 - POST /c/beg HTTP/1.0 567 1578 200`

统计平均请求，响应时间，针对所有请求：

`awk '{print $8 "\t" $10 "\t" $11 "\t"}' access_log.2014-10-15.09.log | awk '{a[$1]+=$2;b[$1]+=$3;c[$1]++}END{for(n in c)print n,a[n]/c[n],b[n]/c[n]}'`

统计平均请求，响应时间，针对非后缀请求(资源.js等)：
`awk '{if(index($8,".")==0) print}' access_log.2014-10-15.*.log | awk '{print $8 "\t" $10 "\t" $11 "\t"}'  | awk '{a[$1]+=$2;b[$1]+=$3;c[$1]++}END{for(n in c)print n,a[n]/c[n],b[n]/c[n]}'`
