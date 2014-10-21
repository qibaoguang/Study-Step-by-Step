####[使用 double/long 和 BigDecimal 进行货币计算](http://java-performance.info/bigdecimal-vs-double-in-financial-calculations/)
 
 * 使用long/double进行货币操作
> 最简单的方式：使用本货币的最小单位，比如使用美分代替美元，分代替元，采用long数据类型进行存储。

 缺点：对于需要进行乘法/除法操作的地方，需要采用浮点运算，这样计算的结果会受精度影响。
 
 **不要使用float数据类型进行任何货币计算，除非你确定这样做没问题。float精度太低，只有23bits。**

double计算也不精确，即使是简单的加减运算：

`System.out.println( "362.2 - 362.6 = " + ( 362.2 - 362.6 ) );`

输出结果为：`362.2 - 362.6 = -0.4000000000000341`

这意味着我们应该：
1.  在最小的货币单位计算中，避免使用非整数的double值。
2. 根据系统要求，使用Math.round/rint/ceil/floor对乘法/除法计算结果进行取整操作。
