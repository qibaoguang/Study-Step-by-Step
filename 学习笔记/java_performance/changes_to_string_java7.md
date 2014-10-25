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

**Java 7u6到Java 7u40版(不包括)的JDK中，新的hashing可能严重影响多线程代码**

**本段适用于Java 7 build 6(包括)到build 40(不包括)，这些代码在Java8中已移除。查看下个段落可以了解到关于Java 7u40+的信息**

Oracle在以下类的hashing实现中制造了一个bug：HashMap，Hashtable，HashSet，LinkedHashMap，LinkedHashSet和WeakHashMap。只有ConcurrentHashMap不受影响。问题是现在所有的非并发类都有下面的字段：
>/**
>
 >\* A randomizing value associated with this instance that is applied to
 >
 >\* hash code of keys to make hash collisions harder to find.
 >
 >*/
 >
>transient final int hashSeed = sun.misc.Hashing.randomHashSeed(this);

 这意味着，每次创建map/set实例都调用sun.misc.Hashing.randomHashSeed方法。相应地，randomHashSeed方法调用java.util.Random.nextInt方法。众所周知，Random类对多线程支持并不友好：它包含private final AtomicLong seed字段。在竞争小的情况下Atomics工作的很好，但竞争大的情况下性能极差。

结果，很多处理HTTP/JSON/XML请求的高负载web应用都会受到这个bug的影响，因为所有已知的解析器都使用受影响的集合类中的一个来表示"name-value"。这些格式解析器会创建内嵌的maps，并且数量每秒都在递增。

如何解决该问题？

1. ConcurrentHashMap方式：只有在系统属性jdk.map.althashing.threshold设置的时候才调用randomHashSed方法。不幸的是，只有JDK核心开发人员才能使用。

>/** A randomizing value associated with this instance that is applied to hash code of keys to make hash collisions harder to find.
 >*/
>private transient final int hashSeed = randomHashSeed(this);
>
>private static int randomHashSeed(ConcurrentHashMap instance) {
>
  >  if (sun.misc.VM.isBooted() && Holder.ALTERNATIVE_HASHING) {
  >  
   >     return sun.misc.Hashing.randomHashSeed(instance);
   >     
   >}
>
>   return 0;
>}

2. hacker方式：修改sun.misc.Hashing类，不过不推荐，如果你想这么干，可以这样：java.util.Random类不是final的，你可以继承这个类，覆盖里面的nextInt()方法，返回thread-local的值（甚至可以是常量）。
更好的方式是，你可以使用Java 7的java.util.concurrent.ThreadLocalRandom，它是一个使用ThreadLocal<ThreadLocalRandom>的线程局部的Random子类（感谢[Benjamin Possolo](https://plus.google.com/u/0/109148277999114144772/posts)指出我在原来的文章中没有提到这个类）。除了是线程局部外，ThreadLocalRandom还是CPU 缓存的（cache-aware）：为了消除在一个缓存线结束时产生两个不同种子的机会，它在每个ThreadLocalRandom实例产生种子后添加64个字节填充(高速缓存大小)。

然后修改sun.misc.Hashing.Holder.SEED_MAKER这个字段，把它设置为你继承的Random类的实例。 虽然这个字段是private static final的，别担心，用反射来搞定： 
> public class Hashing { 
>
>  private static class Holder { 
>
>    static final java.util.Random SEED_MAKER;
    
3. Buddhist方式：不要升级到Java 7u6和更高版本，检查Java 7的更新版本看是否修复了该bug。
   
###Java 7u40+中，新的hash算法不再影响多线程代码的性能。

Oracle在Java 7u40修复了上面提到的问题。他们采用上段的方式１解决的－只有在新的hash算法启用的情况下才调用sun.misc.Hashing.randomHashSeed(this)。Oracle只修复２个类：HashMap和Hashtable，但它也间接修复HashSet，LinkedHashMap和LinkedHashSet，因为这３个类是构建在HashMap之上的。唯一没有修复的类是WeakHashMap，但我无法想象积极创建该类实例的使用示例。

###总结
* 从Java 1.7.0_06开始，String.substring总是为每个它产生的String实例创建一个新的底层char[] value。也就是说，和以前的常量复杂度相比，现在的时间复杂为线性。这个改变的好处是每个String占用的内存字节数减少了８字节，而且还避免了因为调用String.substring导致的内存泄漏（可以查看[ String packing part 1: converting characters to bytes](http://java-performance.info/string-packing-converting-characters-to-bytes/)了解更多关于Java 对象内存模型的信息）。

* 从Java 7u6版本开始(Java8已移除)，String类有了第二个hash方法：hash32。这个方法目前不是public的，不使用反射的情况下，只能通过sun.misc.Hashing.stringHash32(String)调用。如果Jdk 7中基于hash的集合的大小超过jdk.map.althashing.threshold系统属性的值，该方法会被使用。这是试验性的功能，目前不建议你在自己的代码中使用。

* Java 7u6(包括)和Java 7u40(不包括)之间的Java版本中，由于受新的hash算法实现的影响，所有标准的JDK非并发maps和sets都会产生性能问题。这个bug只会影响每秒创建大量map的多线程应用。

###See also
[ comment regarding the reasons behind these changes.](http://www.reddit.com/r/programming/comments/1qw73v/til_oracle_changed_the_internal_string/cdhb77f)
[String.intern in Java 6, 7 and 8 – string pooling](http://java-performance.info/string-intern-in-java-6-7-8/)
