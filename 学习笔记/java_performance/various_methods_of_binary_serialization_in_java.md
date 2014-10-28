[Java中不同二进制序列化方法的性能](http://java-performance.info/various-methods-of-binary-serialization-in-java/)
=============================
我们将去了解java中二进制序列化的性能，下面的类将参与比较：
* DataInputStream(ByteArrayInputStream)和对应的DataOutputStream(ByteArrayOutputStream)。

* 查看同步是如何影响ByteArrayInput/OutputStream的，检查BAInputStream－复制ByteArrayInputStream w/o同步的.

* ByteBuffer的4种风味－基于堆/内存(heap/direct)，大/小端(big/little endian)。

* sun.misc.Unsafe-基于堆的字节数组的内存操作。

以我的经验，所有这些序列化方法的性能既依赖于数据节点的数量，又依赖于缓冲区或流的类型。因此，我写了两组测试。第一个测试工作在只有单一字段-byte[500]的对象，然而第二个测试使用另一个具其他单一字段－long[500]的对象。在ByteBuffer和Unsafe测试中，我们会测试大量的操作，在每次单独的调用中都会序化每个数组元素。

首先，下面是第二联盟(sencond league)的测试结果：流。出于比较目的，我们将记录下使用小端堆(heap little-endian)的ByteBuffer迭代次数相同的结果。请注意，我们在第二测试联盟中使用的迭代次数比较小。

两次测试中，每个单元格显示的是Java6/Java7运行一次需要的秒数。

####第二联盟
|Structure:|DataOutputStream (ByteArrayOutputStream)|DataInputStream (ByteArrayInputStream)|DataInputStream (BAInputStream – not synchronized) |	ByteBuffer |
|----------|---------------|--------------------|------------------|-------------|
|Type | | | |	 	Heap|
|Endian||||	 	 	 	Little|
|Write 1M byte buffers (500 bytes)|	16.178/15.74| - | - |	0.362/0.379|
|Read 1M byte buffers (500 bytes)|	-	|16.106/16.424	|2.432/2.361|	0.471/0.508
|Write 1M long buffers (500 longs)|	11.735/11.632|	-	|-	|3.771/3.691|
|Read 1M long buffers (500 longs)|	-|	11.747/11.622|	9.966/9.532	|6.062/4.069|

需要注意的是：在ByteArray流上同步对小数据类型的影响要大于大数据类型。字符操作中，同步和不同步的版本性能相差8倍，这确实是个问题！
