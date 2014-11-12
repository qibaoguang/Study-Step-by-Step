[java.util.LinkedList性能](http://java-performance.info/linkedlist-performance/)
============
这篇文章中我们将讨论LinkedList的性能。通常，在对性能要求严格的代码中使用LinkedList不是个好主意，但有时候你会需要它....

LinkedList是一种每个节点都有指向前和后节点的指针的集合实现，这样的实现利于快速添加/删除/更新元素，但只有在特定的情况下：
* 要么它们是第一个或最后一个元素
* 要么使用ListIterator滚动到需要的元素

在其他情况下，修改集合时间复杂度为O(n)。访问集合元素（get）也是Ｏ(ｎ)时间复杂度（实际上，集合要么从头部，要么从尾部移动，因此访问LinkedList的任何节点最多需要不会超过size()/2的操作步骤）。

我知道的可以使用LinkedList的场景也就只有两个：

1. 你正在实现一个FIFO的缓冲区，并且不需要往缓冲区中部添加或删除元素（或很少需要这样做）。从LinkedList头部或尾部添加/删除元素是非常快的。尽管如此，这种场景下可以考虑使用java.util.ArrayDeque，它也对头部或尾部的快速操作进行了优化。

2. 你需要频繁的添加或删除集合中间的元素。

**使用LinkedList和ArrayDeque实现FIFO缓冲区**

下面让我们看看使用LinkedList或ArrayDeque实现的FIFO队列会有多快。我们将预先填充两个类实例一定数量的values，然后从列表的头部添加5个元素，从列表的尾部移除5个元素。添加/删除操作将会在一个循环中执行100M次。

    final int PREFILL_COUNT = 100_000;
    final int LOOP_COUNT = 100_000_000;
    final LinkedList<Integer> lst = new LinkedList<Integer>();
    final Integer val = 1;
    for ( int i = 0; i < PREFILL_COUNT; ++i )
        lst.add( 35 );
    //start measuring time here<br/>
    for ( int i = 0; i < LOOP_COUNT; ++i )
    {
        for ( int j = 0; j < 5; ++j )
            lst.addFirst( val );
        for ( int j = 0; j < 5; ++j )
            lst.removeLast();
    }
    
结果很有意思。LinkedList的性能在理论上是不受预填充元素数量影响的。实际上，每个添加操作（add）创建一个节点（4个对象Objects-节点本身，previous指针，next指针和value），每个删除操作（remove）清理这些对象，因此会产生相当可观的垃圾需要回收。你的应用内存占用越大，垃圾回收越慢。ArrayDeque对象自身不会产生垃圾只要集合大小稳定，所以它的性能才是真正的不依赖当前集合大小。
<table>
<thead>
<tr>
 <th></th>
 <th>LinkedList,10 elems prefilled</th>
 <th>LinkedList, 100K elems prefilled</th>
 <th>LinkedList, 1M elems prefilled</th>
 <th>ArrayDeque, 10 elems prefilled</th>
 <th>ArrayDeque, 100K elems prefilled</th>
 <th>ArrayDeque, 1M elems prefilled</th>
</tr>
</thead>
<tbody><tr>
 <td>Java 6</td>
 <td>7.533 sec</td>
 <td>7.879 sec</td>
 <td>9.461 sec</td>
 <td>2.323 sec</td>
 <td>2.422 sec</td>
 <td>2.446 sec</td>
</tr>
<tr>
 <td>Java 7</td>
 <td>6.004 sec</td>
 <td>6.493 sec</td>
 <td>7.945 sec</td>
 <td>2.035 sec</td>
 <td>2.160 sec</td>
 <td>2.343 sec</td>
</tr>
</tbody>
</table>

**如何正确使用LinkedList**

LinkedList需要记住的主要特性－它只提供对元素的顺序访问而不是ArrayList的随机访问。所以，不要尝试将以前采用ArrayList写的逻辑适配到LinkedList上－他们都是集合，不错，但它们的实现差别很大。

LinkedList是一个顺序数据结构。这就是为什么所有的基于链表的集合算法都依赖于迭代器的原因(iterators)。在一些情况下，比如remove(Object)隐式的使用迭代器，其他情况下都是显式的调用。例如，你有一个`LinkedList<String>`需要删除某个有５字符的字符串后的所有字符串。如果指定的字符串没有出现，你需要处理整个缓冲区。如果这个例子看起来比较假，可以使用有时间戳和其他属性的消息代替字符串，这样就比较真实了。

