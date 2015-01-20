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
var that = this; //var和this之间的空格不能移除，其他的空格都可以移除
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
* 柯里化
* 记忆

### 第5章 继承
* 伪类
* 对象说明符
* 原型
* 函数化
* 部件

### 第6章 数组
* 数组字面量
* 长度
* 删除
* 枚举
* 容易混淆的地方
* 方法
* 指定初始值

### 第7章 正则表达式
* 一个例子
* 结构
* 元素

### 第8章 方法
* Array
* Function
* Number
* Object
* RegExp
* String

### 第9章 代码风格

### 第10章 优美的特性

### 附录A 毒瘤

### 附录B 糟粕





