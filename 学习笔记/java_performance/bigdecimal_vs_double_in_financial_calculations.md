[使用 double/long 和 BigDecimal 进行货币计算](http://java-performance.info/bigdecimal-vs-double-in-financial-calculations/)
===========

###使用long/double进行货币操作
* 如何正确使用？

最简单的方式：使用本货币的最小单位，比如使用美分代替美元，分代替元，采用long数据类型进行存储。

缺点：对于需要进行乘法/除法操作的地方，需要采用浮点运算，这样计算的结果会受精度影响。
 
 **不要使用float数据类型进行任何货币计算，除非你确定这样做没问题。float精度太低，只有23bits。**

double计算也不精确，即使是简单的加减运算：

`System.out.println( "362.2 - 362.6 = " + ( 362.2 - 362.6 ) );`

输出结果为：`362.2 - 362.6 = -0.4000000000000341`

这意味着我们应该：

1.  在最小的货币单位计算中，避免使用非整数的double值。

2. 根据系统要求，使用Math.round/rint/ceil/floor对乘法/除法计算结果进行取整操作。

PS：只要能遵守上面的两条建议，还是能够使用long数据类型进行加减运算的。

* 什么情况下使用？

先看一个使用double和BigDecimal进行货币操作的测试用例，分别使用double和BigDecimal计算362.2￥的1.5%，循环100M次。

`int res = 0;
final BigDecimal orig = new BigDecimal( "362.2" );
final BigDecimal mult = new BigDecimal( "0.015" ); //1.5%
for ( int i = 0; i < ITERS; ++i )
{
    final BigDecimal result = orig.multiply( mult, MathContext.DECIMAL64 );
    if ( result != null ) res++;
}`

我们使用double和long不能完全模拟上面的计算。在下面的代码中，JIT会将常量`Math.round( orig * mult )`移出循环。

`final double orig = 36220; //362.2 in cents
final double mult = 0.015; //1.5%
for ( int i = 0; i < ITERS; ++i )
{
    final long result = Math.round( orig * mult );
    if ( result != 543 ) res++;    //543.3 cents actually
}`

所以，我们使用下面稍微不同的测试用例以提高可比性：

`final double orig = 36220; //362.2 in cents
for ( long i = 0; i < ITERS; ++i )
{
    final long result = Math.round( orig * i );
    if ( result != 543 ) res++;    //compare with something
}`

使用BigDecimal计算时花费4.899秒，使用double计算花费0.58秒。从测试结果可以看出，如果你的计算结果不超过52位(double精度)，并且你坚持遵守上面的两条规则，那你就能使用long/double完成快速，精确的货币计算！

###使用BigDecimal进行货币操作

