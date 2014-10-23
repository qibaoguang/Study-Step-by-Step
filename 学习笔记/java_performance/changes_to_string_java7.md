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

