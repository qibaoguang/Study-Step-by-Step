Java 8 Update 20 的新特性 —— 字符串去重
================================
一般的应用程序中，String对象消耗大量的内存。这些字符串有一些可能是重复的－相同的String，不同的实例(a!=b，但a.equals(b))。实际上，由于各种原因，许多字符串都是重复的。

起初，JDK提供String.intern()方法处理字符串重复。该方法的缺点是你需要找出哪些字符串需要驻留(interned)。这一般需要一个能够查找字符串重复功能的堆分析工具，比如[YourKit profiler](http://www.yourkit.com/features/)。尽管如此，如果使用恰当，字符串驻留就是一个强大的内存节省工具－它允许你重用整个String对象（每个对象都会在底层char[]添加24字节的开销）。

从Java 7 update 6开始，每个String对象都有自己私有的底层char[]。这允许JVM做自动优化－如果一个底层char[]从没有暴露给客户端，并且JVM能够找到２个有相同内容的字符串，那么它就会使用一个字符串的底层char[]替换另一个字符串的底层char[]。

字符串去重特性是在Java 8 update 20完成的。它如何工作：

1. 你需要使用G1垃圾收集器并启用该特性：-XX:+UseG1GC -XX:+UseStringDeduplication。这个特性是G1垃圾收集器实现的可选步骤，其他任何垃圾收集器都不可用。

2. 这个特性可能会在G1收集器的minor GC阶段执行。在我的观察，它取决于空闲CPU周期的利用率。所以，不要期望它在一个所有数据都本地处理的计数器下工作。另一方面，一个web服务器可能会经常执行它。

3. 字符串去重会查找没有处理的字符串，计算它们的hash码(如果先前没有被应用代码计算过)并查找是否有其他具有相同hash码且相等的底层char[]的字符串。如果找到－使用已存在的字符串的char[]替换新字符串的char[]。

4. 字符串去重只会处理那些经历过几次垃圾收集的字符串，这确保多数生命周期很短的字符串不会被处理。最小的字符串年龄通过JVM参数-XX:StringDeduplicationAgeThreshole=3管理(３是该参数默认值)。

这里有一些该实现的重要结论：

* 是的，如果想使用字符串去重的特性，你需要使用G1收集器。你不能使用并行GC，通常对于低延迟高吞吐量的应用这是更好的选择。

* 字符串去重不太可能在加载后的系统运行。为了检验它是否调用，可以使用-XX:+PrintStringDeduplicationStatistics选项运行JVM并观察控制台输出。

* 如果需要节省内存，并且你可以在应用中驻留字符串－就这样做，不要依赖字符串去重。记住：字符串去重会处理所有或至少大部分字符串－这意味着即使你知道某个给定的变量内容是独一无二的，比如GUID，但JVM不知道这些，它仍会尝试将该字符串和其他字符串进行匹配。结果，字符串去重的CPU消耗跟堆中字符串的数量(一个新的字符串会与它们中的一些比较)及字符串去重期间你创建的字符串数量(这些字符串需要和堆中的字符串比较)相关。在几GＢ的堆上，使用-XX:+PrintStringDeduplicationStatistics JVM选项检查这个特性的影响。

* 另一方面， 字符串去重是以非阻塞的方式运行的，如果你的服务器有空闲的CPU容量，为什么不使用它呢？

*  最后，记住String.intern允许你作用在那些已知的包含重复的字符串子集，通常只需要跟很小的字符串驻留池比较即可，这样能更有效的利用CPU。此外，它允许你驻留整个String对象，因此每个字符串可以节省额外的24字节。

	这里有我用来检验该特性的测试类，这３个测试每个都需要运行到JVM抛出OOM，所以需要单独运行。

	第一个测试创建内容不同的字符串，如果你想模拟当堆中有大量字符串时，字符串去重花费的时间，那这个是非常有用的。试着为第一个测试分配最多的堆－创建的字符串越多，效果越好。

	第二个和第三个测试用于比较字符串去重(第二个测试)和驻留(第三个测试)间的差别。你需要使用相同的(identical)Xmx设置来运行它们。在程序中，我把这个常量设为Xmx256M，你可以多分配些。然而，你会发现在迭代几次后去重测试将失败，然后是驻留测试失败。为什么？因为，在这些测试中我们只有100个不同的字符串，因此对它们进行驻留意味着你用到的内存就只是存储这些字符串所需要的空间。而字符串去重的话，会产生不同的字符串对象，它仅会共享底层的char[]数组。
	
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





