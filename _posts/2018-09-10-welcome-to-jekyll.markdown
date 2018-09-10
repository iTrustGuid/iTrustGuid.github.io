# 前端基础（二）：变量声明的6种方法

*字数：2869*	

*阅读时间：10分钟*



最新的ECMAScript规范中，变量声明有var、function、let、const、import、class六种方法。

## var

> 语法：
>
> var  varname  [= value1 [, vaname1[,valname2 ...]]];

对应如下代码：

```js
var x;
var x=1;
var x=1,y=2;
var x=y,y=2;	//x->undefined y->2
```

我们需要注意该语法的两个特性：块级作用域和变量提升。

var语法声明的变量作用域为当前上下文（可以简化理解为当前变量被包裹的函数或顶级作用域），这里要特别注意，代码块并没有创建新的上下文，所以 var 没有块级作用域的概念。我们在一个循环体或者判断条件语句中使用的变量都会存储到当前函数对应的上下文中。例：

```js
function test () {
    console.log(i);
    for (var i = 0; i < 10; i++) {
        console.log(i);
    }
    console.log(i);
}
test();
console.log(i);

//输出结果：undefined 0 1 2 3 4 5 6 7 8 9 10 ReferenceError
```

在这个示例中，变量`i`是在循环体中声明的，但是我们在循环体外仍然可以访问到它，而在函数之外就会报引用错误。说明，`i`的作用域在整个test函数中。

我们再来看看，在test函数中，我们在变量声明之前就使用了该变量并且可以正常运行，这就是变量提升。js解析器在解析js代码时，会先遍历该函数中的所有变量声明，凡是使用var语句声明的变量，都会将他们的声明语句提升到函数顶端，即上述代码正真执行的是如下代码：

```js
function test () {
	var i;	//变量提升
    console.log(i);
    for (i = 0; i < 10; i++) {
        console.log(i);
    }
    console.log(i);
}
test();
console.log(i);
```

理解以上两点，就基本上理解了var的使用方式了。至于像`i=10`不使用var语句，直接声明一个变量这种写法，实质就是在全局对象上添加一个属性而已。例如，在浏览器中，其实等同于`window.i = 10;`。看到这样的代码我们明白什么意思就行了，我们自己就千万不要写这种魔法代码了。（在代码顶层通过var语法声明一个变量，实质也是在全局变量上添加一个属性）

## function

> 语法：
>
> function name([param [,param1,...]]){
>
> ​	[statements]
>
> }

函数声明的效果同 var 语句。需要注意下文 let 介绍中的函数声明。 

## let


>MDN对let取名的解释：
>
>Let是一个数学声明，是采用于早期的编程语言如Scheme和Basic。
>变量被认为是不适合更高层次抽象的低级实体，因此许多语言设计者希望引入类似但更强大的概念，
>如在Clojure、f#、Scala，let可能意味着一个值，或者一个变量可以赋值，但不能被更改，
>这反过来使编译器能够捕获更多的编程错误和优化代码更好。
>javascript从一开始就有var，所以他们只是需要另一个关键字，并只是借用了其他数十种语言，
>使用let已经作为一个传统的尽可能接近var的关键字，虽然在javascript 中 let只创建块范围局部变量而已。

>语法：
>
>let letname [ = letval[,letname1 = letval1[,...]]] ;

let语法与var语法使用方式完全一样，主要区别在于作用域和变量提升两个特性的不同。

首先，let语法不会进行变量提升，在声明前使用变量会报引用错误。然后，let是块级作用域，并非作用于整个封闭函数。举个栗子：

```js
    //变量提升
    console.log(valVal); //undefined
    var valVal = 'valVal';
    console.log(letVal); //ReferenceError
    let letVal = 'letVal';

    //作用域
    for (let i = 0; i < 10; i++) {
        console.log(i);
    }
    console.log(i); //ReferenceError
```

### 注意事项	

**①TDZ(temporal dead zone)临时死锁域**

只要在块级作用域中声明了let变量，该变量就和当前作用域绑定，不再受到外面作用域的影响。见如下示例：

```js
{
    let testval = 'val';
    {
        let testval = 'val1';
        console.log(testval); //val1
    }
    console.log(testval); //val
}

{
    var testval = 'val';
    {
        var testval = 'val1';
        console.log(testval); //val1
    }
    console.log(testval); //val1
}
```

如上所示，当js代码解析到一个代码块中时，会先检测代码块中的let语句，然后将检测到的变量绑定到当前作用域中，所有对该变量的操作都会作用于绑定作用域上的变量，不会受外侧作用域的影响。因此，虽然外侧作用域声明了testval变量，但代码解析到内层作用域会重新绑定内层的testval变量，对变量的赋值操作也仅仅作用于当前作用域对应的变量上。因此，内层打印的变量和外层打印的变量会不一致。上例的第二段代码演示了var语法的效果，它就没有临时死锁域的机制。

因此，我们在使用如下代码时需要注意了：

```js
{
    let testval = 'val';
    {
        console.log(testval); //ReferenceError
        let testval = 'val1';
    }

    function testFun (x = y, y = 1) {

    }
    testFun(); //y is not defined
}
```

**②重复声明**	

let语法不允许重复声明，测试代码如下：

```js
//重复声明
{
    var testval = 'val';
    var testval = 'val1';
    console.log(testval);   //val1
}

{
    let testval = 'val1';
    var testval = 'val';    //重复定义错误
    console.log(testval);
}

{
    var testval = 'val';    //重复定义错误
    let testval = 'val1';   
    console.log(testval);
}

{
    let testval = 'val';
    let testval = 'val1';   //重复定义错误
    console.log(testval);
}
```

var语法支持重复声明，但是一旦使用let语法，就不允许重复声明了。上例中，虽然，第二个代码块和第三个代码块中var语句的顺序不一致，但都是var语句报错，这也是上述TDZ机制导致的。

函数参数也是一样，不允许重复声明，示例如下：

```js
{
    function testFun(arg){
        let arg = 'arg';	//重复定义错误
    }
    testFun(arg);
}
```

**③switch语句**	

由于switch语句中，case语句并没有创建一个作用域块，所以也不能重复声明变量：

```js
//switch
{
    let caseVal = '1';
    switch (caseVal) {
    case '1':
        let val = '1';
        break;
    case '2':
        let val = '2';  //重复定义错误
        break;
    default:
    }
}
```

这里插播一下，判断时候产生新的块状作用域的最简单的方式就是是否有一对大括号包裹。如果有，就是一个新的块作用域。

这时，有人就会问了，那如果我写一个没有大括号的条件语句会如何呢？看如下示例：

```js
{
    let testVal = '';
    if(0) let testVal = '1';    //Lexical declaration cannot appear in a single-statement context
}
{
    let testVal = '';
    if(0) var testVal = '1';    //重复定义错误
}
{
    var testVal = '';
    if (0) var testVal = '1'; //正确运行
}
```

那串英文的意思是：词法声明不能出现在单个语句的上下文中。简单来说，就是在单行的条件语句中，不允许使用let表达式。

**④typeof**

因为TDZ的机制，导致之前绝对安全的 typeof 语句不再安全。举个栗子：

```js
console.log(typeof letval);	//报变量未定义错误
let letval = 1;
```

所以，我们在使用变量时，还是先申明再使用为好。

**⑤函数声明**	

ES5中规定，函数只能在顶层作用域或函数作用域中声明，不能在块级作用域中声明。但是为了兼容ES5之前的代码，浏览器并没有遵循这条规范，仍然支持在块级作用域中声明函数。

到了ES6中，明确允许在块级作用域中声明函数。并且，函数的声明默认是使用let语法的，即在块级作用域之外，我们是无法访问到该函数。举个栗子：

```js
{
    function testFun () {
    	console.log('testFun');
    }
}
testFun();
```

按照规范，理应报函数未定义错误。但实际运行结果为正确输出了 testFun 。

原来因为这条规则对之前的代码影响太大，所以ES6在附录B中规定，浏览器的实现可以不遵循这条规定，可以有自己的行为方式。遵循以下两条：

-允许在块级作用域中声明函数

-函数声明类似var语法，有变量提升机制。

建议：

由于这是过度的解决方案，最终还是会向规范靠拢，建议尽量不要在块状作用域中声明函数。如果必须要这么做，则使用 let 语句来声明函数。



### 常见用法

块级作用域比之前的函数作用域更为规范和便利，所以建议可以完全使用 let 替代 var。举个经典的栗子：

```js
//var用法
for (var i = 0; i < 10; i++) {
    (function (i) {
        setTimeout(function () {
            console.log(i);
        }, 100);
    }(i));
}

//let用法
for (let i = 0; i < 10; i++) {
    setTimeout(function () {
        console.log(i);
    }, 100);
}
```

之前我们想在一个循环语句中执行一个异步的函数，只能通过一个匿名函数来人为制造一个新的作用域。使用 let 语法，它每次循环都是一个新的作用域，实现起来就非常简单合理了。

再看一个栗子：

```js
//var
var strName = 'iTurst';
function testFun () {
    console.log(strName);   //undefined
    var strName = 'new name';
}
testFun();

//let
let strName = 'iTurst';
function testFun () {
    console.log(strName);   //变量未定义错误
    let strName = 'new name';
}
testFun();
```

这种情况，使用 var 语句的时候，不会报任何错误，但是执行结果却不是我们预期的结果，这种机制也非常不合理。在 let 语句中，我们就明确地知道代码问题在哪了。



## const

>语法：
>
>cons name1 = value1 [, name2 = value2 [...]];

const 语法特性与 let 完全一致，只是 const 必须在声明时初始化，而后无法继续修改其值。

需要注意，如果 const 语句初始化的值是一个对象，对象的属性我们还是可以做修改的。所以，在这种情况下我们可以将对象做特殊的处理。

## class

> 语法：
>
> class name [extends] {
>
> ​	//class body
>
> }

class语句特性：

-class 语法申明的变量其实就是一个函数，和ES5中创建一个类得到的结果基本一致（除了属性不可枚举）。

-不做变量提升

-不可重复定义

```js
class TestClass {
}
console.log(typeof (TestClass));    //function
```

## import

这个是ES6新引入的模块机制，可以在当前模块中导入其他模块导出的任何类型变量，可以是一个简单类型，也可以是一个对象。作用域范围是当前整个模块。

有关模块的内容，我后续会有单独的文章来详细讲述。

## 总结

①声明一个类，我们使用 class 最为合适。

②变量声明我们酌情使用 let 和 const

③函数声明使用 function 或者 `let funName = function()`。

④模块导入使用 import。



---

*参考资料：*

http://es6.ruanyifeng.com/#docs/let

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let



*欢迎关注我的微信公众号：*

![](https://gitee.com/lsjcoder/img/raw/master/other/1.jpg)