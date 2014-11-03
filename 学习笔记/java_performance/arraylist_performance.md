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
