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
这篇文章的以下部分描述了非线程安全的集合类。所有这些集合类存放在java.util包，它们中的一些是Java1.0添加的（现在已过期），大部分是在Java1.4出现的。Java1.5加入支持枚举类型的集合，所有集合类也都支持泛型了。PriorityQueue也是Java1.5加入的。Java1.6添加的ArrayDeque是最新加入的非线程安全框架。

###Lists
* ArrayList-最常用的List实现。底层是通过数组和一个int-数组中第一个未被使用的元素的位置实现的。和其他Lists实现一样，ArrayList也会在必要的时候扩展自己。ArrayList访问元素的时间复杂度为常量级。尾部更新很廉价（常量级时间复杂度），由于ArrayList是不可变的造成其头部更新很昂贵-底层数组从索引为０到需要更新位置的所有元素需要向右移动以完成插入，向左移动以完成删除。CPU缓存友好的集合由于底层采用数组实现导致其缓存效果并不太好，因为它内部包含Objectｓ，它们是指向实际对象的引用。

* LinkedList－双端队列（Deque）的实现，每个Node由一个value，prev和next指针组成。也就是说，访问或更新元素的时间复杂度为线性复杂度（由于优化，这些方法不会遍历超过一半的集合，所以位于集合中间的元素才是最昂贵的）。如果想写出快速的LinkedList代码，你需要使用ListIterators。如果你实现一个Queue/Deque（只能访问第一个和最后一个元素）－可以考虑使用ArrayDeque。

* Vector－ArrayList的一个遗留版本，所有的方法都是synchronized。使用ArrayList替代它。

###Queues/deques
* ArrayDeque－基于数组的双端队列Deque实现，有head/tail指针。跟LinkedList不一样，这个类不实现List接口，也就是说，除了第一个和最后一个元素你访问不到任何数据。由于ArrayDeque产生的垃圾是有限的（扩展时原始数组会被抛弃），通常情况下这个类比LinkedList更适合实现queues/deques。

* Stack－一个LIFO队列。生产环境的代码不要使用它，而是使用任何双端队列代替（ArrayDeque非常合适）。

* PriorityQueue－基于优先级堆的队列。既可以使用自然顺序也可以使用提供的比较器Comparator。它的主要特性－poll/peek/remove/element方法总是返回队列中剩余的最小元素。除此之外，这个队列实现的Iterable接口不是以有序形式进行遍历的（或任何其他的特殊顺序）。如果你需要获取队列中最小元素，通常PriorityQueue比其他可排序集合更合适，比如TreeSet。

###Maps
* HashMap－一个很流行的map实现。它只是将keys映射到values，没有其他功能。如果有一个高质量的hashcode方法，get/put方法的时间复杂度就是常量级的。

* EnumMap－使用枚举类型作为key的map。通常情况下，EnumMap比HashMap要快，因为EnumMap知道keys的最大数量，内嵌的有enum到int的映射（一个存放values的固定大小的数组）。

* Hashtable－早期遗留下的HashMap同步版本。在新的产品代码中使用HashMap。

* IdentityHashMap－Map中一个违背常识的特殊版本：它使用==而不是调用Object.equals比较引用。这个特性让IdentityHashMap非常适合于各种图的遍历算法－你可以很轻松的将处理过的节点和一些节点相关的数据存储到IdentityHashMap中。

* LinkedHashMap－HashMap和LinkedList的结合，所有元素的插入顺序都被存储到一个LinkedList中。这就是LinkedHashMap entries，keys和values总是以插入的顺序进行遍历的原因。就每个元素的内存消耗而言，LinkedHashMap是JDK最昂贵的集合！

* TreeMap－一种基于红黑树的排序导航Map。它采用自然顺序或给定的比较器对所有实体（entries）进行排序。这个map要求equals和Comparable／Comparator.compareTo的实现保持一致。该类实现了NavigableMap接口：它允许获取map中比给定key大或小的所有实体；获取一个pre/next实体（基于键的顺序）；使用一个给定范围内的键获取一个map（大致上相当于SQL的BETWEEN操作）和这些方法的许多变体。

