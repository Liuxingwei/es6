# 第二章 块作用域与闭包

[“闭包是函数和声明该函数的词法环境的组合。”](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

这是MDN上对闭包的定义。

《JavaScript高级程序设计》中则是这样定义的：闭包是指有权访问另一个函数作用域中的变量的函数。

个人更倾向于MDN的闭包定义，原因有三：

其一，如果仅将闭包定义为可访问其父作用域（链）的局部变量的**函数**，那么就忽视了它持有外部环境（使外部作用域不被销毁）的意义。

其二，闭包有权访问的必然是其父作用域（链）中的局部变量，“另一个函数作用域”的说法不够明确清晰。

其三，就是本篇博文的主题了，闭包在ES6中，是不限于访问另一个**函数**的作用域的，还可以是块作用域。当然，《JavaScript高级程序设计》这本书出版时，还没有ES6，书里也明确说明JavaScript是没有块作用域的，因此这一点不能成为批评《JavaScript高级程序设计》的理由。

定义通常讲究严谨、言简意赅，也就意味着不太好理解。

换个通俗点的说法，闭包就是指在一个非全局作用域中声明的函数及其所在的这个作用域，这个函数在该作用域外被调用时，仍然能够访问到该作用域内的（局部）变量。如果不在声明函数的作用域外调用，或者该函数没有访问外部作用域的局部变量，闭包也就没有什么存在的意义了。

由于ES6出现以前，没有块作用域，这个**非全局作用域**就只能是一个**函数**了，那么闭包就是声明在另一个函数内部的函数（及其所在的函数）了。为了实现在声明它的作用域外也能调用该函数，就需要将该函数作为一个返回值，返回到父作用域（父级函数）之外了。

举例说明（例1）：

```
var age = 30;
var fn;
fn = (function () {
    var age = 20;
    var name = "Tom";
    return function () {
        console.log("name is " + name + ".");
        console.log("age is " + age + ".");
    };
})();
fn();
```

运行结果：

```
name is Tom.
age is 20.
```

可以看到，age 获取的是匿名函数中声明的局部变量 age 的值 20，不是全局变量 age 的值 30。name 更是干脆没有同名全局变量，只有匿名函数中声明的局部变量。

对于ES6，因为块作用域的存在，闭包就有了另一种实现，举例如下（例2） 

```
let age = 30;
let fn;
{
    let age = 20;
    let name = "Tom";
    fn = function () {
        console.log("name is " + name + ".");
        console.log("age is " + age + ".");
    };
}
fn();
```

运行结果与例1相同：

```
name is Tom.
age is 20.
```

可见，在ES6中，声明在块作用域内的函数，在离开块作用域后，优先访问的依然是声明它的块作用域的局部变量。

在《你不知道的JavaScript》中文版下卷中，曾经提到过**块作用域函数**，即声明在块作用域内的函数，在块外无法调用。

原文的例子如下（例3）：

```
{
    foo();
    function foo() {
        //...
    }
}
foo();
```

书中认为，第一个foo\(\)调用会正常返回结果，第二个foo\(\)调用会报 ReferenceError 错误。经在 chrome（64.0） 和 firefox（58.0）版中测试，非严格模式下，两个调用均正常返回结果，不会出现 ReferenceError 错误。仅在严格模式下，与其预期结果相同。

也就是说，非严格模式下，声明在块作用域内的函数，是在块作用域外的父作用域中有效的。

这也导致了例2还有一个变体（例4）：

```
let age = 30;
{
    let age = 20;
    let name = "Tom";
    function fn() {
        console.log("name is " + name + ".");
        console.log("age is "+ age + ".");
    }
}
fn();
```

其结果也是：

```
name is Tom.
age is 20.
```

究其原因，在于函数是在块作用域内声明的，因此它在被调用时，会优先访问块作用域内的局部变量。又因为它虽然是在块内声明，却被提升至其父作用域，所以可以在块作用域外被访问。

不过这种写法，意图不够清晰，且在多层作用域的情况下，容易产生混乱，严格模式下，还会导致错误。



现在再来看上一篇博文中的循环变量的例子（例17、例18和例19）：

（例17）

```
var i;
var fn = [];
for (i = 0; i < 3; i++) {
    fn.push(function () {
        console.log(i);
    });
}
fn[0]();
fn[1]();
fn[2]();
```

之所以会输出三个3，是因为函数在调用时才会尝试获取i值，而不是在定义时就获取了i的值，而调用是在循环之后发生的。调用时因为i是全局变量，其值已经在循环中自增到了3。因此3次调用均返回3。

（例19）

```
var i;
var fn  = [];
for (i = 0; i < 3; i++) {
    fn.push((function (i) {
       returnfunction () {
            console.log(i);
        }
    })(i));
}
fn[0]();
fn[1]();
fn[2]();
```

实际是个障眼法，循环内部的函数定义中，形参使用了和全局变量 i 同名的变量，由于子作用域同名变量的遮蔽作用，函数内部的 i 实际已经不是全局变量 i 了，而是一个匿名函数内部的局部变量。调用匿名函数时，将全局变量 i 的值传递给了局部变量 i 。而返回的那个闭包函数，按照闭包的定义，无论在何处调用，都只会先访问其父作用域中的局部变量。

如果把匿名函数中的 i 换个名字，就更能清晰地看出闭包在这里的作用了：

```
var i;
var fn  = [];
for (i = 0; i < 3; i++) {
    fn.push((function (k) {
        returnfunction () {
            console.log(k);
        }
    })(i));
}
fn[0]();
fn[1]();
fn[2]();
```

而（例18）：

```
var fn = [];
for (let i = 0; i < 3; i++) {
    fn.push(function () {
        console.log(i);
    });
}
fn[0]();
fn[1]();
fn[2]();
```

就刚好是本篇博文所说的块作用域闭包。每个循环都会产生一个块作用域；而 for 语句中的 let，会在每个循环产生的块作用域内生成一个局部变量 i；声明在每个循环内的匿名函数，都会优先访问声明自己的那个循环产生的块作用域中的 i 的值。

其实际意义与如下例子是一样的：

```
var fn = [];
for (let i = 0; i < 3; i++) {
    let k = i;
    fn.push(function () {
        console.log(k);
    });
}
fn[0]();
fn[1]();
fn[2]();
```

比较而言，用函数作为外部作用域的闭包，可以用返回闭包函数的方式将闭包函数传递到闭包作用域外。而块作用域闭包没办法使用return，就只能是直接为外部作为域的变量赋值的方式，将闭包函数传递出去。

不过，对于例19，可以改造成不使用返回值，直接在闭包函数内使用外部作用域变量的形式：

```
var i;
var fn  = [];
for (i = 0; i < 3; i++) {
    (function (k) {
        fn.push(function () {
            console.log(k);
        });
    })(i));
}
fn[0]();
fn[1]();
fn[2]();
```

由于这种匿名函数立即调用的方式构造的闭包只执行一次，要将闭包函数传递给哪个变量，也是coding时能够确定的，返回值传递，还是直接使用外部变量，都是一样的。而这种形式，在ES6中都可以用块作用域闭包代替。

就代码本身的理解难度而言，ES6的块级作用域更容易一些。

回到本文开头的闭包定义，广义的解读，由于任何一个函数必然有声明它的记法环境，所以所有的函数和声明它的记法环境都构成闭包。比如全局作用域内的函数，它和全局作用域就构成了闭包。这也是《ES6标准入门》（阮一峰）在「let 和 const」一章中解释例17时，会说fn\[\*\]\(\)的调用是通过闭包获取的全局变量 i 的原因吧。

PS：

顺便说一下**块作用域函数**，《你不知道的JavaScript》中，关于块作用域函数有两个示例，其一见上文例3。

另一个例子如下（例4）：

```
if (something) {
    function foo() {
        console.log("1");
    }
} else {
    function foo() {
        console.log("2");
    }
}
foo();
```

原文说，在前ES6环境中（应该相当于非严格模式），不管something的值是什么，foo\(\)都会打印出“2”，因为两个函数声明都被提升到了块外，第二个总会胜出。

经在chrome（64.0） 和 firefox（58.0）版中测试，实际运行结果是：something为真，foo\(\)打印“1”，something为假，foo\(\)打印“2”。

严格模式下，则与其预期相符，抛出一个ReferenceError。

其实这种全局定义函数，在ES6中，与函数变量方式相比，不能算是最佳实践了。

