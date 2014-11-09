[java.util.ArrayList性能指南](http://java-performance.info/arraylist-performance/)
==========================
本文中，我们将讨论ArrayList大部分的性能问题。我们将ArrayList的方法划分为若干组，并讨论每组的性能。

ArrayList是一个普通的列表实现，适用于大多数场景。底层采用Object数组实现，它的大小随着用户添加或删除列表中的元素而动态调整。如果你需要一个可以改变大小且存放原生类型值的列表，你应该读[a Trove library article](http://java-performance.info/primitive-types-collections-trove-library/)。

###向列表中添加元素
这里有两对方法用于添加元素：
* `add(E)`
* `add(int，E)`
* `addAll(List<E>)`
* `addAll(int，List<E>)`

单参数的方法用于将新元素添加到列表的尾部，它们是最快的方法。指定插入位置的方法需要调用System.arraycopy拷贝从插入位置开始右边的所有数组元素，这就是这些方法时间复杂度为O(n)的原因（如果新元素添加到尾部附近，性能损耗很小。但插入的位置越靠近头部，性能损耗越大）。也就是说，你不应该向一个大的ArrayList的头部添加很多元素。在我的电脑上，将1M的元素添加到`List<String>`的头部花费了125秒。添加0.5M花费30秒，这证明了调用时间复杂度为O(n)的方法n次的复杂度为O(n*n)（多添加２倍元素，耗时增加４倍）。

如果数组中没用足够的空间存放新元素，它应该被扩展。Java7中扩展逻辑改变了：先前新数组的大小为oldSize*3/2 + 1，Java7中它的大小为oldSize*3。

###移除列表中的元素
每个移除方法都有自己的问题，所以它们会被分开讨论。

**remove(int)**

这个方法移除指定位置的单个元素。之后，通过调用System.arraycopy，右边的所有元素都被往左转移，这就是该方法时间复杂度为O(n)的原因。这里有两个潜在的问题：

如果我们有`ArrayList<Integer>`，你就需要注意所有对remove(int)的调用：
你将调用的是remove(int)还是remove(Object)？你可能需要手动转换remove(int)参数为int数据类型，remove(Object)参数为Integer数据类型。例如，如果你想移除列表的第一个元素，你应该调用remove(0)，但如果你想移除列表第一个值为０的元素，你最好调用remove((Integer)0)。

如上所述，remove(int)时间复杂度为Ｏ(n)。这个方法带来的最大问题（author has ever seen）是在缓冲区的实现中。首先，调用者代码向缓冲区添加很多元素，然后使用下面的代码片段处理缓冲区的内容：
<pre>
while ( !buffer.isEmpty() )
{
    Elem el = buffer.remove( 0 );
    process( el );
}
</pre>
初始数据集不会超过100K元素，所以该方法处理速度的下降也不会被注意。之后其他人决定使用同样的程序去处理更大的数据集－几百万消息。结果，该代码差不多停止执行了。在我的机器上，使用上述代码清理1M字符串列表耗时126秒（清理0.5M耗时30秒－为了证明这种方式的时间复杂度为O(n*n)）。差不多和add测试用例花费相同的时间。这证明两个用例中多数时间都花费在Sytem.arraycopy调用上。唯一不同的是，将新元素添加到头部会将其他元素向右移，而移除头部的元素会将它们向左移。

那么，如何解决这个问题？如果所有的元素都是先添加到缓冲区然后处理（删除掉），我们可以通过调用Collection.reverse翻转列表，然后从列表尾部删除消息。
<pre>
Collections.reverse( buffer );
while ( !buffer.isEmpty() )
{
  Elem el = buffer.remove( buffer.size() - 1 );
  process( el );
}
</pre>
删除最后元素的操作不会调用System.arraycopy，所以调用这个方法的时间复杂度为O(1)。使用该代码删除即使是百万级消息也是咋眼之间就完成。

如果是不同的使用模式：添加一些元素，处理一些元素，添加更多的元素等等，我们可能需要一个LinkedList或使用下面讨论的ArrayList.subList	。

**remove(Object)**

这个方法删除列表中第一次出现的指定元素。它支持null参数，并且有一个处理null的分支。它遍历列表所有的元素，所以时间复杂度为O(n)。该方法在任何情况下都会访问所有的元素-不管是查找指定元素时对它们进行读取，还是在指定的元素找到后，调用System.arraycopy将它们移动到左边。移动操作快些，但仍需要访问所有元素。

这个方法和上面讨论的remove(int)具有相同的问题。使用该方法我们甚至可以设计出(devise)更糟糕的缓冲区处理代码：
<pre>
while ( !buffer.isEmpty() )
{
    Elem el = buffer.get( 0 );
    process( el );
    buffer.remove( el );
}
</pre>
当你知道元素的位置时绝不要调用remove(Object)! Besides a lookup,你可能会删错数据。

目前为止，**remove方法最重要的性质-它们不会缩小内部数组的大小，clear方法也不会，只有trimToSize会**。如果你的数据有个峰值(peak)，那在峰值之间的时间段内都在浪费内存，因为内部数组将保持足够大以适应峰值数据。这就是为什么在缓冲区大小降低到预定义级别（例如，从100K以上的元素到100K以下的元素）以下时，考虑调用trimToSize是值得的。

**removeAll(Collection)，retainAll(Collection)**

第一个方法删除所有出现在参数中的元素，第二个方法保留所有出现在参数中的元素。两个方法的时间复杂度都为Ｏ(n*n)，因为它们遍历ArrayList中的所有元素，并在每个ArrayList元素上调用contains(Collection)方法。它们不会收缩（shrink）内部数组。

这些方法可能用于扫描列表中的某些值，然后把它们添加到一个单独的集合并从原列表中删除的场景。为什么有人会这样做？可能因为他习惯于使用for-each循环，而for-each循环不允许从遍历的列表中移除元素，或任何其他原因。例如，你阅读过有关单个remove方法的性能问题，想通过使用那些方法来避免该问题。

你可以通过一种方式来清理集合中被删除的元素，要么使用nulls代替删除的元素，要么维护另一个存放所有删除元素索引的集合（根据集合中元素的数量和打算删除的元素数量，你可以使用数组，Set或BitSet）。下面就是如何删除ArrayList中的所有null值：

    public static <T> void cleanNulls( final List<T> lst )
        {
            int pFrom = 0;
            int pTo = 0;
            final int len = lst.size();
            //copy all not-null elements towards list head
            while ( pFrom < len )
            {
                if ( lst.get( pFrom ) != null )
                    lst.set( pTo++, lst.get( pFrom ) );
                ++pFrom;
            }
            //there are some elements left in the tail - clean them
            lst.subList( pTo, len ).clear();
    }

以上代码只是一种方式。它使用了2个指针-pFrom，表示当前检查的元素，每次遍历都会增加；pTo，表示目的位置，只有在非null元素拷贝时才会增加。由于所有非空元素已被拷贝，留在ArrayList末端的元素也就不再需要（它们已被复制过），我们将使用下面讨论的subList方法清理它们。

**subList(int,int)**

这个方法创建一个当前列表特定部分的视图，并且在Java6和Java7的工作机制是不一样的。

在Java6中，它定义在AbastractList类里。每个子列表都保存一个父列表的引用，并使用父列表的方法添加索引的偏移量（subList的第一个参数）。

在Java7中，子列表是通过调用ArrayList.subList方法创建的，它保存一个原始列表的引用，并直接访问原始列表的elementData数组。

这个区别是不重要的，除非你想使用subList方法写一个递归算法，比如快速排序。Java6中，递归的层次越深，每次方法调用需要传递的subList对象越多（多数情况下，每个递归层次都需要对子列表做范围校验）。Java7中，递归多深并不重要-每次多数方法只需要做一次范围检验，然后就可以访问ArrayList类elementData基础数组中需要的元素。

下面有3个subList方法常见的用例：

* 快速清理部分列表
* 对列表的部分数据进行for-each遍历
* 在递归算法中使用

如果由于某些原因（例如，你需要处理它），你需要清理列表的部分数据，并且你已经知道连续调用remove方法是个坏主意（并且你不知道subList方法），你要么将剩余的元素复制到新的列表中并使用新列表替换老列表，要么调用remove方法，如果你认为它跟其他方式相比还不错的话。

实际上，你只需要调用：`list.subList(from,to).clear()`。不幸的是，从JavaDoc中绝对不能显而易见的看出它会清理源列表的部分数据。尽管如此，它是清理一个子列表的最快方法-这个方法最终只调用了System.arraycopy将剩余的元素移到左边。

为了避免在遍历列表或数组元素时使用索引index变量，Java语言添加了for-each循环。但如果只需要遍历列表的一部分那你应该怎么做呢？仍旧使用index变量？没必要－你可以遍历子列表的所有元素。下面的方法计算一个字符串列表给定部分中所有字符串的总长度。

	public static int getTotalLength( final List<String> lst, final int from, final int to )
	{
	    int sum = 0;
	    for ( final String s : lst.subList( from, to ) )
	    {
	        sum += s.length();
	    }
	    return sum;
	}

子列表的最后使用案例是递归算法。如前所述，在Java6中最好不要在递归算法中使用子列表（sublists）。Java7中sublist性能有了明显提升。这里有个小的测试方法－和"total length"一样，只是采用递归实现：

	public static int getTotalLengthRec( final List<String> lst )
	{
	    if ( lst.size() == 1 )
	        return lst.get( 0 ).length();
	    final int middle = lst.size() >> 1;
	    return getTotalLengthRec( lst.subList( 0, middle ) ) + getTotalLengthRec( lst.subList( middle, lst.size() ) );
	}

基准测试中，我们使用了一个包含1M短字符串的列表，调用这个方法1000次。Java6中耗时78秒，Java7中耗时22秒。在这些案例中，我们应该怎么做才能避免依赖于Java版本？手动处理索引。下面是同样的递归方法，但它使用显式的边界。

	public static int getTotalLengthRec2( final List<String> lst )
	{
	    return getTotalLengthRecHelper( lst, 0, lst.size() );
	}
	 
	public static int getTotalLengthRecHelper( final List<String> lst, final int from, final int to )
	{
	    if ( to - from <= 1 )
	        return lst.get( from ).length();
	    final int middle = ( from + to ) >> 1;
	    return getTotalLengthRecHelper( lst, from, middle ) + getTotalLengthRecHelper( lst, middle, to );
	}

调用1000次getTotalLengthRec2方法在Java6中耗时7.8秒，Java7中耗时9.3秒。也就是说，在Java6中快了10倍，Java7中快了2倍。

**get(int)**

提到这个方法也就一句话，它在Java7中比Java6慢了大概1/3，因为在Java7中它使用额外的方法去访问内部数组（Java6直接访问数组）。本来期望JIT能够内联这些简单的方法以此消除Java6和Java7的不同，但事实看起来并不是这样。也许在Java7的发布版可以解决这个问题。不管怎样，这个方法仍旧很快，在成千上万的访问上你看不出任何不同。
