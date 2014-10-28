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

####第一阵营
在第一阵营中，我们将比较各种通过sun.misc.Unsafe进行内存访问的ByteBuffers。不幸的是，Unsafe类只工作在本地字节顺序下(大端或小端)。通过使用Short/Integer/Long.reverseBytes方法(覆盖2/4/8字节数据类型)可以解决该问题，但没有临时缓冲区的话，大量的非本地字节顺序的操作是不可能的，即使有临时缓冲区也没多大意义－这意味着２数据拷贝而不是一个。

Structure:|ByteBuffer|ByteBuffer|ByteBuffer|ByteBuffer| Unsafe
-------------|------------|---------|-------|---------|---------
Type | 	Heap	|Heap	|Direct	|Direct|	| 	 
Endian|	Little|	Big|	Little|	Big	|Little (native)
Write 80M byte buffers (as bytes)|26.21/26.26|27.457/27.365|	101.967/43.338	|102.912/43.396	|31.578/31.846
Write 80M byte buffers (as 1 byte[])|2.966/2.888|3.489/3.321|4.036/4.016|	4.27/4.038	|2.456/2.035
Read 80M byte buffers (as bytes)|	56.163/36.089	|56.322/36.084	|40.1/41.411	|40.519/41.195|	36.68/41.711
Read 80M byte buffers (as 1 byte[])	|6.909/6.842|	6.926/7.036|	6.795/6.808	|7.377/7.176	|6.307/6.317
Write 10M long buffers (as longs)|	36.179/37.316|	51.063/51.221|	14.702/7.366|	64.305/10.127|	7.301/6.699
Write 10M long buffers (as 1 long[])|	32.301/29.651|	53.477/54.115|	2.014/1.912	|27.265/28.319|	1.703/1.701
Read 10M long buffers (as longs)	|59.625/39.923|	53.978/52.097	|26.715/10.941	|67.355/15.932	|8.492/10.026
Read 10M long buffers (as 1 long[])|	47.19/36.373|	60.107/35.186	|6.668/6.754	|32.925/35.071|	6.075/6.143


我们发现了什么?
1. 将字节一个一个地写入直接字节缓冲区是极慢地。你应该避免使用直接字节缓冲区写几乎都是单一字节字段的记录。
2. 如果你有原始数组字段－总是使用批操作方法处理它们。ByteBuffer大量操作方法的性能几乎接近Unsafe方法(不过，ByteBuffer总是慢点)。如果你需要存储或加载除了byte的任何原始数组－在更新完字节缓冲区位置后，调用ByteBuffer.as[yourType]Buffer.put(array)方法。
3. 平均字段长度越长－堆缓存区越慢，直接字节缓冲区越快。Unsafe即使访问分开的字段也是挺快的。
4. 和Java6相比，Java7中许多类型的ByteBuffer访问都经过明显的优化。
5. 总是尝试使用直接字节缓冲区以平台自然的字节顺序序列化原生数组－它的性能和Unsafe相当，不过更灵活。

#####代码
相关代码
