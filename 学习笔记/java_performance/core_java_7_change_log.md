Core Java 7更新日志
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

**跟踪文件I/O回调**

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

**重写java.lang.invoke**

这个包中的大多数类都使用JDK8的Lambda格式重写了。不幸的是，Java8的lambdas看起来不可用。这个更新只有在使用Java 7 MethodHandle类或其他JVM动态语言时才会影响你。

在依赖于该更新的Groovy2.0中存在一个可能的性能回退－我的Groovy版动态方法执行测试两次的运行性能，[Java7u45跟Java7u25相比要慢](http://java-performance.info/static-code-compilation-groovy-2-0/)。

