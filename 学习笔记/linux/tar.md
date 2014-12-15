###解压

语法：tar [主选项+辅选项] 文件或者目录

使用该命令时，主选项是必须要有的，它告诉tar要做什么事情，辅选项是辅助使用的，可以选用。

###主选项：

c 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。相当于打包。

x 从档案文件中释放文件。相当于拆包。

t 列出档案文件的内容，查看已经备份了哪些文件。

特别注意，在参数的下达中， c/x/t 仅能存在一个！不可同时存在！因为不可能同时压缩与解压缩。

###辅助选项：

-z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩或解压？ 一般格式为xx.tar.gz或xx. tgz

-j ：是否同时具有 bzip2 的属性？亦即是否需要用 bzip2 压缩或解压？一般格式为xx.tar.bz2  

-v ：压缩的过程中显示文件！这个常用

-f ：使用档名，请留意，在 f 之后要立即接档名喔！不要再加其他参数！

-p ：使用原文件的原来属性（属性不会依据使用者而变）

--exclude FILE：在压缩的过程中，不要将 FILE 打包！

###范例：

* 范例一：将整个 /etc 目录下的文件全部打包成为 /tmp/etc.tar

[root@linux ~]# tar -cvf /tmp/etc.tar /etc　　　　<==仅打包，不压缩！

[root@linux ~]# tar -zcvf /tmp/etc.tar.gz /etc　　<==打包后，以 gzip 压缩

[root@linux ~]# tar -jcvf /tmp/etc.tar.bz2 /etc　　<==打包后，以 bzip2 压缩

***特别注意，在参数 f 之后的文件档名是自己取的，我们习惯上都用 .tar 来作为辨识。***

***如果加 z 参数，则以 .tar.gz 或 .tgz 来代表 gzip 压缩过的 tar file ～***

***如果加 j 参数，则以 .tar.bz2 来作为附档名啊～***

***上述指令在执行的时候，会显示一个警告讯息：***

***『tar: Removing leading `/" from member names』那是关於绝对路径的特殊设定***

 

范例二：查阅上述 /tmp/etc.tar.gz 文件内有哪些文件？

[root@linux ~]# tar -ztvf /tmp/etc.tar.gz

# 由於我们使用 gzip 压缩，所以要查阅该 tar file 内的文件时，

# 就得要加上 z 这个参数了！这很重要的！

 

范例三：将 /tmp/etc.tar.gz 文件解压缩在 /usr/local/src 底下

[root@linux ~]# cd /usr/local/src

[root@linux src]# tar -zxvf /tmp/etc.tar.gz

# 在预设的情况下，我们可以将压缩档在任何地方解开的！以这个范例来说

# 我先将工作目录变换到 /usr/local/src 底下，并且解开 /tmp/etc.tar.gz

# 则解开的目录会在 /usr/local/src/etc ，另外，如果您进入 /usr/local/src/etc

# 则会发现，该目录下的文件属性与 /etc/ 可能会有所不同喔！

 

范例四：在 /tmp 底下，我只想要将 /tmp/etc.tar.gz 内的 etc/passwd 解开而已

[root@linux ~]# cd /tmp

[root@linux tmp]# tar -zxvf /tmp/etc.tar.gz etc/passwd

# 我可以透过 tar -ztvf 来查阅 tarfile 内的文件名称，如果单只要一个文件，

# 就可以透过这个方式来下达！注意到！ etc.tar.gz 内的根目录 / 是被拿掉了！

 

范例五：我要备份 /home, /etc ，但不要 /home/dmtsai

[root@linux ~]# tar --exclude /home/dmtsai -zcvf myfile.tar.gz /home/* /etc

 

另外：tar命令的C参数

 

　　$ tar -cvf file2.tar /home/usr2/file2
　　tar: Removing leading '/' from members names
　　home/usr2/file2
　　该命令可以将/home/usr2/file2文件打包到当前目录下的file2.tar中，需要注意的是：使用绝对路径标识的源文件，在用tar命令压缩后，文件名连同绝对路径（这里是home/usr2/，根目录'/'被自动去掉了）一并被压缩进来。使用tar命令解压缩后会出现以下情况：
　　$ tar -xvf file2.tar
　　$ ls
　　…… …… home …… …… 
　　解压缩后的文件名不是想象中的file2，而是home/usr2/file2。

　　$ tar -cvf file2.tar -C /home/usr2 file2
　　该命令中的-C dir参数，将tar的工作目录从当前目录改为/home/usr2，将file2文件（不带绝对路径）压缩到file2.tar中。注意：-C dir参数的作用在于改变工作目录，其有效期为该命令中下一次-C dir参数之前。
　　使用tar的-C dir参数，同样可以做到在当前目录/home/usr1下将文件解压缩到其他目录，例如：
　　$ tar -xvf file2.tar -C /home/usr2
　　而tar不用-C dir参数时是无法做到的：
　　$ tar -xvf file2.tar /home/usr2
　　tar: /tmp/file: Not found in archive
　　tar: Error exit delayed from previous errors
