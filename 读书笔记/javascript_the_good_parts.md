JavaScript语言精粹
====================
### 前言

约定：=> 表示参考相关文章或书籍; JS是JavaScript的缩写。

本书专注于JavaScript的精华部分，同时会偶尔警告要去避免的糟粕部分。作者提炼出的JavaScript精华子集，更可靠，更易读，更易于维护。

本书的目的是揭示JavaScript中的精华，让大家知道它是一门杰出的动态编程语言。

[JavaScript：世界上最被误解的语言](http://javascript.crockford.com/javascript.html)

### 第1章 精华
* 为什么要使用JavaScript语言

  1. 你没有选择，JavaScript是唯一一门所有浏览器都可以识别的语言。
  2. JavaScript有缺陷，但它真的很优秀。轻量级又富有表现力，一旦熟练掌握就能体会到函数式编程的乐趣。

* 分析JavaScript

JavaScript优秀的思想：函数，弱类型，动态对象，对象字面量。糟糕的思想：基于全局变量的编程模型。

函数：基于词法来划分作用域，而不是动态划分作用域 =>《JavaScript权威指南》第5章“8.8.1 词法作用域”。

弱类型：强类型允许编译器在编译时检测错误。事实证明，强类型并不会让你的测试工作变得轻松。弱类型是自由的，不需要建立复杂的类层次，也不用做强制类型转换。

对象字面量：通过列出对象的组成部分，它们就能简单地被创建出来。JSON的灵感来源于此（作者是JSON的创立者）。

全局变量：JavaScript依赖于全局变量来进行连接。所有编译单元的所有顶级变量被撮合到一个被称为全局对象(the global object)的公共命名空间中。全局变量是魔鬼，而它们在JavaScript中却是基础，非常糟糕！

* 一个简单的实验场

### 第2章 语法
* 空白

空白可能表现为被格式化的字符或注释的形式。空白通常没有意义，但有时候必须要用它来分隔字符序列，否则它们就会被合并成一个符号。

```javascript
var that = this; //var和that之间的空格不能移除，其他的空格都可以移除
```

注释：JavaScript提供两种注释形式，块注释和行注释。注释应该被优先用来提高程序的可读性。

**注释一定要精确地描述代码,没有用的注释比没有注释更糟糕!**

块注释：/* */，由于这些字符可能出现在正则表达式字面量里，所以不建议使用块注释。

行注释：//，建议使用行注释替代块注释。

* 标识符

标识符由一个字母开头（JS规范中还允许以下划线_和美元符$开头），其后可选择性地加上一个或多个字母，数字或下划线。

标识符不能使用的保留字：abstract，boolean，break，byte，case，catch，char，class，const，continue，debugger，default，delete，do，double，else，enum，export，extends，false，final，finally，float，for，function，goto，if，implements，import，in，instanceof，int，interface，long，native，new，null，package，private，protected，public，return，short，static，supper，switch，synchronized，this，throw，throws，transient，true，try，typeof，var，volatile，void，while，with。

该列表中不包括一些本应该被保留而没有保留的字，诸如undefined，NaN和Infinity。

JS不允许使用保留字来命名变量或参数。更糟糕的是，JS不允许在对象字面量中，或者用点运算符提取对象属性时，使用保留字作为对象的属性名。

标识符被用于语句，变量，参数，属性名，运算符和标记。

* 数字

JS只有一个数字类型，在内部被表示为64位的浮点数，和Java的double数字类型一样。与其他大多数编程语言不同的是，它没有分离出整数类型，所以1和1.0的值相同。这避免了短整型的溢出问题和一大堆因数字类型导致的错误。

数字字面量有指数部分，则这个字面量的值等于e之前的数字与10的e之后数字的次方相乘。100=1e2。

负数：前置运算符-加数字。-100。

NaN：一个数值，表示不能产生正常运算结果。NaN不等于任何值，包括它自己。可以使用函数isNaN(number)来检测NaN。

Infinity：表示所有大于1.79769313486231570e+308的值。

数字拥有方法，JS中的Math对象包含一套作用于数字的方法。Math.floor(number)可以将一个数字转换为一个整数。

* 字符串

字符串字面量可以被包在一对单引号或双引号中，可能包含0个或多个字符。\反斜杠是转义字符。JS被创建的时候，Unicode是一个16位的字符集，所以JS中的所有字符都是16位的。

JS没有字符类型。要表示一个字符，只需创建仅包含一个字符的字符串即可。

转义字符：用于把正常情况下不被允许的字符插入到字符串中，比如反斜线，引号和控制符。\u约定用来指定数字字符编码。"A" == "\u0041"。

字符串是不可变的，可以通过length属性获取长度，通过+连接其他字符串。两个包含完全相同的字符且字符顺序也相同的字符串被认为是相同的字符串，'c' + 'a' + 't' === 'cat'。

字符串有相应的方法，比如'cat'.toUpperCase() === 'CAT'。

* 语句

var：var语句用于函数内部，则定义的是这个函数的私有变量。

label：swith，while，for和do语句允许有一个可选的前置标签（label），它配合break语句来使用。

语句执行顺序：通常按照从上到下的顺序执行，JS可以通过条件语句（if和switch），循环语句（while，for和do），强制跳转语句（break，return和throw）和函数调用来改变执行序列。

代码块：包在一对花括号中的一组语句。JS中的代码块不会创建新的作用域，因此变量应该定义在函数的头部，而不是在代码块中。

JS的假值：false，null，undefined，空字符串' '，数字0，数字NaN，其他所有的值都被当做真，包括true，
字符串"false"，以及所有的对象。

JS中的语句，比如if，switch，while，for，for in，do，try catch，throw，
return（没有指定返回表达式，则返回undefined），break，和Java中的语义相同。JS不允许在return关键字和表达式之间换行，也不允许在break关键字和标签之间换行。

* 表达式

表达式：最简单的表达式是字面量值（比如字符串或数字），变量，内置的值（true，false，null，undefined，NaN和Infinity），以new开头的调用表达式，以delete开头的属性提取表达式，包在圆括号中的表达式，以一个前置运算符作为前导的表达式，或者表达式后面跟着：

  1. 一个中置运算符与另一个表达式;
  2. 三元运算符？后面跟着另一个表达式，然后接一个：，再然后接第3个表达式;
  3. 一个函数调用;
  4. 一个属性提取表达式。
  
运算符优先级：下表中排在越上面的优先级越高，结合性越强。圆括号可以改变正常情况下的优先级。

|运算符  |说明               |
| ------ | -----------------:|
|() . []| 调用函数与提取属性|
|delete new typeof + - ! |一元运算符|
|* / % | 乘法，除法，求余|
|+ - |加法/连接，减法|
|>= <= > <|不等式运算符|
|=== !==|等式运算符|
|&&|逻辑与|
| II | 逻辑或|
| ?: | 三元|

typeof：typeof运算符产生的值有'number'，'string'，'boolean'，'undefined'，'function'和'object'。如果运算符是一个数组或null，则结果是'object'，其实不应该是这样的！（作者意思是应该为array或null）。

函数调用运算符：函数调用引发函数的执行，函数调用运算符是跟随在函数名后面的一对圆括号。圆括号中可能包含传递给这个函数的参数。

属性存取表达式：用于获取或设置一个对象或数组的属性和元素。

* 字面量

字面量(literal)：包括number字面量，string字面量，object字面量，array字面量，function，regexp字面量（正则表达式）。

对象字面量：一种可以方便地按指定规格创建新对象的表示法。属性名可以是标识符或字符串，这些名字被当作字面量名而不是变量名来对待，所以对象的属性名在编译时才能知道。属性的值就是表达式。

数组字面量：一种可以方便地按指定规格创建新数组的表示法。

* 函数

函数字面量：定义函数值，可以指定可选的名字，用于递归地调用自己。可以指定参数列表，函数主体包括变量定义和语句。

### 第3章 对象

简单数据类型：JS的简单数据类型包括数字，字符串，布尔值（true和false），null值和undefined值。这些类型虽然拥有方法，但它们是不可变的，所以不能称为对象。

对象：JS中的对象是可变的键控集合（keyed collections）。在JS中，数组，函数，正则表达式都是对象。
对象是属性的容器，其中每个属性都拥有名字和值。属性的名字可以是空字符串在内的任意字符串。属性值可以是除undefined值以外的任何值。JS里的对象是无类型的，且允许对象继承和嵌套。

* 对象字面量

对象字面量：一个对象字面量就是包围在一对花括号中的零或多个"名/值"对，它可以方便的创建新对象值。对象字面量可以出现在任何允许表达式出现的地方。如果属性名是一个合法的JS标识符且不是保留字，则并不强制要求用引号括住属性名。JS的标识符包含连接符（-）是不合法的，但允许包含下划线（_）。

* 检索

检索对象：可以使用.或[]检索对象，优先考虑使用.表示法，因为它更紧凑且可读性更好。如果字符串表达式不是合法的JS标识符，则必须使用[]来检索对象。

检索一个不存在的成员属性的值将返回undefined，可以使用||运算符填充默认值。
```javascript
var middle = stooge["middle-name"] || "(none)" ;
```

尝试从undefined的成员属性中取值将导致TypeError异常，可以通过&&运算符避免错误。
```javascript
flight.equipment //undefined
flight.equipment.model //throw 'TypeError'
flight.equipment && flight.equipment.model //undefined
```

* 更新

更新对象：对象的值可以通过赋值语句来更新。如果属性值已经存在于对象里，则这个属性的值会被替换，否则该属性会被扩充到对象中。

* 引用

对象引用：对象通过引用来传递，它们永远不会被复制。
```javascript
var a={},b={},c={}; //a,b,c引用不同的空对象
a=b=c={}; //a,b,c引用相同的空对象
```

* 原型

原型：每个对象都连接到一个原型对象，并且它可以从中继承属性。所有通过对象字面量创建的对象都连接到Ojbect.prototype，它是JS中的标配对象。

原型选择：当创建一个新对象时，可以选择某个对象作为它的原型，JS提供的实现机制杂乱而复杂，其实可以被明显地简化。

原型选择简化方法：为Object增加一个create方法，这个方法创建一个使用原对象作为其原型的新对象。
```javascript
if(typeof Object.create !== 'function'){ //书中代码为Object.beget,印刷错误？
    Object.create = function(o){
      var F = function(){};
      F.prototype = o;
      return new F();
    }
}
var another_stooge = Object.create(stooge);
```

原型连接与委托机制：原型连接在更新时不起作用，当对某个对象做出改变时，不会触及该对象的原型。原型连接只在检索值的时候才被用到。如果尝试获取对象中不存在的属性值，则JS会试着从原型对象中获取该属性值。如果原型对象也没有该属性，则继续从原型对象的原型中寻找，依此类推，直到到达终点Object.prototype。如果仍旧找不到，则返回undefined。这个过程就是委托。

原型关系是一种动态的关系。如果向原型中添加一个新的属性，则该属性会立即对所有基于该原型创建的对象可见。

* 反射

反射：检查对象并确定对象的属性。typeof操作符可以方便的确定属性的类型。

处理不需要的属性：当你想让对象在运行时动态获自身信息时，关注更多的是数据，这时应该让你的程序做检查并丢弃掉值为函数的属性。使用hasOwnProperty方法可以检查对象是否拥有独有的属性，如果有则返回true，它不会检查原型链。
```javascript
flight.hasOwnProperty('number')       //true
flight.hasOwnProperty('constructor')  //false
```

* 枚举

for in：可用来遍历一个对象中的所有属性名。该枚举过程将会列出所有的属性-包括函数和原型中的属性，这些一般都需要过滤掉。最常用的过滤器是hasOwnProperty方法，及使用typeof排除函数。for in遍历，
属性名出现的顺序是不确定的。如果想要确保属性以特定的顺序出现，则创建一个数组，将属性以正确的顺序放入，使用for获取它们的值。

* 删除

delete运算符：用于删除对象的属性。如果对象包含该属性，则该属性会被移除。它不会触及原型中的任何对象。删除对象的属性可能会让来自原型链中的属性透现出来。

* 减少全局变量污染

JS的全局变量：JS可以很随意地定义全局变量来容纳你的应用的所有资源。遗憾的是，全局变量会削弱程序的灵活性，应该避免使用。

最小化全局变量：为你的应用只创建一个唯一的全局变量！（后面会介绍另一种有效减少全局污染的方法：闭包）
```javascript
var MYAPP = {}; //命名空间，整个应用的容器
MYAPP.stooge={
  "first-name":"Joe",
  "last-name":"Howard"
};
```

### 第4章 函数
JS设计最出色的就是它的函数的实现，几乎接近于完美！

函数：函数包含一组语句，它们是JS的基础模块单元，用于代码复用，信息隐藏和组合调用。函数用于指定对象的行为。一般来说，编程就是将一组需求分解为一组函数与数据结构的技能。（编程如此简单？？）

* 函数对象

函数对象：JS中的函数就是对象。对象是"名/值"对的集合并拥有一个连到原型对象的隐藏连接。对象字面量产生的对象连接到Object.prototype。函数对象连接到Function.prototype（该原型对象本身连接到Object.prototype）。每个函数在创建时会附加两个隐藏属性：函数的上下文和实现函数行为的代码（类似于句柄）。

* 函数字面量

函数字面量：包括4个部分，function + 函数名（可以省略，匿名函数）+ 圆括号中的参数列表 + 花括号中的语句。函数对象是通过函数字面量来创建的。函数字面量可以出现在任何允许表达式出现的地方。

闭包：函数可以被定义在其他函数中。一个内部函数除了可以访问自己的参数和变量，同时它也能自由访问父函数的参数和变量。通过函数字面量创建的函数对象包含一个连到外部上下文的连接。这被成为闭包（closure）。它是JS强大表现力的来源。

* 调用

函数调用：调用一个函数会暂停当前函数的执行，传递控制权和参数给新函数。除了声明时定义的形参，每个函数还接收两个附加的参数：this和arguments。

this与调用模式：参数this的值取决于调用的模式。在JS中，一共有4种调用模式：方法调用模式，函数调用模式，构造器调用模式和apply调用模式。这些模式在如何初始化关键参数this上存在差异。

调用运算符：调用运算符是跟在任何产生一个函数值的表达式之后的一对圆括号。圆括号内包含参数列表。实际参数（arguments）个数与形式参数（parameters）个数不匹配时，不会导致运行时错误。如果实参过多，则超出的参数值会被忽略。如果实参过少，缺失的值会被替换为undefined。对参数值不会进行类型检查：任何类型的值都可以被传递给任何参数。

* 方法调用模式

方法：当一个函数被保存为对象的一个属性时，我们称之为方法。当方法被调用时，this被绑定到该对象。如果调用表达式包含一个提取属性的动作（.或[]），那它就是被当作一个方法来调用。
```javascript
var myObj = { //创建myObj对象，有一个value属性和一个increment方法.
  value: 0,
  increment: function(inc){ //increment方法接受一个可选的参数。如果该参数不是数字，则默认使用1.
    this.value += typeof inc === 'number' ? inc : 1;
  }
}

myObj.increment();
document.writeln(myObj.value) // 1

myObj.increment();
document.writeln(myObj.value) //3
```

方法可以使用this访问自己所属的对象，所以它能从对象中取值或对对象进行修改。this到对象的绑定发生在调用的时候，这样的延迟绑定使得函数可以高度复用this。

公共方法：通过this可取得它们所属对象的上下文的方法称为公共方法。

* 函数调用模式

函数调用：当一个函数并非一个对象的属性时，那么它就是被当做一个函数来调用的。以此模式调用函数时，this被绑定到全局对象。这是语言设计的一个错误！如果设计正确，那么当内部函数被调用时，this应该仍然绑定到外部函数的this变量。这个错误设计的后果是方法不能利用内部函数来帮助它工作，因为内部函数的this被绑定了错误的值，所以不能共享该方法对对象的访问权。

解决方案：在外部方法中定义一个变量that，并赋值为this，内部函数可以通过that访问this。
```javascript
myObj.double = function(){ //给myObj增加一个double方法
    var that = this; //解决方法
    var helper = function(){ //double方法内部的函数
        that.value = add(that.value,that.value); 
    };
    helper();   //以函数的形式调用helper
};

myObj.double(); //以方法的形式调用double
document.writeln(myObj.value); //6
```

* 构造器调用模式

JS是一门基于原型继承的语言，对象可以直接从其他对象继承属性。该语言是无类型的。

构造器函数：函数创建的目的是结合new前缀来调用，那它就被称为构造器函数。按照约定，它们保存在以大写格式命名的变量里。

构造器函数缺点：如果调用构造器函数时，没有在前面加上new，可能会产生非常糟糕的事情，即没有编译时警告，也没有运行时警告，所以约定非常重要。不推荐使用这种形式的构造器函数。（下一章有更好的替代方式）
```javascript
var Quo = function(str){ //创建一个名为Quo的构造器函数，它创建一个带有status属性的对象。
    this.status = str;
};

//给Quo的所有实例提供一个get_status的公共方法
Quo.prototype.get_status = function(){
  return this.status;
};
//构造一个Quo实例
var myQuo = new Quo("confused");
document.writeln(myQuo.get_status()); //打印显示"confused"
```

* Apply调用模式

JS是一门函数式的面向对象编程语言，所以函数可以拥有方法。

apply方法：apply方法允许我们构建一个参数数组传递给调用函数，同时允许我们选择this的值。apply方法接收两个参数，第1个是要绑定给this的值，第2个是一个参数数组。
```javascript
//构建一个包含两个数字的数组，并将它们相加
var array = [3,4]; 
var sum = add.apply(null,array); //sum=7

//构造一个包含status成员的对象
var statusObject = {
  status:'A-OK'
}

//statusObject并没有继承自Qup.prototype，但我们可以在statusObject上调用get_status方法，
//尽管statusObject并没有一个名为get_status的方法。
var status = Quo.prototype.get_status.apply(statusObject); //status='A-OK'
```

* 参数

arguments：函数调用时会隐式传递arguments数组。函数通过此参数能访问所有它被调用时传递给它的参数列表，包括那些没有被分配给函数声明时定义的形参的多余参数。利用该特性可以编写不需要指定参个数的函数，不过不是特别有用。

arguments的语言设计错误：arguments并不是一个真正的数组。它只是一个"类似数组(array-like)"的对象。arguments拥有一个length属性，但它没有任何数组的方法。

* 返回

正常返回：函数从第一个语句开始执行，并在遇到关闭函数体的}时结束。然后把控制权交还给调用该函数的程序。

return：return语句可用来使函数提前返回。当return被执行时，函数立即返回而不再执行余下的语句。

一个函数总会返回一个值。如果没有指定返回值，则返回undefined。如果函数调用时在前面加上new前缀，且返回值不是一个对象，则返回this（该新对象）。

* 异常

throw语句：throw中断函数的执行。它应该抛出一个exception对象（自定义对象），该对象包含一个用来识别异常类型的name属性和一个描述性的message属性，还可以添加其他属性。

try catch：一个try语句只会有一个捕获所有异常的catch代码块（跟Java异常机制不同）。

* 扩充类型的功能

通过给Object.prototype添加方法，可以让该方法对所有对象可用; 通过给Function.prototype增加方法，
可以使该方法对所有函数可用。由于JS原型继承的动态本质，新的方法立刻被赋予到所有的对象实例上，即使对象实例是在方法增加之前创建的。

功能扩充实例：
```javascript
//为Function.prototype增加method方法，方便以后创建新的方法
Function.prototype.method = function(name,func){
    if(!this.prototype[name]){ //没有该方法时才添加
        this.prototype[name] = func;
    }
    return this;
};

//为Number.prototype增加一个integer方法，用于提取数字中的整数部分
Number.method('integer',function(){
  return Math[this<0 ? 'ceil' : 'floor'] (this);
});

//为String添加移除字符串首尾空白的方法
String.method('trim',function(){
    return this.replace(/^\s+|\s+$/g,'');
})
```

* 递归

递归函数：直接或间接地调用自身的函数。

汉诺塔问题：
```javascript
var hanoi = function(disc,src,aux,dst){
    if(disc > 0){
      hanoi(disc-1,src,dst,aux);
      document.writeln('Move disc '+disc+' from '+src+' to '+dst);
      hanoi(disc-1,aux,src,dst);
    }
};
hanoi(3,'Src','Aux','Dst');
```

递归函数操作树形结构：
```javascript
//定义walk_the_DOM函数，它从某个指定的节点开始，按HTML源码中的顺序访问该树的每个节点。
//它会调用传入的函数，并依次传递每个节点给它。walk_the_DOM调用自身去处理每个子节点。
var walk_the_DOM = function walk(node,func){
    func(node);
    node = node.firstChild;
    while(node){
      walk(node,func);
      node = node.nextSibling;
    }
};

//定义getElementsByAttribute函数，它以一个属性名称字符串和一个可选的匹配值作为参数，调用
//walk_the_DOM，传递一个用来查找节点属性名的函数作为参数。匹配的节点累加到结果数组中。
var getElementsByAttribute = function(att,value){
    var results = [];
    walk_the_DOM(document.body, function(node){
        var actual = node.nodeType === 1 && node.getAttribute(attr);
        if(typeof actual === 'string' && (actual === value || typeof value !== 'string')){
              results.push(node);
        }
    });
    return results;
};
```

尾递归优化：一种在函数的最后执行递归调用语句的特殊形式的递归。这意味着如果一个函数返回自身递归调用的结果，那么调用的过程会被替换为一个循环，它可以显著提高速度。But，JS当前没有提供尾递归优化。深度递归的函数可能会因为堆栈溢出而运行失败。
```javascript
//构造一个带尾递归的函数（返回自身调用结果），JS没对这种形式的递归做优化。
var factorial = function factorial(i,a){
    a = a || 1;
    if(i<2){
      return a;
    }
    return factorial(i-1, a*i);
};
document.writenln(factorial(4)); //24
```
* 作用域

作用域：作用域控制变量和参数的可见性及生命周期。For us，作用域减少了名称冲突，并提供了自动内存管理。

JS作用域：**不支持块级作用域**,支持函数作用域。定义在函数中的参数和变量在函数外部不可见，而在函数内部任何位置定义的变量，在该函数内部任何地方都可见。由于JS缺少块级作用域，所以不建议延迟声明变量，最好的做法是在函数体的顶部声明函数中可能用到的所有变量。
```javascript
var foo = function(){
    var a = 3, b = 5;
    var bar = function(){
        var b = 7, c = 11; 
        //此时，a为3, b为7, c为11
        a += b + c; 
        //此时，a为21, b为7, c为11
    };
    //此时，a为3, b为5, 而c还没定义
    bar();
    //此时，a为21, b为5
};
```
* 闭包

作用域的好处是内部函数可以访问定义它们的外部函数的参数和变量。当内部函数拥有比它的外部函数更长的生命周期时，内部函数引用的外部函数变量不会被释放（Java中一般会引起内存泄漏，而JS闭包恰好利用该特性）。

示例：
```javascript
//创建一个myObj对象，把匿名函数调用结果赋值给它。
var myObj = (function(){
  var value = 0;//由于函数作用域，外部对value的操作只能基于return的对象的两个方法。
  //该匿名函数返回一个包含两个方法的对象，并且这些方法继续享有访问value变量的特权。
  return {
    increment: function(inc){
      value += typeof inc === 'number' ? inc : 1;
    },
    getValue: function(){
      return value;
    }
  };
}());
```
重构Quo构造器：
```javascript
//将status作为私有属性，提供对应的getter方法(原版本直接可以访问status，提供getter没意义，还需要显示new)。
var quo = function(status){
  return {
    get_status:function(){
      return status;
    }
  };
};
var myQuo = quo("amazed");//由于不需要加上new，所以名字没有首字母大写
document.writeln(myQuo.get_status());
```
注：当调用quo时，它返回一个包含get_status方法的新对象。该对象的一个引用保存在myQuo中。即使quo已经返回，但get_status方法仍然享有访问quo对象的status属性的特权。get_status方法并不是访问该参数的一个副本，而是参数本身。**因为该函数可以访问它被创建时所处的上下文环境，这被称为闭包。**

加深理解：
```javascript
//糟糕的例子
//构造一个用于显示节点序号的函数，但由于用错误的方式给数组中的节点设置事件处理函数，而造成每次点击总是显示节点的数目
var add_the_handlers = function(nodes){
    var i;
    for(i=0; i<nodes.length; i+=1){
        nodes[i].onclick = function(e){
            alert(i);
        };
    }
};

//改良后的例子
var add_the_handlers = function(nodes){
    var helper = function(i){
        return function(e){
            alert(i);
        };
    };
    var i;
    for(i=0; i<nodes.length; i+=1){
        nodes[i].onclick = helper(i);
    }
};
```
注：避免在循环中创建函数，它可能只会带来无谓的计算，还会引起混淆。建议先在循环之外创建一个辅助函数，让辅助函数返回一个绑定当前i值的函数，这样就不会导致混淆。

* 回调

回调：函数使得对不连续事件的处理变得更容易，因为我们可以注册回调函数，以异步的方式处理请求。

C/S请求响应模式对比：

同步方式：网络上的同步请求会导致客户端进入假死状态，特别是网络传输或服务器很慢时。
```javascript
request = prepare_the_request();
response = send_request_synchronously(request);
display(response);
```
异步方式：发起异步请求，提供一个当服务器响应到达时随即触发的回调函数。异步函数立即返回，这样客户端就不会阻塞。
```javascript
request = prepare_the_request();
send_request_asynchronously(request,function(response){
    display(response);
});
```
* 模块

模块：为了屏蔽JS全局变量的使用，我们可以使用函数和闭包来构造模块。模块是一个提供接口却隐藏状态与实现的函数或对象。

模块模式：模块模式利用了函数作用域和闭包来创建对象与私有成员的关联。在下面示例中，只有deentityify方法有权访问字符实体表这个数据对象。

模块模式的一般形式：一个定义了私有变量和函数的函数; 利用闭包创建可以访问私有变量和函数的特权函数; 最后返回这个特权函数，或把它们保存到一个可访问的地方。

模块模式好处：使用模块模式可以摒弃全局变量的使用。模块模式促进信息隐藏和其他优秀的设计实践，比如单例模式，利于应用程序的封装。模块模式也可以用来产生安全的对象，比如示例2中用来产生序列号的对象。

模块示例：为String增加deentityify方法
```javascript
//寻找字符串中的HTML字符实体并把它们替换为对应的字符。
String.method('deentityify',function(){
//字符实体表。它映射字符实体的名字到对应的字符。把该信息放到全局变量不合适，
//定义在函数的内部会带来运行时的损耗（每次执行此函数，该字面量都会被求值一次）。
var entity = {
    quot: '"',
    lt: '<',
    gt: '>'
}; 
//返回deentityify方法
return function(){
//这才是deentityify方法。它调用字符串的replace方法，查找'&'开头和';'结束的子字符串。
//如果这些字符可以在实体表中找到，则将其替换为映射表中的值。
    return this.replace(/&([^&;]+);/g,function(a, b){
        var r = entity[b];
        return typeof r === 'string' ? r : a ;
    };
};
}());
document.writeln('&lt;&quot;&gt'.deentityify()); //<">
```
模块示例2：产生序列号
```javascript
var serial_marker = function(){
  var prefix = '';
  var seq = 0;
  return {
      set_prefix: function(p){
          prefix = String(p);
      },
      set_seq: function(s){
          seq = s;
      },
      gensym: function(){
          var result = prefix + seq;
          seq += 1;
          return result;
      }
  };
};
//虽然seqer可变，且可以替换它的方法，但替换后的方法依然不能访问私有成员。
var seqer = serial_marker();
seqer.set_prefix('Q');
seqer.set_seq(1000);
var unique = seqer.gensym(); //unique='Q1000'
```
* 级联

级联：在单独一条语句中依次调用同一个对象的多个方法，关键是返回this而不是undefined。JQuery等JS框架常使用级联简化编程。级联技术可以产生极富表现力的接口，并保持单个接口的简单性。

* 柯里化

柯里化：也称局部套用，是把多参数函数转换为一系列单参数函数并进行调用的技术。柯里化允许我们把函数与传递给它的参数相结合，产生出一个新的函数。

柯里化示例：curry
```javascript
//curry方法创建一个保存原始函数和要被套用的参数的闭包来工作。它返回另一个函数，该函数被调用时，
//会返回调用原始函数的结果，并传递调用curry时的参数加上当时调用的参数。
Function.method('curry',function(){
  var args = arguments, that = this;
  return function(){
      //Wrong! 本意使用Array的concat方法连接两个数组，但arguments不是真正的数组，它没有concat方法。
      return that.apply(null, args.concat(arguments)); 
  };  
});
//修正版：对两个arguments应用数组的slice方法，产生拥有concat方法的常规数组。
Function.method('curry',function(){
  var slice = Array.prototype.slice,
      args = slice.apply(arguments); //arguments:传给原方法的参数
      that = this;
  return function(){
     return that.apply(null, args.concat(slice.apply(arguments)));//arguments:传递给curry的参数
  };
});
```
* 记忆

记忆：函数可以将先前操作的结果记录在某个对象里，从而避免无谓的重复运算，这种优化被称为记忆。JS的对象和数组能很方便的实现这种优化。

带记忆功能的函数：memoizer
```javascript
//memoizer取得一个初始的memo数组和formula函数，返回一个管理memo存储和调用formula的recur函数。
//把这个recur函数和它的参数传递给formula函数。
var memoizer = function(memo, formula){
  var recur = function(n){
      var result = memo[n];
      if(typeof result !== 'number'){
          result = formula(recur, n);
          memo[n] = result;
      }
      return result;
  };
  return recur;
};
//应用memoizer产生可记忆的阶乘函数
var factorial = memoizer([1, 1], function(recur, n){
    return n * recur(n-1);
});
```
### 第5章 继承

继承的好处：代码重用; 引入类型系统的规范，程序员无需做显式类型转换。

JS的继承：JS是一门弱类型语言，从不需要类型转换。对于JS对象来说，重要的是它能做什么，而不是它从哪里来，它是什么（鸭式辩型）。JS基于原型的特性，意味着对象直接从其他对象继承，而不像基于类的语言。下面是JS常用的继承模式。

* 伪类

原型与伪类：JS的原型存在诸多矛盾，它的某些复杂的语法看起来就像那些基于类的语言，这些语法问题掩盖了它的原型机制。它不能直接从其他对象继承，反而插入了一个多余的间接层：通过构造器函数产生对象。

当一个函数对象被创建时，Function构造器产生的函数对象会运行类似如下的代码：
```javascript
this.prototype = {constructor:this};
```
新函数对象被赋予一个prototype属性，它的值是一个包含constructor属性且属性值为该新函数的对象。这个prototype对象是存放继承特征的地方。由于JS语言没有提供确定构造器函数的方法（不像Java），所以每个函数都会得到一个prototype对象。constructor属性没什么用，重要的是prototype对象。

当采用构造器调用模式，即用new前缀去调用一个函数时，函数执行的方式会被修改。如果new运算符是一个方法而不是一个运算符，它可能会像这样执行：
```javascript
Function.method('new',function(){
  //创建一个继承构造器函数原型对象的新对象
  var that = Object.create(this.prototype);
  //调用构造器函数，绑定-this-到新对象上
  var other = this.apply(that, arguments);
  //如果它的返回值不是一个对象，就返回该新对象。
  return (typeof other === 'object' && other) || that;
});
```
伪类之伪：伪类本意是想向面向对象靠拢，但它看起来格格不入。没有私有环境，所有的属性都是公开的，无法访问super（父类）的方法。更糟糕的是，使用构造器如果忘记加new前缀，那么this将不会绑定到新对象上，而是绑定到全局对象上。这样不但没有扩充新对象，反而破坏了全局变量环境。出现这种情况事，既没有编译时错误，也没有运行时警告。

放弃伪类：与其时刻担心忘记加new前缀，不如根本就不使用new（即使有构造器函数命名首字母大写的约定）。借鉴类的表示法可能误导程序员去编写过于深入与复杂的层次结构。许多复杂类层次结构产生原因是因为静态类型检查的约束，而JS完全摆脱了那些约束。在基于类的语言中，类继承是代码重用的唯一方式（当然还有组合），而JS有更多且更好的选择。所以放弃使用伪类，坚持JS的本色！

* 对象说明符

对象说明符：使用对象说明符来描述要构建的对象规格说明，而不是传一大串参数，这样可以避免参数顺序的问题，又能和JSON配合。

* 原型

原型，从构造有用对象开始：在一个纯粹的原型模式中，我们会摒弃类，转而专注于对象。基于原型的继承比基于类的继承在概念上更为简单：一个新对象可以继承一个旧对象的属性。通过构造一个有用的对象，接着构造更多类似的对象，这就可以完全避免把一个应用拆解成一系列嵌套抽象类的分类过程。

差异化继承：通过定制一个新的对象，我们指明它与所基于的基本对象的区别。

* 函数化

函数化：前面几种继承模式的一个弱点是没法保护隐私。对象的所有属性都是可见的。我们无法得到私有变量和私有函数。不要试图通过**伪装私有（pretend privacy）**来实现私有属性的保护（给私有属性起个怪模怪样的名字，并希望其他使用代码的用户假装看不到这些奇怪的成员，掩耳盗铃！），应该使用模块模式来完成该效果！

函数化，从构造一个生成对象的函数开始：
  1. 创建一个新对象。构造方式很多，比如构造一个对象字面量，或者new+构造器函数，或者调用任意一个会返回对象的函数。
  2. 有选择地定义私有实例变量和方法。这些就是函数中通过var语句定义的普通变量。
  3. 给这个新对象扩充方法。这些方法拥有特权去访问变量，以及在第2步中通过var语句定义的变量。
  4. 返回这个新对象。

伪代码：
```javascript
//spec对象包含构造器需要构造新实例的所有信息。
var constructor = function(spec, my){
    //声明该对象私有的实例变量和方法  
    var that, 其他私有变量实例;
    //my为继承链中的构造器提供秘密共享的容器,可选
    my = my || {};
    //把共享的变量和函数添加到my中;
    my.member = value;
    .....
    //构造新对象并赋值给that
    that = 一个新对象;
    //扩充that，添加组成该对象接口的特权方法
    var methodical = function(){
      ...
    };
    that.methodical = methodical;
    //返回that
    return that;
};
```
示例：
```javascript
var mammal = function(spec){
    var that = {};
    that.get_name = function(){
      return spec.name;
    };
    that.says = function(){
      return spec.saying || '';
    };
    return that;
};
var myMammal = mammal({name: 'Herb'});
```
父类方法：函数化模式给我们提供处理父类方法的方法。
```javascript
//构造一个superior方法，它取得一个方法名并返回调用那个方法的函数。该函数会调用原来的方法。
Object.method('superior', function(name){
    var that = this, method = that[name];
    return function(){
        return method.apply(that, arguments);
    };
});
```
函数化模式优点：函数化模式有很大的灵活性。相比伪类模式，它不仅带来的工作更少，还让我们得到更好的封装和信息隐藏，以及访问父类方法的能力。如果对象的所有状态都是私有的，那就可以保证对象的完整性不被破坏。如果使用函数化样式创建一个对象，并且该对象的所有方法都不使用this或that，那该对象就是持久性的。一个持久性对象就是一个简单功能函数的集合。一个持久性的对象不会被入侵。访问一个持久性的对象时，除非有方法授权，否则攻击者不能访问对象的内部状态。

* 部件

一套部件组装出对象。

示例：
```javascript
//构造一个给任何对象添加简单事件处理特性的函数。
//它会给对象添加一个on方法，一个fire方法和一个私有的事件注册对象。
var eventuality = function(that){
   var registry = {};
   //在一个对象上触发事件。该事件可以是一个包含事件名称的字符串，或一个拥有包含事件名称的
   //type属性的对象。通过'on'方法注册的事件处理程序中匹配事件名称的函数将被调用。
   that.fire = function(event){
      var array, func, handler, i, type = typeof event === 'string' ? event : event.type;
      //如果这个事件存在一组事件处理程序，则遍历它们并按顺序依次执行。
      if(registry.hasOwnProperty(type)){
          array = registry[type];
          for(i=0; i<array.length; i += 1){
              handler = array[i];
              //每个处理程序包含一个方法和一组可选的参数。
              //如果该方法是一个字符串形式的名字，那么就寻找该函数。
              func = handler.method;
              if(typeof func === 'string'){
                  func = this[func];
              }
              //调用一个处理程序。如果该条目包含参数，那么传递它们过去，否则，传递该事件对象。
              func.apply(this, handler.parameters || [event]);
          }
      }
      return this;
   };
//注册一个事件。构造一条处理程序条目，并将它插入到处理程序数组中，如果不存在该类型的事件，则构造一个。
    that.on = function(type, method, parameters){
        var handler = {
            method : method,
            parameters : parameters
        };
        if(registry.hasOwnProperty(type)){
            registry[type].push(handler);
        }else{
            registry[type] = [handler];
        }
        return this;
    };
    return that;
};
```
### 第6章 数组

数组：一段线性分配的内存，通过整数计算偏移并访问其中的元素。数组是一种性能出色的数据结构。不幸的，JS没有提供这样的结构。

JS数组：JS提供一种拥有一些类数组（array-like）特性的对象。它把数组的下标转变为字符串，用其作为属性。它明显比一个真正的数组慢，但使用起来更方便。属性的检索和更新的方式与对象一模一样，只不过多一个可以用整数作为属性名的特性。

* 数组字面量

数组字面量：数组字面量提供一种非常方便地创建新数组的表示法。JS数组允许混合类型。
```javascript
var empty = [];
//数组的第一个值将获得属性名'0'，第二个值将获得属性名'1'，依次类推:
var numbers = [
'zero', 'one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten'
];

empty[1]; //undefined
numbers[1]; //'one'

empty.length; //0
numbers.length; //10
```
* 长度

JS数组长度：JS数组的length没有上界，如果你用大于或等于当前length的数字作为下标来存储一个元素，那么lengt值会被增大以容纳新元素，不会发生数组越界错误。

注意：**length属性的值是这个数组最大整数属性名加上1，不一定等于数组里属性的个数!**
```javascript
var myArray = [];
myArray.length; //0
myArray[10000] = true;
myArray.length; //10001
```
[]后置下标运算符：[]后置下表运算符把它所含的表达式转换成一个字符串，如果该表达式有toString方法，就使用方法的值。这个字符串将被用作属性名。如果这个字符串看起来像一个大于等于这个数组当前length且小于2^32-1的正整数，那么这个数组的length将会被重新设置为新的下标加1。可以直接设置length的值，设置更大的length不会给数组分配更多的空间，但把length设小将导致所有下标大于等于length的属性被删除。

* 删除

删除：由于JS数组是对象，所以delete运算符可以用来从数组中移除元素：
```javascript
//删除之后数组会留下一个空洞，除非移动每个数组的元素
delete numbers[2];//numbers 是['zero', 'one', undefined, 'three'...]
```
splice方法：如果即想删除元素又不想留下空洞，可以使用splice方法。由于需要移除和重新插入，splice对于大型数组来说可能效率不高。
```javascript
//第1个参数是数组中的一个序号，第2个参数是要删除的元素个数。
//任何额外的参数都会在序号那个点的位置被插入到数组中。
numbers.splice(2, 1);//numbers 是['zero', 'one', 'three'...]
```
* 枚举
 
for in：由于JS数组其实就是对象，所以可以用for in语句来遍历一个数组的所有属性。遗憾的是，for in无法保证属性的顺序，而大多数要遍历数组的场合都期望按照阿拉伯数字顺序产生元素。此外，可能从原型中得到意外属性的问题依旧存在。

for循环：常规的for语句可以避免for in语句的问题。
```javascript
var i;
for(i=0; i<arrays.length; i+=1){
    document.writeln(arrays[i]);
}
```
* 容易混淆的地方

对象与数组的选择：当属性名是小而连续的整数时，你应该使用数组。否则，使用对象。JS本身对数组和对象的区别是混乱的。typeof运算符报告数组的类型是'object'，这没有任何意义。

如何区别对象和数组：is_array（JS没有提供相应机制）
```javascript
//此方法不能识别从不同的窗口(window)或帧(frame)里构造的数组。
var is_array = function(value){
    return value && typeof value === 'object' && value.constructor === Array;
};
//better
var is_array = function(value){
    return Object.prototype.toString.apply(value) === '[object Array]';
};
```
* 方法

Array.prototype：可以通过Array.prototype给数组扩充方法。

* 指定初始值

JS数组通常不会预置值。如果你用[]得到一个新数组，它将是空的。如果你访问一个不存在的元素，得到的值则是undefined。JS应该提供为数组指定初始值的方法，但我们可以弥补这个疏忽：
```javascript
Array.dim = function(dimension, initial){
    var a = [], i;
    for(i=0; i<dimension; i++){
      a[i] = initial;
    }
    return a;
};
//创建一个包含10个0的数组
var myArray = Array.dim(10, 0);
```
多维数组：JS没有多为数组，但就像大多数类C语言一样，它支持元素为数组的数组：
```javascript
var matrix = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8]
];
matrix[2][1]; //7
//注意：Array.dim(n, [])在这里不能工作，如果使用它，每个元素都指向同一个数组的引用，后果不堪设想。
```
初始化多维数组：一个空的矩阵每个单元都会拥有一个初始值undefined。
```javascript
Array.matrix = function(m, n, initial){
    var a, i, j, mat=[];//由于JS变量作用域问题，将变量在函数体最前面声明。
    for(i=0; i<m; i++){
      a = [];
      for(j=0; j<n; j++){
        a[j] = initial;
      }
      mat[i] = a;
    }
    return mat;
};
//构造一个用0填充的4X4矩阵
var myMatrix = Array.matrix(4, 4, 0);
document.writeln(myMatrix[3][3]); //0
```
### 第7章 正则表达式

正则表达式：一门简单语言的语法规范，它应用在一些方法中，对字符串中的信息实现查找，替换和提取操作。JS中，正则表达式相较于等效的字符串处理有着显著的性能优势。JS正则表达式不支持注释和空白。

* 一个例子

匹配URL：parse_url
```javascript
var parse_url = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/;
var url = "http://www.ora.com:80/goodparts?q#fragement";
var result = parse_url.exec(url);
var names = ['url', 'scheme', 'slash', 'host', 'port', 'path', 'query', 'hash'];
var blanks = '    ';
var i;
for(i=0; i<names.length; i++){
    document.writeln(names[i] + ':' + blanks.substring(names[i].length), result[i]);
}
//输出结果
url: http://www.ora.com:80/goodparts?q#fragement
scheme: http
slash: //
host: www.ora.com
port: 80
path: goodsparts
query: q
hash: fragment
```

* 结构

创建RegExp对象的两个方法：正则表达式字面量（优先考虑）和使用RegExp构造器。

正则表达式字面量：正则表达式字面量被包围在一对斜杠中，这多少有点令人迷惑，因为斜杠也被用作除法运算符和注释符。

正则表达式标识：

| 标识       | 含义  |
| --------| -----:  |
|g     |全局的(匹配多次;不同的方法对g标识的处理各不相同)|
|i     |大小写不敏感（忽略字符大小写）|
|m     |多行（^和$能匹配行结束符）|

RegExp构造器：使用RegExp构造器创建正则表达式时需要注意双写反斜杠以及对引号进行转义。RegExp构造器适用于必须在运行时动态生成正则的情形。
```javascript
//创建一个匹配JS字符串的正则
var my_regexp = new RegExp("\"(?:\\\\.|[^\\\\\\\"])*\"",'g');
```
RegExp对象的属性：

| 属性       | 用法  |
| --------| -----:  |
|global    |如果标识g被使用，值为true|
|ignoreCase|如果标识i被使用，值为true|
|lastIndex |下一次exec匹配开始的索引，初始值为0|
|multiline |如果标识m被使用，值为true|
|source    |正则表达式源码文本|

正则对象共享问题：
```javascript
function make_a_matcher(){
    return /a/gi;
}
var x = marke_a_matcher();
var y = make_a_matcher();
//当心！x和y是相同的对象！
x.lastIndex = 10;
document.writeln(y.lastIndex); // 10
```
* 元素

正则表达式分支：一个正则表达式分支包含一个或多个正则表达式序列。这些序列被|(竖线)字符分割。如果这些序列中的任何一项符合匹配条件，那这个选择就被匹配。它尝试按顺序依次匹配这些序列项。
```javascript
//匹配in，不会匹配int，因为in已被成功匹配
"into".match(/in|int/);
```
正则表达式序列：一个正则表达式序列包含一个或多个正则表达式因子。每个因子能选择是否跟随一个量词，这个量词决定着这个因子被允许出现的次数。如果没有指定这个量词，那么该因子只会被匹配一次。

正则表达式因子：一个正则表达式因子可以是一个字符，一个由圆括号包围的组，一个字符类，或一个转义序列。除了控制字符和特殊字符以外，所有的字符都会被按照字面处理：\ / [ ] { } ? + * | . ^ $ (如果希望这些字符按字面去匹配，需要用\前缀对其进行转义)。一个未被转义的.会匹配除行结束符以外的任何字符。当lastIndex属性值为0时，一个未转义的^会匹配文本的开始。当指定了m标识时，它也能匹配行结束符。一个未转义的$将匹配文本的结束。当指定了m标识时，它也能匹配行结束符。

正则表达式转义：

\f:换页符  \n:换行符 \r:回车符 \t:制表符(tab) \u:unicode字符表示的十六进制常量。

\d:匹配一个数字，[0-9]，\D与其相反[^0-9]。 

\s:等同于[\f\n\r\t\u000B\0020\u00A0\u2028\u2029]，这是unicode空白符的一个不完全子集,\S则表示与其相反的：[^\f\n\r\t\u000B\0020\u00A0\u2028\u2029]。

\w：等同于[0-9A-Z_a-z]，\W与其相反：[^0-9A-Z_a-z]。\W本意是希望表示出现在话语中的字符。遗憾的是，它所定义的类实际上对任何真正的语言来说都不起作用。如果你需要匹配信件一类的文本，你必须指定自己的类。

\b：字边界标识，它能方便的对文本的字边界进行匹配。遗憾的是，它使用\w去寻找字边界，所以它对多语言应用来说是完全无用的。这并不是一个好的特性。

\1：指向分组1所捕获到的文本的一个引用，所以它能被再次匹配。
```javascript
//用于搜索文本中重复的单词,该单词的后面跟着一个或多个空白，然后再跟着与它相同的单词。
var doubled_words = /([A-Za-z\u00C0-\u1FFF\u2800-\uFFFD]+)\s+\1/gi;
```
分组：
  1. 捕获型：一个捕获型分组是一个被包围在圆括号中的正则表达式分支。任何匹配这个分组的字符都会被捕获。每个捕获型分组都被指定了一个数字。在正则表达式中第一个捕获（的是分组1,第二个捕获（的是分组2。
  2. 非捕获型：非捕获型分组有一个(?:前缀。非捕获型分组仅做简单的匹配，并不会捕获所匹配的文本。这会带来微弱的性能优势。非捕获型分组不会干扰捕获型分组的编号。
  3. 向前正向匹配（positive lookahead）：向前正向匹配分组有一个(?=前缀。类似于非捕获型分组，但在这个组匹配后，文本会倒回到它开始的地方，实际上并不匹配任何东西。这不是一个好的特性。
  4. 向前负向匹配（negative lookahead）：向前负向匹配分组有一个(?!前缀。类似于向前正向匹配，但只有当它匹配失败时它才继续向前进行匹配。这不是一个好的特性。

正则表达式字符集（RegExp Class）：正则表达式字符集是一种指定一组字符的便利方式。例如，如果想匹配一个元音字母，可以写做(?:a|e|i|o|u)，但更方便方法应该是写成一个类[aeiou]。类提供另外两个便利。第1个是能够指定字符范围，比如[a-z]。另一个方便之处是类的求反。如果[后的第一个字符是^，那么这个类会排除这些特殊字符，比如[^a-z]会匹配任何一个非小写字母的字符。

正则表达式字符转义（RegExp Class Escape）：在字符类中需要转义的特殊字符- / [ \ ] ^

正则表达式量词：正则表达式因子可以用一个正则表达式量词后缀来决定这个因子应该被匹配的次数。包围在一个花括号中的一个数字表示这个因子应该被匹配的次数。？等同于{0,1}，*等同于{0,}，+等同于{1,}。如果只有一个量词，表示趋向于进行贪婪性匹配，即匹配尽可能多的副本直至达到上限。如果这个量词附加一个后缀?，则表示趋向于进行非贪婪匹配，即只匹配必要的副本。一般情况下最好坚持使用贪婪匹配。

### 第8章 方法
* Array
  
array.concat(item...) : concat方法产生一个新数组，它包含一份array的浅复制（shallow copy）并把一个或多个参数item附加在其后。如果参数item是一个数组，那么它的每个元素都会被分别添加。功能和array.push(item...)类似。
```javascript
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z'];
var c = a.concat(b, true);// c=['a', 'b', 'c', 'x', 'y', 'z', true];
```
array.join(separator)：join方法把一个array中的每个元素构造成一个字符串，并用separator分隔符把它们连接在一起。默认的separator是逗号','。可以使用空白字符串作为separator实现无间隔的连接。对于连接大量的字符串片段，将它们放到一个数组中并用join方法连接通常比+元素运算符连接效率高。（注：**现在多数浏览器对+运算符连接字符串做了特别优化，性能已显著高于Array.join()，多数情况下，建议连接字符串首选+运算符**）。
```javascript
var a = ['a', 'b', 'c'];
a.push('d');
var c = a.join(' '); //c = 'abcd';
```
array.pop()：pop方法移除array中的最后一个元素并返回该元素。如果array是empty，它会返回undefined。
```javascript
var a = ['a', 'b', 'c'];
var c = a.pop(); //c = 'c', a = ['a', 'b'];
//pop的一种实现方式
Array.method('pop',function(){
    return this.splice(this.length - 1, 1)[0];
});
```
array.push(item...)：push方法把一个或多个参数item附加到一个数组的尾部。和concat方法不同的是，它会修改array，如果参数item是一个数组，它会把参数数组作为单个元素整个添加到数组中，并返回这个array的新长度值。
```javascript
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z'];
var c = a.push(b, true); // a=['a', 'b', 'c', ['x', 'y', 'z'], true] , c=5。
//push可以这样实现
Array.method('push',function(){
    this.splice.apply(this, 
        [this.length, 0].concat(Array.prototype.splice.apply(arguments)));
    return this.length;
});
```
array.reverse():reverse方法反转array里的元素的顺序，并返回array本身。
```javascript
var a = ['a', 'b', 'c'];
var b = a.reverse();//a=b=['c', 'b', 'a']
```
array.shift():shift方法移除数组array中的第1个元素并返回该元素。如果这个数组array是空的，则返回undefined。shift通常比pop慢得多：
```javascript
var a = ['a', 'b', 'c'];
var c = a.shift(); //a=['b', 'c'], c='a'
//shift可以这样实现
Array.method('shift', function(){
  return this.splice(0, 1)[0];
});
```
array.slice(star, end):slice方法对array中的一段做浅复制。end参数可选，默认是array.length。如果两个参数中的任何一个是负数，array.length会和它们相加，试图让它们变成非负数。如果start大于等于array.length，得到的结果是一个新的空数组。
```javascript
var a = ['a', 'b', 'c'];
var b = a.slice(0, 1); //b=['a']
var c = a.slice(1); //c=['b', 'c']
var d = a.slice(1, 2); //d=['b']
```
array.sort(comparefn):sort方法对array中的内容进行排序。它不能正确地给一组数字排序：
```javascript
//JS的默认比较函数把要被排序的元素都视为字符串。
var n = [4, 8, 15, 16, 23, 42];
n.sort(); //n=[15, 16, 23, 4, 42, 8] 
//自定义比较函数：相等返回0, 第1个参数排列在前面则返回一个负数，否则返回一个正数
n.sort(function(a, b){
  return a-b;
}); // n=[4, 8, 15, 16, 23, 42];
//如果想使排序适用范围更广，需要判断元素类型。
```
array.splice(start, deleteCount, item...):splice方法从array中移除一个或多个元素，并用新的item替换它们。
```javascript
//splice主要用于从一个数组中删除元素
var a = ['a', 'b', 'c'];
var r = a.splice(1, 1, 'ache', 'bug');
//a = ['a', 'ache', 'bug', 'c'], r = ['b']
```
array.unshift(item...):unshift方法像push方法一样，用于把元素添加到数组中，但它是把item插入到array的开始部分而不是尾部。它返回array的新length。
```javascript
var a = ['a', 'b', 'c'];
var r = a.unshift('?', '@');
//a=['?', '@', 'a', 'b', 'c'] , r=5
```
* Function

function.apply(thisArg, argArray) : apply方法调用function，传递一个会被绑定到this上的对象和一个可选的数组作为参数。apply方法被用在apply调用模式中。
```javascript
//bnd:返回一个函数，调用这个函数就像调用那个对象的一个方法
Function.method('bind', function(that){
  var method = this,
      slice = Array.prototype.slice,
      args = slice.apply(arguments, [1]);
  return function(){
      return method.apply(that, args.concat(slice.apply(arguments, [0])));
  };
});
var x = function(){
  return this.value;
}.bind({value: 666});
alert(x()); //666
```
* Number

number.toExponential(fractionDigits) : 把number转换成一个指数形式的字符串。可选参数fractionDigits控制其小数点后的数字位数，值必须在0~20。

number.toFixed(fractionDigits) : 把number转换为一个十进制数形式的字符串。可选参数fractionDigits控制其小数点后的数字位数，值必须在0～20,默认为0。

number.toPrecision(precision) : 把number转换为一个十进制数形式的字符串。可选参数precision控制数字的精度，值必须在0～21。

number.toString(radix) : 把number转换为一个字符串。可选参数radix控制基数，值必须在2～36，默认为10。通常情况下，number.toString()可以简单地写为String(number);

* Object

object.hasOwnProperty(name) : 如果这个object包含一个名为name的属性，那么hasOwnProperty方法返回true。原型链中的同名属性不会被检查，当name为"hasOwnProperty"时不起作用，返回false。

* RegExp

regexp.exec(string)：如果成功匹配regexp和字符串string，则返回一个数组。数组中下标为0的元素将包含正则表达式regexp匹配的子字符串。下标为1的元素是分组1捕获的文本，下标为2的元素是分组2捕获的文本，依次类推。如果匹配失败，则返回null。

exec循环调用：如果regexp带有一个g标识（全局标识），那么查找将从regexp.lastIndex（初始值为0）位置开始。如果匹配成功，那regexp.lastIndex将被设置为该匹配后第一个字符的位置。不成功的匹配将设置regexp.lastIndex为0。这允许你通过循环exec去查询一个匹配模式在一个字符串中发生了几次。需要注意的是，如果你提前退出了循环，再次进入这个循环前必须把regexp.lastIndex重置为0。而且，^因子仅匹配regexp.lastIndex为0的情况。

regexp.test(string) : 如果regexp匹配string，则返回true; 否则，返回false。不要对这个方法使用g标识。

* String

参考JS String API! 使用类似Java String。

### 第9章 代码风格

* [Google JavaScript 代码风格指南](http://bq69.com/blog/articles/script/868/google-javascript-style-guide.html)
* [Google JSON 风格指南](https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md)
* [CoffeeScript 编码风格指南](https://github.com/geekplux/coffeescript-style-guide)


### 第10章 优美的特性

* 函数是顶级对象
* 基于原型继承的动态对象
* 对象字面量和数组字面量

### 附录A 毒瘤

* 全局变量
  1. 在任何函数之外通过var声明的变量。
  2. 直接给全局对象添加一个属性，比如web浏览器的全局对象window。
  3. 直接使用未经声明的变量，这被称为隐式的全局变量：foo = value;

* 作用域
  
JS没有提供块级作用域：代码块中声明的变量在包含此代码块的函数的任何位置都是可见的。最好在每个函数的开头部分声明所有变量。

* 自动插入分号

JS自动修复机制：通过自动插入分号来修正有缺损的程序。但是，它可能会掩盖更为严重的错误。
```javascript
//如果return语句返回一个值，这个值表达式的开始部分必须和return位于同一行。
return  //自动插入分号会让它返回undefined
{
  status : true
};
//正确写法
return {
  status : true
};
```
* 保留字
* Unicode
* typeof
  
  typeof不能区分null和对象，typeof null 返回的是'object'。

* parseInt

parseInt是一个把字符串转换为整数的函数。它遇到非数字时会停止解析，parseInt("16")和parseInt("16 tons")产生的结果相同。如果该字符串第1个字符是0,那么该字符串会基于八进制而不是十进制来求值。在八进制中，8和9不是数字，所以parseInt("08")和parseInt("09")结果都是0。这个错误会导致程序解析日期和时间时出现问题。幸运的是，parseInt可以接受一个基数作为参数，parseInt("08",10)结果为8，建议加上基数参数。

* +

+运算符可以用于加法运算或字符串连接，具体如何执行取决于参数的类型。如果想使用+做加法运算，需确保两个运算数都是整数。

* 浮点数

二进制的浮点数不能正确地处理十进制的小数，因此0.1+0.2不等于0.3。不过，浮点数中的整数运算是精确的，小数可以通过指定精度来避免错误。常见的货币转换，可以先将元乘以100转换为分，然后用分进行计算，最后再除以100转换为元。

* NaN
NaN是IEEE754中定义的一个特殊的数量值，用于表示不是一个数字，尽管typeof NaN === 'number'返回true。

判断数字和NaN：typeof不能辨别NaN和数字，而且NaN也不等于它自己。如果需要区分数字和NaN，可以使用JS的isNaN函数。

判断数字：判断一个值是否可用作数字的最佳方法是使用isFinite函数，它会筛选掉NaN和Infinity。遗憾的是，isFinite会试图把运算数转换为一个数字，所以如果值事实上不是一个数字，它就不是一个好的测试。
```javascript
//自定义isNumber函数
var isNumber = function isNumber(value){
    return typeof value === 'number' && isFinite(value);
};
```
* 伪数组
* 假值
* hasOwnProperty

hasOwnProperty是方法而不是运算符，这就有hasOwnProperty被其他函数甚至一个非函数的值替换的危险!

* 对象

### 附录B 糟粕

* ==
* with语句
* eval
* continue语句
* swith穿越
* 缺少块的语句
* ++ --
* 位运算符
* function语句对比function表达式
* 类型的包装对象
* new
* void
