Java集合概述
==================
本文将为你简要介绍Java所有的标准集合。我们会对集合的不同属性和主要使用场景进行分类。除此之外，我们会列出所有在不同的集合类型间转换数据的正确方式。

###数组(Arrays)
数组是Java唯一的内嵌集合类型。当处理的元素数量上限预先知道时，数组是非常有用的。java.util.Arrays包含很多有用的数组处理方法：

* Arrays.asList－可以方便的将数组转换为List，list可以传递给其他标准集合的构造器。
* Arrays.binarySearch－在一个排序的数组或其子集中进行快速查找。
* Arrays.copyOf－需要扩展数组并保留原始内容时使用。
* Arrays.copyOfRange－需要拷贝整个数组或其子集时使用。
* Arrays.deepEquals，Arrays.deepHashCode－Arrays.equals/hashCode支持内嵌子数组的版本。
* Arrays.equals－如果需要比较两个数组内容是否相同，使用该方法而不是array的equals方法（array.equals在任何数组中都没有覆写，因此它只比较数组的引用而不是内容）。为了简化类equals方法的实现，你可以把这个方法和Java５的装箱和参数列表结合起来使用－在比较对象类型后，把类所有字段传给Arrays.equals 。
* Arrays.fill－使用给定的值填充整个或部分数组。
* Arrays.hashCode－计算数组内容hash值的非常有用的方法（数组的hashcode方法就不行）。为了简化类hashcode方法的实现，你可以把这个方法和Java５的装箱和参数列表结合起来使用－把类所有字段都传给Arrays.hashcode。
* Arrays.sort－对整个数组或其部分进行自然排序。Arrays.sort还有一个使用提供的比较器对Object[]数组进行排序的方法。
* Arrays.toString－对数组内容提供良好的打印输出。

如果需要将整个数组或其部分拷贝到另一个已存在的数组中，你需要使用System.arraycopy，它可以从原数组的指定位置开始，复制指定数量的元素到目标数组中。通常，这是Java复制数组内容最快的方式（不过，在某些情况下，你需要检测下ByteBuffer bulk copy是否更快）。

最后，需要提醒的是，任何集合都可以使用T[] Collection.toArray(T[] a)方法复制到一个数组里。这个方法通常的调用模式如下：
`return coll.toArray( new T[ coll.size() ] );`

这个方法分配的数组足以存储整个集合，因此toArray没有必要分配足够大的返回数组。

###单线程安全的集合
这篇文章的以下部分描述了非线程安全的集合类。所有这些集合类存放在java.util包，它们中的一些是Java1.0添加的（现在已过期），大部分是在Java1.4出现的。
