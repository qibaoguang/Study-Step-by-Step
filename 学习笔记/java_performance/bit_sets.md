位集(Bit sets)
============
你偶尔会需要映射一个整型的键到一个或一些标记（boolean值），位集最适合这样的场景。

核心Java中只有一个位集实现：java.util.BitSet。它内部使用一个long[]数组，每个位映射到一个整数：第一个long的第０位映射到第０键，第１位映射到键１，依次类推。这意味着如果你想从高位映射键，比如将100,000,000映射成布尔值，那不可变的BitSet可能不是最好的选择，因为它会为从０到你的位集中最高位的键全部分配位。对于一个只在位置100,000,000存在位的位集来说，这意味着分配12.5M内存。

高基数问题可以通过在所有的位集操作中加上和减去这个基数来避免（当然，你需要事先知道这个基数）。如果需要映射负数值到布尔值你也应该这样做：java.util.BisSet不支持负数类型的键。

第三个需要避免的问题是映射超出int值范围的大数值。原始的java.util.BitSet只支持整型键。

最后，但并非最不重要的是划分问题－如果你需要在这存储一些值，在那存储一些值－你仍旧需要分配用于存储从0到位集最高位的键的全部内存。


**处理BitSet问题的LongBitSet**

让我们尝试处理上面的那些问题。我们将实现一个LongBitSet－一个和java.util.BitSet相似的类，唯一的区别是它的键是long而不是int。

LongBitSet底层采用Map<Long,BitSet>实现。我们的想法是使用位集键的高位作为map的键，低位作为值域BitSet的键。这样我们将支持划分（分区，partitioning）和long键，问题解决。

如何处理每个BitSet的大小？这取决于你的数据，但在每个BitSet中保持百万级的键通常是不错的选择。保存一个LongBitSet的全部数量，每个位集需要的内存不会超过128K，甚至更少。

	public class LongBitSet
	{
	    /** Number of bits allocated to a value in an index */
	    private static final int VALUE_BITS = 20; //1M values per bit set
	    /** Mask for extracting values */
	    private static final long VALUE_MASK = ( 1 << VALUE_BITS ) - 1;
	 
	    /**
	     * Map from a value stored in high bits of a long index to a bit set mapped to the lower bits of an index.
	     * Bit sets size should be balanced - not to long (otherwise setting a single bit may waste megabytes of memory)
	     * but not too short (otherwise this map will get too big). Update value of {@code VALUE_BITS} for your needs.
	     * In most cases it is ok to keep 1M - 64M values in a bit set, so each bit set will occupy 128Kb - 8Mb.
	     */
	    private Map<Long, BitSet> m_sets = new HashMap<Long, BitSet>( 20 );
	 
	    /**
	     * Get set index by long index (extract bits 20-63)
	     * @param index Long index
	     * @return Index of a bit set in the inner map
	     */
	    private long getSetIndex( final long index )
	    {
	        return index >> VALUE_BITS;
	    }
	 
	    /**
	     * Get index of a value in a bit set (bits 0-19)
	     * @param index Long index
	     * @return Index of a value in a bit set
	     */
	    private int getPos( final long index )
	    {
	        return (int) (index & VALUE_MASK);
	    }
	 
	    /**
	     * Helper method to get (or create, if necessary) a bit set for a given long index
	     * @param index Long index
	     * @return A bit set for a given index (always not null)
	     */
	    private BitSet bitSet( final long index )
	    {
	        final Long iIndex = getSetIndex( index );
	        BitSet bitSet = m_sets.get( iIndex );
	        if ( bitSet == null )
	        {
	            bitSet = new BitSet( 1024 );
	            m_sets.put( iIndex, bitSet );
	        }
	        return bitSet;
	    }
	 
	    /**
	     * Set a given value for a given index
	     * @param index Long index
	     * @param value Value to set
	     */
	    public void set( final long index, final boolean value )
	    {
	        if ( value )
	            bitSet( index ).set( getPos( index ), value );
	        else
	        {  //if value shall be cleared, check first if given partition exists
	            final BitSet bitSet = m_sets.get( getSetIndex( index ) );
	            if ( bitSet != null )
	                bitSet.clear( getPos( index ) );
	        }
	    }
	 
	    /**
	     * Get a value for a given index
	     * @param index Long index
	     * @return Value associated with a given index
	     */
	    public boolean get( final long index )
	    {
	        final BitSet bitSet = m_sets.get( getSetIndex( index ) );
	        return bitSet != null && bitSet.get( getPos( index ) );
	    }
	 
	    /**
	     * Clear all bits between {@code fromIndex} (inclusive) and {@code toIndex} (exclusive)
	     * @param fromIndex Start index (inclusive)
	     * @param toIndex End index (exclusive)
	     */
	    public void clear( final long fromIndex, final long toIndex )
	    {
	        if ( fromIndex >= toIndex ) return;
	        final long fromPos = getSetIndex( fromIndex );
	        final long toPos = getSetIndex( toIndex );
	        //remove all maps in the middle
	        for ( long i = fromPos + 1; i < toPos; ++i )
	            m_sets.remove( i );
	        //clean two corner sets manually
	        final BitSet fromSet = m_sets.get( fromPos );
	        final BitSet toSet = m_sets.get( toPos );
	        ///are both ends in the same subset?
	        if ( fromSet != null && fromSet == toSet )
	        {
	            fromSet.clear( getPos( fromIndex ), getPos( toIndex ) );
	            return;
	        }
	        //clean left subset from left index to the end
	        if ( fromSet != null )
	            fromSet.clear( getPos( fromIndex ), fromSet.length() );
	        //clean right subset from 0 to given index. Note that both checks are independent
	        if ( toSet != null )
	            toSet.clear( 0, getPos( toIndex ) );
	    }
	 
	    /**
	     * Iteration over all set values in a LongBitSet. Order of iteration is not specified.
	     * @param proc Procedure to call. If it returns {@code false}, then iteration will stop at once
	     */
	    public void forEach( final LongProcedure proc )
	    {
	        for ( final Map.Entry<Long, BitSet> entry : m_sets.entrySet() )
	        {
	            final BitSet bs = entry.getValue();
	            final long baseIndex = entry.getKey() << VALUE_BITS;
	            for ( int i = bs.nextSetBit( 0 ); i >= 0; i = bs.nextSetBit( i + 1 ) ) {
	                if ( !proc.forEntry( baseIndex + i ) )
	                    return;
	            }
	        }
	    }
	}
	
遍历java.util.BitSet中的位通常这样做：
	for ( int i = bs.nextSetBit( 0 ); i >= 0; i = bs.nextSetBit( i + 1 ) ) {
	    //i is a key which bit was set
	}

如果是LongBitSet，可以很容易的实现类似于visitor的访问：
	public void printAllSetBits()
	{
	    forEach( new LongProcedure() {
	        @Override
	        public boolean forEntry( final long value ) {
	            System.out.println( value );
	            return true;
	        }
	    });
	}

**LongBisSet使用场景**

最简单的情况当然是每个整型键存储一个标记。使用LongBitSet，你可以在这保存一些密集值的映射，在那保存一些到单一标记的映射，从内存消耗这点看，它比原来的BitSet要好。

实际上，这里有另一个极其相似的例子－检查你代码中的Set<Short/Integer/Long>。逻辑上，它们和位集相同：一个set中的值要么存在要么不存在，所以从一个整型到布尔值，我们的映射是相同的。下面从[Trove 章节](http://java-performance.info/primitive-types-collections-trove-library/)拷贝过来的表格告诉我们一个包含10M整数的map消耗的内存：

<table border="1">
<thead>
<tr>
<td>JDK HashSet&lt;Integer&gt;</td>
<td>Trove THashSet&lt;Integer&gt;</td>
<td>Trove TIntSet</td>
</tr>
</thead>
<tbody>
<tr>
<td>525M</td>
<td>236M</td>
<td>103M</td>
</tr>
</tbody>
</table>

