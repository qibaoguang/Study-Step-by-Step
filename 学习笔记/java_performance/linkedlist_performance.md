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

这可能是解决该问题的第一种途径－使用indexOf方法找到需要的字符串位置，然后将位置作为参数调用listIterator(int)方法（当然，对于使用特殊的字符串'/'来定义字符串没找到的情况，需要将位置加１）。

	public static void cleanStringListSlow( final LinkedList<String> lst, final String first )
	{
	    final int startPos = lst.indexOf( first ) + 1;
	    final ListIterator<String> iter = lst.listIterator( startPos );
	    while ( iter.hasNext() )
	    {
	        if ( iter.next().length() == 5 )
	            iter.remove();
	    }
	}

不幸的是，在这个示例中，我们平均需要遍历列表长度的1.25倍。首先，indexOf方法为了找到需要的字符串需要遍历列表。最好的情况下，需要的字符串位于列表的第一个。最坏的情况下，它在列表中就不存在，所以我们需要校验列表中的所有元素。平均下来，调用indexOf方法需要遍历列表长度的0.5倍。接着，我们将开始位置传给listIterator(int)方法。这个方法经过优化（跟其他通过索引访问的方法一样，比如get(int)，remove(int)），如果索引属于列表的前半部分，迭代器将从列表的开始处遍历，否则它将从后往回遍历。在我们的示例中，最好的情况是我们需要的字符串刚好是列表的最后一个元素－什么也不用做，因为listIterator方法的参数和列表的长度相等。最坏的情况是需要的元素位于列表的前半部分－listIterator(int)首先从开始处遍历，然后该方法会循环直到列表的尾部，因此访问了整个列表元素。

重写该算法的方法是实现一个自己版本的indexOf方法，它将会返回listIterator。我们清楚的是返回的迭代器应该指向需要的元素，如果它在列表中存在的话（next方法将返回该元素）。当列表中不存在要求的元素时，迭代器应该指向列表的最后位置（hasNext()==false）。让我们假设要求的元素不等于null。

	public static <T> ListIterator<T> findElem( final List<T> lst, final T elem )
	{
	    final ListIterator<T> iter = lst.listIterator();
	    while ( iter.hasNext() )
	    {
	        if ( elem.equals( iter.next() ) )
	        {
	            iter.previous();
	            break;
	        }
	    }
	    return iter;
	}
	
现在前面的方法就可以化简了。我们只需要在列表中不存在要求的元素时获取一个新的ListIterator。

	public static void cleanStringListFast( final LinkedList<String> lst, final String first )
	{
	    ListIterator<String> iter = findElem( lst, first );
	    if ( !iter.hasNext() ) //if element is not present - process the full list<
	        iter = lst.listIterator();
	    while ( iter.hasNext() )
	    {
	        if ( iter.next().length() == 5 )
	            iter.remove();
	    }
	}

因此，作为规则，不要使用任何接收或返回列表中元素位置的方法，特别是在老的遍历风格下：

	final List<Integer> lst = new LinkedList<Integer>();
	for ( int i = 0; i < 100000; ++i )
	    lst.add( i );
	long sum = 0;
	for ( int i = 0; i < 100000; ++i )
	    sum += lst.get( i );
	  
这段代码运行结束竟然耗费难以想象的６秒！不要尝试使用这种方式遍历一个包含上百万元素的LinkedList列表。你会等的不耐烦！该规则的唯一例外是访问或删除列表的第一个或最后一个元素（或少量的前面或后面的元素）。
          
**removeFirst/pollFirst**

使用LinkedList时要记着它不只是一个简单的List，还是一个Deque。在使用LinkedList的代码中经常看到下面的结构：

	public T next()
	{
	    if ( lst.isEmpty() )
	        return null;
	    return lst.removeFirst();
	}       

如果列表不为空，LinkedList.removeFirst（还有LinkedList.remove()）将返回第一个元素，否则抛出NoSuchElementException。该异常是removeFirst需要调用isEmpty进行保护的原因。

这段代码是多余的，因为LinkedList提供的pollFirst方法跟上面提到的next方法完全相同－如果列表为空返回null，否则返回第一个元素。因此，正确的方法可以节省一个检查，并让代码更简洁。同样的情况也适用于removeLast/pollLast。

**批处理**

有时你可能会有一个链表，它包含从不同源获取的数据，你需要单独处理每个源的数据。例如，你有一个真实的根据事件时间戳排序的网络事件日志。日志的每个元素有用于定义网络设备（计算机，路由器等）的IP地址属性。你需要单独处理每个IP地址关联的事件。此外，你不能长时间的收集信息，并单独处理IP地址－它是一个实时日志，因此我们承受不起长时间的处理延迟（在接收到这些事件后，我们不能晚于N秒响应它们或我们有一个很大的网络，所以一次性保存/处理所有的事件将耗费大量内存）。

对于这种数据，LinkedList有一个很好的解决方案－我们可以廉价的删除列表中任何位置的元素。首先，我们需要一个方法，用于从网络日志提取特定IP地址的事件。不要尝试查询元素，然后从列表中将它们删除，即使在你的库中有相应的方法，因为为了完成提取操作它需要２次遍历而不是一次。我们需要在提取方法中使用ListIterator，因为这是遍历元素同时从源列表中删除它们的唯一方式。

	private static final class LogEvent
	{
	    public final int ipv4;
	    public final long time;
	    public final String eventDesc;
	 
	    private LogEvent( final int ipv4, final long time, final String eventDesc ) {
	        this.ipv4 = ipv4;
	        this.time = time;
	        this.eventDesc = eventDesc;
	    }
	}
	 
	private static List<LogEvent> extractForIp( final LinkedList<LogEvent> fullLst, final int ip )
	{
	    final List<LogEvent> res = new ArrayList<LogEvent>( 10 );
	    final ListIterator<LogEvent> iter = fullLst.listIterator();
	    while ( iter.hasNext() )
	    {
	        final LogEvent event = iter.next();
	        if ( event.ipv4 == ip )
	        {
	            res.add( event );
	            iter.remove();
	        }
	    }
	    return res;
	}

现在我们能处理当前时间戳的所有IP地址：

	private static void processIp( final List<LogEvent> lst )
	{
	    //...processing logic here
	}
	 
	private static void processFirstTimestamp( final LinkedList<LogEvent> fullList )
	{
	    if ( fullList.isEmpty() )
	        return;
	    final long firstTime = fullList.getFirst().time;
	    while ( !fullList.isEmpty() && fullList.getFirst().time == firstTime )
	    {
	        final int ip = fullList.getFirst().ipv4;
	        final List<LogEvent> eventsForIp = extractForIp( fullList, ip );
	        processIp( eventsForIp );
	    }
	}
            
不幸的是，当日志中有很多消息或IP地址需要处理时这个算法表现非常糟糕。每次你需要扫描所有的事件但最后只提取很少的消息。比较好的方式是维护一个IP地址到它们对应事件列表的map。采用这种方式，不管是提取还是追加都非常快。为了保持找到IP的原始顺序我们使用LinkedHashMap，如果我们删除一些IP地址的所有实体，稍后又添加相同IP地址的新实体，这个IP地址将添加到遍历顺序的尾部。updateMap方法将输入列表的所有实体添加到map中，并清空输入列表。我们在初始化和截取子序列(subsequent)时才需要调用该方法。

	private static Map<Integer, List<LogEvent>> extractMap( final List<LogEvent> fullLst )
	{
	    final Map<Integer, List<LogEvent>> res = new LinkedHashMap<Integer, List<LogEvent>>( 10 );
	    updateMap( res, fullLst );
	    return res;
	}
	 
	private static void updateMap( final Map<Integer, List<LogEvent>> eventMap, final List<LogEvent> fullLst )
	{
	    for ( final LogEvent event : fullLst )
	    {
	        List<LogEvent> lst = eventMap.get( event.ipv4 );
	        if ( lst == null )
	        {
	            lst = new ArrayList<LogEvent>( 10 );
	            eventMap.put( event.ipv4, lst );
	        }
	        lst.add( event );
	    }
	    fullLst.clear();
	}

现在在接收到新的日志实体的时间里调用updateMap是足够的，它们将被添加到正确的map实体，并且map实体的顺序也不会改变。在此之后，我们只需要一个新的处理逻辑。

	private static void processFirstTimestamp( final Map<Integer, List<LogEvent>> eventMap )
	{
	    if ( eventMap.isEmpty() )
	        return;
	    final Iterator<Map.Entry<Integer, List<LogEvent>>> iter = eventMap.entrySet().iterator();
	    Long firstTime = null;
	    while ( iter.hasNext() )
	    {
	        final Map.Entry<Integer, List<LogEvent>> entry = iter.next();
	        final List<LogEvent> lst = entry.getValue();
	        if ( firstTime == null )
	            firstTime = lst.get(0).time;
	        else if ( lst.get(0).time != firstTime )
	            break;
	        //extract entries for processing
	        iter.remove();
	        processIp(lst);
	    }
	}

下面实现的一个小的测试用例。它产生50个时间戳，每个时间戳对应1000个IP地址，每个IP地址产生100个实体。每次我们产生的实体只是IP地址集稍微不同。前5个时间戳没有经过处理（可以当作缓存），然后对于每个数据的新增部分我们处理一个时间戳（所以我们总是有５个时间戳）。最后只剩下处理过的时间戳。这是一个针对批处理模式的测试方法。测试方法的初始化方式不是使用一个map而是调用`processFirstTimestamp(LinkedList<LogEvent>)`。

	private static void testEventsMap()
	{
	    final long start = System.currentTimeMillis();
	    final LinkedList<LogEvent> lst = new LinkedList<LogEvent>();
	    final Map<Integer, List<LogEvent>> map = new HashMap<Integer, List<LogEvent>>( 1000 );
	    int mlt = 0;
	    for ( long t = 0; t < 50; ++t )
	    {
	        for ( int ip = mlt * 100; ip < 1000 + mlt * 100; ++ip )
	        {
	            for ( int num = 0; num < 100; ++num )
	            {
	                final LogEvent event = new LogEvent( ip, t, "Event " + num );
	                lst.add( event );
	            }
	        }
	        mlt++;
	        if ( mlt > 4 )
	            mlt = 0;
	        updateMap( map, lst );
	        if ( t > 5 )
	            processFirstTimestamp( map );
	    }
	    while ( !map.isEmpty() )
	        processFirstTimestamp( map );
	    System.out.println( "Total time batch = " + ( System.currentTimeMillis() - start ) );
	}

基于map的实现比一般的列表实现快35倍－10秒对351秒。这就是在整个日志事件列表使用多迭代器的代价。

##总结
如果你想在自己的代码中优化LinkedList的性能，试着遵守这些规则：

* 对基于队列的算法考虑使用ArrayDeque
* 使用LinkedList的ListIterator
* 避免使用任何接收或返回列表元素索引的LinkedList方法，它们跟性能没有一丁点关系
* 检查是否有必要使用LinkedList.remove/removeFirst/removeLast方法，没有必要的话使用pollFirst/pollLast代替
* 尝试对LinkedList进行批处理

##See also

[Java集合概述](java_collections_overview.md)
