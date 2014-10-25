Java1.7中String内部表示的变化
========================
###Java1.7之前String的内部表示
Java1.7之前，String内部有４个非静态的域：
>char[] value;　//存放字符串，多个字符串可以共享
>
>int offset;	   //value中被使用的第一个字符索引
>
>int count;   //value中被使用的字符数量
>
>int hash;　//缓存的String hash码

在很多情况下，一个String的offset总是为０，而count为value.length。唯一例外的情况是，通过调用String.substring或内部使用该方法的API创建的字符串。

String.substring创建的字符串会共享原字符串内部的char[] value，这样做允许你：

1. 通过共享字符数据节省内存。

2. 以常量时间（Ｏ(1)）运行String.substring。

于此同时，这种特性可能是造成内存泄漏的罪魁祸首：如果你从一个巨大的字符串提取一个小的子串并丢弃原字符串，你将仍持有一个巨大的对原字符串的char[] value的活引用。避免这种情况的唯一方式是调用new String(String)构造器创建一个新的String－该过程会拷贝底层char[]中需要的部分，而不是短字符串链接到长的父字符串。

从Java 1.7.0_06，包括目前的Java 8版本，String类中的offset和count字段已被删除。这意味着你不能再共享底层的char[] value了！上面描述的内存泄漏也不会出现，你也不用通过new String(String)构造器重新创建新字符串了！这样做的缺点是，你需要记住String.substring的复杂度为线性而不再是常量！

###hash逻辑的变化
**文章剩下的部分只适用于Java 7u6+，Java 8中该部分代码已移除**

在同一更新中，String类还有一个变化：一个新的hash算法。Oracle建议使用一个能使hash码分布更好的新算法，这样能提升一些基于hash的集合类的性能：HashMap，Hashtable，HashSet，LinkedHashMap，LinkedHashSet，WeakHashMap和ConcurrentHashMap。和文章第一部分改变不同的是，这些改变是试验性的，默认是关闭的。

如你所料，这些改变只针对String键。如果想要打开它，你需要设置jdk.map.althashing.threshold系统参数为非负值(默认为-1)。该值将成为一个集合size的阀值，用于新的hashing方法中。注意：只有rehashing（没有足够的可用空间）的过程hashing方法才有变化！所以，如果一个集合上一次rehash时size=160，且jdk.map.althashing.threshold=200，那当集合增长到大约320时才会再次rehash。

现在String有个hash32()方法，它执行的结果缓存到int hash32字段中。hash32()方法最大的特点是，它在同一个String上执行的结果在不同的JVM运行过程中产生的值可能不一样（实际上，多数情况下都会不同，因为该方法使用一个System.currentTimeMills()和两个System.nanoTime()调用作为初始化的种子seed）。因此，每次运行程序时，同样的集合遍历顺序将会不同。

实际上，我对hash32()有点惊讶。既然原有的hashCode方法能够工作，为什么还需要这个方法呢？我决定运行[hashCode方法性能优化指南](http://java-performance.info/hashcode-method-performance-tuning/)文章中的测试程序来检测下hash32方法会产生多少重复的hash码。

String.hash32()方法不是public的，所以我需要查看HashMap的实现，找出如何调用该方法。答案是sun.misc.Hashing.stringHash32(String)。

使用相同的测试方法测试１million不同的键，String.hash32产生304个重复的hash值，对应地，使用String.hashCode却没有重复值。我认为，我们需要wait for further improvements or use case descriptions from Oracle。
