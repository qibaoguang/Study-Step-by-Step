[Java中不同二进制序列化方法的性能](http://java-performance.info/various-methods-of-binary-serialization-in-java/)
=============================
我们将去了解java中二进制序列化的性能，下面的类将参与比较：
* DataInputStream(ByteArrayInputStream)和对应的DataOutputStream(ByteArrayOutputStream)。

* 查看同步是如何影响ByteArrayInput/OutputStream的，检查BAInputStream－复制ByteArrayInputStream w/o同步的.

* ByteBuffer的４种风味－基于堆/内存(heap/direct)，大/小端(big/little endian)。

以我的经验，所有这些序列化方法的性能既依赖于数据节点的数量，又依赖于缓冲区或流的类型。因此，我写了两类测试。第一个测试工作在只有单一字段-byte[500]的对象，然而第二个测试使用另一个具其他单一字段－long[500]的对象。
在ByteBuffer和Unsafe测试中，我们会测试大量的操作，在每次单独的调用中都会序化每个数组元素。
