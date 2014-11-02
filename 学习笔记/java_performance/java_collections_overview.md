Java集合概述
==================
本文将为你简要介绍Java所有的标准集合。我们会对集合的不同属性和主要使用场景进行分类。除此之外，我们会列出所有在不同的集合类型间转换数据的正确方式。

##数组(Arrays)
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

##单线程集合
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

* WeakHashMap－这个map通常用于数据缓存实现中。它使用WeakReference来保存所有的键，这意味着如果没有指向键对象的强引用，这些键就可能被垃圾回收。另一方面，对应的values采用强引用存储。因此，你要么确保没有从values到keys的引用，要么把values也保存到弱引用中：m.put(key,new WeakReference(value))。

###Sets
* HashSet－一种基于dummy values的HashMap的set实现（some Object is used for every value），和HashMap有相同的特性。因为这样的实现，导致HashSet消耗的内存比这种数据结构实际需要的多。

* EnumSet－一种值为枚举类型的set。Java中每个枚举都被映射为一个整数：每个枚举值都对应一个不同的整数。这允许EnumSet使用跟BitSet类似的结构，这种结构中每个bit映射为一个不同的枚举值。这里实际上有２种实现－底层使用单一的long型存储的RegularEnumSet（能够存储多达64个枚举值，涵盖了99.9%的使用案例）和底层采用long[]的JumboEnumSet。

* BitSet－一种位set。需要注意的是，你可以使用一个BitSet代替一个密集的整数集（比如事先知道起始位置的ids）。这个类采用long[]存储位数据。

* LinkedHashSet－和HashSet相似，这个类上层采用LinkedHashMap实现。这是唯一以插入顺序保存元素的set。

* TreeSet－和HashSet类似，这个类基于一个TreeMap实例。这是标准JDK单线程部分中唯一的有序集合。

##java.util.Collections
就像java.util.Arrays对于数组那样，java.util.Collections有很多用于集合处理的有用方法。

第一组方法可以返回集合的不同视图：

* Collections.checkedCollection/checkedList/checkedMap/checkedSet/checkedSortedMap/CheckedSortedSet－返回一个在运行时校验添加元素类型的集合视图。任何试图添加一个不兼容类型的元素都会抛出ClassCastException。为了防止运行时类型转换错误，这个功能可能是需要的。

* Collections.emptyList/emptyMap/emptySet－当你需要返回一个不可变的空集合且不想分配任何对象时非常有用。

* Collections.singleton/singletonList/singletonMap－返回一个单一实体的不可变set/list/map。

* Collections.synchronizedCollection/synchronizedList/synchronizedMap/synchronizedSet/synchronizedSortedMap/synchronizedSortedSet－返回一个所有方法都同步的集合视图（廉价且低效的多线程实现，不支持复合操作，比如put-or-update）。

* Collections.unmodifiableCollection/unmodifiableList/unmodifiableMap/unmodifiableSet/unmodifiableSortedMap/unmodifiableSortedSet－返回一个不能修改的集合视图。当你需要实现能存放任何集合的不可变对象时很有用。

第二组包含各种由于某些原因没有添加到集合中的方法：

* Collections.addAll－如果你需要往集合中添加一定数量的元素或一个数组内容时调用它。
* Collections.binarySearch－和数组的Arrays.binarySearch一样。
* Colletions.disjoint－检查两个集合是否有共同元素。
* Collections.fill－使用指定的值替换列表中所有的元素。
* Collections.frequency－统计给定集合中有多少元素跟指定对象相同。
* Collections.indexOfSubList/lastIndexOfSubList－这些方法看起来和String.indexOf(String)/lastIndexOf(String)相似－它们寻找给定列表中指定子列表第一次或最后一次出现的位置。
* Collections.max/min－找到集合中基于自然顺序或比较器的最大/最小元素。
* Collections.replaceAll－使用指定元素替换列表中的另一个元素。
* Collections.reverse－翻转给定列表中元素的顺序。如果在对列表排序后立马调用此方法，你最好在排序过程中使用**Collections.reverseOrder比较器**。
* Collections.rotate－将列表中的元素旋转给定的距离。
* Collections.shuffle－打乱（洗牌shuffle）列表。注意你可以为该方法提供自定义的随机数生成器－可以是java.util.Random/java.util.ThreadLocalRandom或java.security.SecureRandom。
* Collections.sort－根据自然顺序或给定的比较器对列表进行排序。
* Collections.swap－交换列表中给定位置的两个元素（需要开发者都是自己写）。

##并发集合
文章的本部分讨论java.util.concurrent包中线程安全的集合。这些集合的主要特征是保证它们的方法能够以原子性执行。你最好不要忘记复合操作，比如"add-or-update"或"check-then-update"，它们涉及到多个方法调用，应该仍就处于同步，因为复合操作第一步需要的集合信息在到达第二步前可能已变得无效了。

大部分的并发集合是在Java1.5引进的。ConcurrentSkipListMap／ConcurrentSkipListSet和LinkedBlockingDeque是在Java1.6中添加的。Java 1.7中最新添加的是ConcurrentLinkedDeque和LinkedTransferQueue。

###Lists
* CopyOnWriteArrayList－list实现，每次更新都会创建一个新的底层数组的拷贝。这是一个昂贵的操作，因此这个类可以用于遍历操作远多于更新的场合。该集合的常见用例是作为监听者/观察者集合。

###Queues/deques
* ArrayBlockingQueue－一个底层采用数组实现的有界阻塞队列。不能改变大小，因此当你尝试往一个满的队列添加元素时，方法调用会阻塞直到另一个线程从队列取出一个元素。

* ConcurrentLinkedDeque/ConcurrentLinkedQueue－一个底层采用链表实现的无界双端队列/队列。往队列中添加元素不会阻塞，但有个不幸的要求就是集合的消费者必须至少跟生产者工作的同样快，否则你会用完内存。严重依赖于CAS操作（compare-and-set）。

* DelayQueue－一个具有延迟元素的无界阻塞队列。元素只有在延迟过期后才能从队列中取出。队列的头是具有最短延迟的元素（包括负的值－延迟已过期）。这个队列可能会有用，例如你想实现一个具有延迟任务的队列（不要手动实现这样的队列－使用ScheduledThreadPoolExecutor代替）。

* LinkedBlockingDeque/LinkedBlockingQueue－基于链表的可选边界（创建时可以指定最大容量）的队列/双端队列。它使用ReentrantLock-s作为empty/full条件。

* LinkedTransferQueue－一个基于链表的无界队列。除了普通的队列操作，它还有一组transfer方法，允许生产者直接将消息发送给等待的消费者，因此也就不需要将元素存储到队列中了。这是一个基于CAS操作的不需要锁的集合。

* PriorityBlockingQueue－一个PriorityQueue的无界阻塞版本。

* SynchronousQueue－一个没有任何内部容量的阻塞队列。这意味着任何插入请求必须等待相应的删除请求，反之亦然（vice versa）。如果你不需要一个Queue接口，那通过Exchanger类也能实现同样的功能。


