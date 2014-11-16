[Core Java 7更新日志](http://java-performance.info/core-java-7-change-log/)
================

**2014年1月1日更新－覆盖所有的Java7版本直到Java 7u45**

本文将列出Java７各版本核心部分更新中与性能相关的变化。我将追踪以下各包的更新：
* java.lang.*
* java.io.*
* java.math
* java.net
* java.nio.*
* java.text.*
* java.util.*

**Java 7u45(跟Java 7u25相比)**

* ArrayList和HashMap的更新是共享空的table

	File changed: \util\ArrayList.java
	
	File changed: \util\HashMap.java

两个非常流行的Java集合－ArrayList和HashMap在实例为空（size=0）的情况下将消耗更少的内存。依我看，这次更新是因为在Oracle的性能基准测试中有大量需要创建map或list的场景，不过由于条件逻辑的存在而没有填充它们（例如，HTTP请求中的空参数列表）。

这个更新通过创建更少的无用垃圾来降低垃圾收集器的负载。

如果是ArrayList，只有通过空构造器创建的对象才受影响。如果你使用指定初始化大小的构造器，那就没有任何变化。

	/**
	 * Shared empty array instance used for empty instances.
	 */
	private static final Object[] EMPTY_ELEMENTDATA = {};
	 
	public ArrayList() {
	    super();
	    this.elementData = EMPTY_ELEMENTDATA;
	}
		
HashMap更新的更彻底－内部table的初始化现已移出构造器。现在它总是使用一个空的table进行初始化。结果，HashMap中所有getters和setters都需要检查map是否为空。这样在map为空的情况下getters会略微快点－你不需要查找实际的table，同时在其他情况下所有的setters和getters就变得稍慢了（这是一个不足为虑的下滑－对单一int字段的０检测，但这仍是一条需要执行的额外指令）。

我希望HashMap的这个更新能够有个坚实的理由－在Oracle的基准测试程序中有足够的总是空的maps。以我个人的观点看，这个更新相当令人怀疑－我不喜欢为个别案例的优化买单(I do not like "taxes" you always have to pay just for a corner case optimization)。

	/**
	 * An empty table instance to share when the table is not inflated.
	 */
	static final Entry<?,?>[] EMPTY_TABLE = {};
	 
	/**
	 * The table, resized as necessary. Length MUST Always be a power of two.
	 */
	transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

[这里](http://www.reddit.com/r/java/comments/1s35n5/core_java_7_change_log/cdtm1z7)是那些更新的一个作者给我的回应.

* 跟踪文件I/O回调

	File changed: \io\FileInputStream.java
	File changed: \io\FileOutputStream.java
	File changed: \io\RandomAccessFile.java
	File changed: \net\SocketInputStream.java
	File changed: \net\SocketOutputStream.java

添加对新类sun.misc.IoTrace的回调调用。这些回调被认为是IO分析的廉价替代品，应该用于JMX代理中。你需要小心不要在自己的代理中使用那些类－否则将会出现死循环。

	public int read() throws IOException {
	    Object traceContext = IoTrace.fileReadBegin(path);
	    int b = 0;
	    try {
	        b = read0();
	    } finally {
	        IoTrace.fileReadEnd(traceContext, b == -1 ? 0 : 1);
	    }
	    return b;
	}

[这里](http://axtaxt.wordpress.com/2013/09/19/experimenting-with-sun-misc-iotrace/)有该更新的一些调查。

* 重写java.lang.invoke

这个包中的大多数类都使用JDK8的Lambda格式重写了。不幸的是，Java8的lambdas看起来不可用。这个更新只有在使用Java 7 MethodHandle类或其他JVM动态语言时才会影响你。

在依赖于该更新的Groovy2.0中存在一个可能的性能回退－我的Groovy版动态方法执行测试两次的运行性能，[Java7u45跟Java7u25相比要慢](http://java-performance.info/static-code-compilation-groovy-2-0/)。

* 解决替换hash算法性能衰退问题（出现于Java 7u6）
	
	File changed: \util\HashMap.java
	File changed: \util\Hashtable.java

Java7 update 6版本中包含新的"替代hash算法"方法，用于String和基于散列的非并发JDK的maps和sets中。详情可参考[我的文章](changes_to_string_java7.md)（从最初写完到现在经过几次修改）。

差不多一年以后，由于在JDK hash sets/maps的构造过程中强制调用sun.misc.Hashing.randomHashSeed方法而产生的高并发性能问题被解决了。现在这个方法只有在设置了jdk.map.althashing.threshold系统属性时才会被调用。这个更新解决了所有受影响的类，除了WeakHashMap（但谁会同时创建大量WeakHashMap实例呢？）

**Java 7u25(和Java 7u21相比)**

* Oracle正在终止使用sun.reflect.Reflection.getCallerClass(int)

sun.reflect.Reflection.getCallerClass(int steps)方法允许你获取调用栈中在你上面的第steps栈帧的类。这个方法允许（Java7中仍然允许）你写对调用者很敏感的代码（caller-sensitive）。

这里是该问题的起源：[Oracle bug database post](http://bugs.java.com/view_bug.do?bug_id=8014925)

其实，Oracle打算在Java 7u55(或以后版本)中移除这个方法，并在Java 8中不支持它。如果你想找出谁是你的调用者，那该方法是有用的。这个方法的性能比Thread.currentThread().getStackTrace()方法好很多。下面的测试报告sun.reflect.Reflection.getCallerClass比公共的API快27倍。

	private static void testGetCallerClass(int cnt, int depth)
	{
	    final Class[] classes = new Class[ cnt ];
	    final long start = System.currentTimeMillis();
	    for ( int i = 0; i < cnt; ++i )
	    {
	        classes[i] = sun.reflect.Reflection.getCallerClass(depth);
	    }
	    final long time = System.currentTimeMillis() - start;
	    System.out.println( "Time for " + cnt + " getCallerClass calls = " + time / 1000.0 + " sec" );
	}
	 
	private static void testGetStackTrace(int cnt, int depth)
	{
	    final String[] classes = new String[ cnt ];
	    final long start = System.currentTimeMillis();
	    for ( int i = 0; i < cnt; ++i )
	    {
	        StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
	        classes[i] = stackTrace[ depth ].getClassName();
	    }
	    final long time = System.currentTimeMillis() - start;
	    System.out.println( "Time for " + cnt + " getStackTrace calls = " + time / 1000.0 + " sec" );
	}
	 
	public static void main(String[] args) {
	    testGetStackTrace(1000000, 2);
	    testGetCallerClass(1000000, 2);
	}

尽管承诺（威胁？）在Java 7u40版本中这个方法将不可用（并添加一个可以打开它的开关），但我的Java 7u45版本中该方法依旧可以访问。

See also:
* [JDK-8014925 : Disable sun.reflect.Reflection.getCallerClass(int) with a temporary switch to re-enable it](http://bugs.sun.com/view_bug.do?bug_id=8014925)
* [JEP 176: Mechanical Checking of Caller-Sensitive Methods](http://openjdk.java.net/jeps/176)
* [Oracle Discontinuing sun.reflect.Reflection.getCallerClass](http://www.infoq.com/news/2013/07/Oracle-Removes-getCallerClass)

**Java 7u21(和Java 7u15相比)**

没有性能相关的更新。

**Java 7u15(和Java 7u7相比)**

没有性能相关的更新。

**Java 7u7(和Java 7u2相比)**

* String.subString不再共享内部的char[]数组

从最初的Java版本以来，这是String第一个大的更新：从Java7u6开始，String不再有offset和count字段。这意味着char[] value字段必须包含从第一个字符到最后一个的整个字符串。

这也意味着唯一的对共享内部char[]的公共访问接口－String.substring方法现在必须拷贝原始char[]的一部分，因此它的时间复杂度从O(1)（创建一个单一的对象并设置3个字段）增加到O(n)（拷贝新字符串的所有字符）。

new String(String)构造器现在变得毫无价值。以前需要“拷贝子数组”的行为时可以使用它（为了避免内存泄漏）。

整个来龙去脉可以读我的文章：[Java 1.7.0_06中String内部表示的变化](changes_to_string_java7.md)

* 不同JDK maps/sets的可选散列算法

以下类受到影响：HashMap，HashTable，HashSet，LinkedHashMap，LinkedHashSet，WeakHashMap和ConcurrentHashMap。

如果map/set的大小超过jdk.map.althashing.threshold这个JVM参数的值，这些类就会切换到String类里的一个可选hash32方法。理论上，这个更新应该提高散列码的分布水平，因此可以使基于散列的maps/sets更快。实际上，这个更新只是Java 7的特性，在Java 8中并没有出现。详情参考[Java 1.7.0_06中String内部表示的变化](changes_to_string_java7.md)。

除了CocurrentHashMap以外的其他所有类	在Java 7u6(引入)和Java 7u40(解决)版本中会有并发问题－在构造期间，它们依赖一个java.util.Random的单例（实际处于jdk.map.althashing.threshold参数的处理中）－这限制了你在高竞争环境下可以创建的maps/sets数量。

* java.	io.InputStream.skip更新

java.io.InputStream.skip方法是通过将跳过的数据读到临时缓冲区，并丢弃它来实现的。原先的实现会在每个InputStream实例第一次调用skip时缓存２k的缓冲区。一方面它创建更少的垃圾（理论上）。另一方面，你可能不会再用	skip方法（不像read方法），但你需要保留那份临时缓冲区。

新的实现（至少从Java 7u7版本开始）在每次调用skip方法时都会分配一个临时的缓冲区。缓冲区的大小是2k和跳过的数据量中最小的那个。

* Integer/Long.toString稍微快了一些

更新的版本中用了一个包级私有的构造器new String(char[],boolean)，它不会拷贝提供的char[]。老的代码使用会拷贝提供的char[]的new String(offset,cout,char[])构造器。

Byte/Short.toString方法没有更新，因为它们调用Integer.toString方法进行转换。最后，Float/Double.toString使用sun.misc.FloatingDecimal进行转换，它不能访问String包级私有的构造器。

* java.util.Collections包装类的equals方法更新

java.util.Collections中的一些集合包装类现在有了稍微快点的equals方法：添加了if(this==other) return true分支。
