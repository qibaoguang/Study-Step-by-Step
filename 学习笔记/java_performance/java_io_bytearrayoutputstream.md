[java.io.ByteArrayOutputStream](http://java-performance.info/java-io-bytearrayoutputstream/)
========================
### 在对性能要求苛刻的代码中不要使用ByteArrayOutputStream

***非常重要***：在对性能要求苛刻的代码中不要使用ByteArrayOutputStream。如果你仍认为需要它-读一下本文的剩余部分。

ByteArrayOutputStream经常用在向一些输出流写入不知道长度的短消息的情况（你的消息中有很多不能计算大小的吗？）。

***重要***：如果你事先知道消息的大小（或至少知道它的上限）-分配一个ByteBuffer来替代ByteArrayOutputStream（或复用先前分配的），并将消息写入ByteBuffer。它比ByteArrayOutputStream快很多。

ByteArrayOutputStream允许你向它内部的一个可扩展字节数组写入任何东西，随后将该数组作为一整块进行输出。默认的缓冲区大小为32字节，所以如果你写入的数据比较长，可以在ByteArrayOutputStream(int)构造器中显式设置缓冲区大小。

当你实现一个回调方法，需要调用者提供一些属性未定义的输出流，或你正在实现一些消息到字节数组转化的序列化方法时，多数情况下需要用到ByteArrayOutputStream。

从文章[Various methods of binary serialization in Java](http://java-performance.info/various-methods-of-binary-serialization-in-java/)中你可以了解到ByteArrayOutputStream的方法是同步的，这会严重影响它的性能。所以，如果你不需要同步，找到JDK的源码，将类的内容拷贝到你的工程中，删除类中所有的同步代码（忘了它吧，这只是我的建议），这将让它更快些...

### 如何使用ByteArrayOutputStream
你是不是仍然想使用未打过补丁的ByteArrayOutputStream？那好吧，本文的剩余部分将告诉你如何使用它。

即便如此，你也应该使用BufferedOutputStream而不是ByteArrayOutputStream。它们唯一的区别是底层流的write方法调用次数-对于ByteArrayOutputStream每次总会调用write，而BufferedOutputStream只有在内部缓冲区满的时候才会调用（将缓冲区的内容写到底层流中）。

所以，当你实现存储若干消息到底层流的方法时，可以创建一个ByteArrayOutputStream，并保持对它的引用。为了方便的向流中写入大部分的数据类型，你可以在ByteArrayOutputStream外包装一层DataOutputStream，然后写入消息的各个字段到DataOutputStream中，关闭它（为了将所有的数据刷新到ByteArrayOutputStream里）。

示例如下：
<pre>
private static final class LogEvent
{
    public final int ipv4;
    public final long time;
    public final String eventDesc;
 
    public void saveTo( final OutputStream os ) throws IOException {
        final ByteArrayOutputStream baos = new ByteArrayOutputStream( 12 + 2 + eventDesc.length() * 2 );
        final DataOutputStream dos = new DataOutputStream( baos );
        try
        {
            dos.writeInt( ipv4 );
            dos.writeLong( time );
            dos.writeUTF( eventDesc );
        }
        finally {
            dos.close();
        }
        baos.writeTo( os );
    }
}
</pre>
如何使用它？这里有第二个技巧。很多开发者知道toByteArray方法，它是一个非常方便的方法，不过，使用它需要一定的代价。这个方法创建了对内部字节数组的拷贝，并返回给调用者。多数情况下，你可能立刻将字节数组写到另一个输出流。所以，使用ByteArrayOutputStream.writeTo(OutputStream)方法替代。这个方法使用的是内部的字节数组而不是拷贝。

我们很少了解ByteArrayOutputStream.toString()和ByteArrayOutputStream.toString(String charsetName)方法。它们允许你把内部的字节数组转为一个字符串。第一个方法使用默认的编码（不是一个好主意），第二个使用提供的编码名。和其他相似方法组不同的是，ByteArrayOutputStream没有提供接受Charset对象的方法。

### 总结
* 对于性能要求严格的代码，使用ByteBuffer代替ByteArrayOutputStream。如果你仍要使用ByteArrayOutputStream-远离它的同步。
* 如果你需要实现一个将一些短消息写入未知的输出流的方法，总是先将你的消息写入到ByteArrayOutputStream，然后使用它的writeTo(OutputStream)方法。如果需要从字节数组的表示形式构造一个字符串，别忘了ByteArrayOutputStream.toString方法。
* 多数情况下要避免使用ByteArrayOutputStream.toByteArray方法-它创建一个内部数组的拷贝。如果你的程序只使用很少G的内存，垃圾收集器处理这些拷贝耗时很明显（参考文章[Inefficient `byte[]` to String constructor](http://java-performance.info/inefficient-byte-to-string-constructor/)中的另一个示例）。
