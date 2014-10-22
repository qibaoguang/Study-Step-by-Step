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

>1.  在最小的货币单位计算中，避免使用非整数的double值。

>2. 根据系统要求，使用Math.round/rint/ceil/floor对乘法/除法计算结果进行取整操作。

PS：只要能遵守上面的两条建议，还是能够使用long/double数据类型进行加减运算的。

* 什么情况下使用？

先看一个使用double和BigDecimal进行货币操作的测试用例，分别使用double和BigDecimal计算362.2￥的1.5%，循环100M次。

>    int res = 0;

>    final BigDecimal orig = new BigDecimal( "362.2" );

>    final BigDecimal mult = new BigDecimal( "0.015" ); //1.5%

>    for ( int i = 0; i < ITERS; ++i )

>    {

>     final BigDecimal result = orig.multiply( mult, MathContext.DECIMAL64 );
    
>     if ( result != null ) res++;
    
>    }

我们使用double和long不能完全模拟上面的计算。在下面的代码中，JIT会将常量Math.round( orig * mult )移出循环。

>    final double orig = 36220; //362.2 in cents

>    final double mult = 0.015; //1.5%

>    for ( int i = 0; i < ITERS; ++i )

>    {

>     final long result = Math.round( orig * mult );

>     if ( result != 543 ) res++;    //543.3 cents actually
     
>    }

所以，我们使用下面稍微不同的测试用例以提高可比性：

>    final double orig = 36220; //362.2 in cents

>    for ( long i = 0; i < ITERS; ++i )

>    {

>    final long result = Math.round( orig * i );
    
>    if ( result != 543 ) res++;    //compare with something
    
>   }

使用BigDecimal计算时花费4.899秒，使用double计算花费0.58秒。从测试结果可以看出，如果你的计算结果不超过52位(double精度)，并且你坚持遵守上面的两条规则，那你就能使用long/double完成快速，精确的货币计算！

###使用BigDecimal进行货币操作

* 如何正确使用?

对于BigDecimals，如果需要定义取整模式和精度，可以使用MathContext类。该类中预定义了一些常量，比如MathContext.DECIMAL32/DECIMAL64/DECIMAL128，可用于模拟float/double/decimal_128算术运算，而不会出现任何rounding问题。MathContext.UNLIMITED是MathContext默认的值。

>加减运算中，你可以不定义MathContext，但乘除运算中最好定义DECIMAL*上下文中的一个。因为，乘除运算在计算结果为无限小数时需要定义精度，比如１除３。否则将会抛出ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result。

可以尝试运行代码：

>final BigDecimal three = new BigDecimal( "3" );

>try{

>	System.out.println( BigDecimal.ONE.divide( three ) );
	
>}

>	catch ( ArithmeticException ex )
	
>{

>	System.out.println( "Got an exception while calculating 1/3 ex.getMessage() );
	
>}

* BigDecimal性能如何？

测试用例：计算10M E*E+E的和，其中E=Math.E

>BigDecimal res = BigDecimal.ZERO;

>final BigDecimal a = new BigDecimal( Math.E, context );

>final BigDecimal b = new BigDecimal( Math.E, context );

>final BigDecimal c = new BigDecimal( Math.E, context );

>for ( int i = 0; i < 10000000; ++i )

> {

>    final BigDecimal val = a.multiply( b, context ).add( c, context );
    
>   　res = res.add( val, context );
   　
> }

使用double，没有设置MathContext，设置不同的MathContext的测试结果：

|	类型	 | 	耗时(秒) |	计算结果 		|
| -------|---------|---------------------|
| double | 0.018Sec	| 1.010733792587689E8| 
|noMathContext|4.1sec|101073379.273896945320908905278183855697464192452494578591950602844407036684515333035960793495178222656250000000|
|MathContext.UNLIMITED|3.9sec|101073379.273896945320908905278183855697464192452494578591950602844407036684515333035960793495178222656250000000|
|MathContext.DECIMAL32|4.2sec|100000000|
|MathContext.DECIMAL64|9.5sec|101073379.2938854|
|MathContext.DECIMAL128|13.9sec|101073379.2738969453209089052948157|

从测试结果中可以看出，使用BigDecimal进行运算开销很大，在可以避免的情况下需要尽量避免。比如，有个String型的数值，需要除以10的n次方(n为输入)，采用对输入进行小数点移位会更快！对于double类型的数值，乘以或除以２的幂，一般情况都会得到正确的结果，因为浮点数值的指数部分表示２的幂。

###数值转字符串
浮点数转字符串相当困难，比如双精度double转换，你需要知道浮点数的二进制表示形式(IEEE-754)，具体实现可以参考JDK sun.misc.FloatingDecimal类。

* Java6转换Double对象到String需要经过一系列调用：

>Double

>public String toString() {

>    return String.valueOf(value);

>}

>String

>public static String valueOf(double d) {

>  return Double.toString(d);
  
>}

>Double

>public static String toString(double d) {

>  return new FloatingDecimal(d).toJavaFormatString();

>}


* Java7中就非常直接：

>jdk 7 Double

>public String toString() {

>  return toString(value);

>}

* BigDecimal转换为String

BigDecimal有３个用于转换为String的方法：toString，toPlainString和toEngineeringString。toString会缓存toEngineeringString的结果，用于后续的调用(这样做可能是因为BigDecimal的值是不可变的)。下面测试了将Math.E转为字符串10M次耗时：

Double.toString(double)|BigDecimal.toPlainString|BigDecimal.toEngineeringString
-----------------------------|---------------------------------|---------------
4.1 sec	 |12.4 sec	|12.5 sec

最好不要将double转换为BigDecimal，因为double会先转为String，然后String转为BigDecimal。如果你的算法输入为String，那就直接将它转换为BigDecimal，这样你可以避免rounding错误！

###总结
* 如果你想使用Java实现快速且正确的货币计算操作，你应该遵守以下规则：
	1. 将货币值以最小的货币单位(比如分)存放到long类型变量中。
	2. 如果以最小的货币单位计算时使用了double类型，要避免产生非整数的值，否则会产生精度问题。
	3. 使用long类型进行加减运算。
	4. 根据系统需求，使用Math.round/rint/ceil/floor对乘除结果进行取整。
	5. 计算要满足double精度（52位）。

* 为了避免产生无限小数时抛出ArithmeticException，在BigDecimal乘法和除法运算中需要使用MathContext。由于MathContext.UNLIMITED相当与没有使用上下文，所以不建议使用它。

* 可能的情况，不要将double转化为BigDecimal，而是将String转化为BigDecimal。

See alse：
[实现一个高性能的Money类](http://java-performance.info/high-performance-money-class/)
