G1垃圾回收器中的字符串去重(Java 8 Update 20)
================================
从平均情况来看，应用程序中String对象会消耗大量的内存。这里面有一部分可能是重复(冗余)的－同样的字符串存在多个不同的实例(a!=b，但a.equals(b))。在实践中，许多字符串由于各种原因造成重复。

起初，JDK提供String.intern()方法处理字符串重复的问题。该方法的缺点是你需要找出哪些字符串需要驻留(interned)。这通常需要一个具备重复字符串查找功能的堆分析工具，比如[YourKit profiler](http://www.yourkit.com/features/)。尽管如此，如果使用恰当，字符串驻留会是一个强大的节省内存的工具－它允许你重用整个String对象（每个对象会在底层char[]的基础上增加24字节的额外开销）。

从Java 7 update 6开始，每个String对象都有自己私有的底层char[]。这样允许JVM做自动优化－如果底层的char[]数组从没有暴露给客户端，那么JVM就能去判断两个字符串的内容是否一致，进而将一个字符串底层的char[]替换成另一个字符串的底层char[]数组。

Java 8 update 20中引入的字符串去重特性就是用来做这个的，下面是它的工作原理：

1. 你需要使用G1垃圾收集器并启用该特性：-XX:+UseG1GC -XX:+UseStringDeduplication。这个特性是作为G1垃圾收集器的一个可选的步骤来实现的，如果使用其他垃圾收集器则不能使用该特性。

2. 这个特性可能会在G1收集器的minor GC阶段执行。根据我的观察看，它是否执行取决于空闲CPU周期的利用率。所以，不要指望它在一个处理本地数据的数据分析器中会被执行。另一方面，WEB服务器中倒是很可能会执行这个优化。

3. 字符串去重会查找那些未被处理的字符串，计算它们的hash值(如果先前没有被应用代码计算过的话)，然后查找是否有其他具有相同hash值且相等的底层char[]的字符串。如果找到－它会用新字符串的char[]替换掉现有的这个字符串的char[]。

4. 字符串去重只会处理那些经历过几次垃圾收集仍然存活的字符串，这确保多数生命周期很短的字符串不会被处理。字符串的这个最小存活年龄是通过JVM参数-XX:StringDeduplicationAgeThreshole=3管理的(３是该参数的默认值)。

下面是关于这个实现的一些重要结论：

* 没错，如果你想享受字符串去重特性这份免费午餐的话，你需要使用G1收集器。你不能使用并行GC，通常对于追求高吞吐量胜于低延迟的应用这可能是更好的选择。

* 字符串去重无法在一个已加载完的系统中运行。为了检验它是否执行过，可以使用-XX:+PrintStringDeduplicationStatistics参数运行JVM，并观察控制台输出。

* 如果需要节省内存，并且你可以在应用中驻留字符串－就这样做，不要依赖字符串去重的功能。你需要时刻注意的是字符串去重会处理所有或至少大部分字符串－也就是说尽管你知道某个给定的字符串内容是唯一的，比如GUID，但JVM不知道这些，它仍会尝试将这个字符串和其它字符串进行匹配。结果，字符串去重产生的CPU开销既取决于堆中字符串的数量(新的字符串会与它们中的一些进行比较)，也取决于你在字符串去重期间创建的字符串数量(这些字符串需要和堆中的字符串比较)。在拥有好几个G的堆上，可以通过-XX:+PrintStringDeduplicationStatistics JVM选项检查这个特性的影响。

* 另一方面， 字符串去重基本是以非阻塞的方式完成的，如果你的服务器有足够多的空闲CPU，那为什么不用呢？

*  最后，请记住String.intern允许你只针对应用程序中那些已知的会产冗余的字符串进行驻留，通常它只需要跟一个很小的字符串驻留池比较即可，这样能更有效的利用CPU。此外，你可以驻留整个String对象，这样每个字符串可以额外节省24字节。

###特性测试
这是我用来试验这一特性的一个测试类，这３个测试都需要运行到JVM抛出OOM，所以需要单独运行。

第一个测试会创建内容不同的字符串，如果你想模拟当堆中有大量字符串时，字符串去重花费的时间，那这个测试是非常有用的。尽量为第一个测试分配尽可能多的内存－创建的字符串越多，去重效果越好。

第二个和第三个测试用于比较字符串去重(第二个测试)和驻留(第三个测试)间的差别。你需要使用相同的(identical)Xmx设置来运行它们。在程序中，我把这个常量设为Xmx256M，你可以多分配些。然而，你会发现在迭代几次后去重测试将先失败，然后是驻留测试。这是为什么？因为，在这些测试中我们只有100个不同的字符串，因此对它们进行驻留意味着你用到的内存就只是存储这些字符串所需要的空间。而字符串去重的话，会产生不同的字符串对象，它仅会共享底层的char[]数组。

测试用例：

<pre class="no-parse">
/**
 * String deduplication vs interning test
 */
public class StringDedupTest {
    private static final int MAX_EXPECTED_ITERS = 300;
    private static final int FULL_ITER_SIZE = 100 * 1000;

    //30M entries = 120M RAM (for 300 iters)
    private static List&lt;String&gt; LIST = new ArrayList&lt;&gt;( MAX_EXPECTED_ITERS * FULL_ITER_SIZE );

    public static void main(String[] args) throws InterruptedException {
        //24+24 bytes per String (24 String shallow, 24 char[])
        //136M left for Strings

        //Unique, dedup
        //136M / 2.9M strings = 48 bytes (exactly String size)

        //Non unique, dedup
        //4.9M Strings, 100 char[]
        //136M / 4.9M strings = 27.75 bytes (close to 24 bytes per String + small overhead

        //Non unique, intern
        //We use 120M (+small overhead for 100 strings) until very late, but can't extend ArrayList 3 times - we don't have 360M

        /*
          Run it with: -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+PrintStringDeduplicationStatistics
          Give as much Xmx as you can on your box. This test will show you how long does it take to
          run a single deduplication and if it is run at all.
          To test when deduplication is run, try changing a parameter of Thread.sleep or comment it out.
          You may want to print garbage collection information using -XX:+PrintGCDetails -XX:+PrintGCTimestamps
        */

        //Xmx256M - 29 iterations
        fillUnique();

        /*
         This couple of tests compare string deduplication (first test) with string interning.
         Both tests should be run with the identical Xmx setting. I have tuned the constants in the program
         for Xmx256M, but any higher value is also good enough.
         The point of this tests is to show that string deduplication still leaves you with distinct String
         objects, each of those requiring 24 bytes. Interning, on the other hand, return you existing String
         objects, so the only memory you spend is for the LIST object.
         */

        //Xmx256M - 49 iterations (100 unique strings)
        //fillNonUnique( false );

        //Xmx256M - 299 iterations (100 unique strings)
        //fillNonUnique( true );
    }

    private static void fillUnique() throws InterruptedException {
        int iters = 0;
        final UniqueStringGenerator gen = new UniqueStringGenerator();
        while ( true )
        {
            for ( int i = 0; i &lt; FULL_ITER_SIZE; ++i )
                LIST.add( gen.nextUnique() );
            Thread.sleep( 300 );
            System.out.println( "Iteration " + (iters++) + " finished" );
        }
    }

    private static void fillNonUnique( final boolean intern ) throws InterruptedException {
        int iters = 0;
        final UniqueStringGenerator gen = new UniqueStringGenerator();
        while ( true )
        {
            for ( int i = 0; i &lt; FULL_ITER_SIZE; ++i )
                LIST.add( intern ? gen.nextNonUnique().intern() : gen.nextNonUnique() );
            Thread.sleep( 300 );
            System.out.println( "Iteration " + (iters++) + " finished" );
        }
    }

    private static class UniqueStringGenerator
    {
        private char upper = 0;
        private char lower = 0;

        public String nextUnique()
        {
            final String res = String.valueOf( upper ) + lower;
            if ( lower &lt; Character.MAX_VALUE )
                lower++;
            else
            {
                upper++;
                lower = 0;
            }
            return res;
        }

        public String nextNonUnique()
        {
            final String res = "a" + lower;
            if ( lower &lt; 100 )
                lower++;
            else
                lower = 0;
            return res;
        }
    }
}
</pre>

###总结
* 字符串去重是Java 8 update 20添加的新特性。它是G1垃圾回收器的一部分，因此你必须使用G1回收器才能启用它：-XX:+UseG1GC -XX:+UseStringDeduplication。

* 字符串去重是G1的一个可选阶段，它取决于当前系统的负载。

* 字符串去重会查询内容相同的字符串，并统一底层存储字符的char[]数组。使用此特性你不需要编写任何代码，不过这意味着你最后得到的是不同的String对象，每个对象占用24字节。有时候，显式的调用String.intern方法进行字符串驻留还是有必要的。

* 字符串去重不会处理太年轻的字符串。处理字符串的最小年龄是通过JVM参数：-XX:StringDeduplicationAgeThreshold=3来管理的(3是这个参数的默认值)。

###See also
[JEP 192 - a formal description of String deduplication](http://openjdk.java.net/jeps/192)
