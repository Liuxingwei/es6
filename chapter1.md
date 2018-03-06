# 第一章 块作用域与 let 和 const

在学习ES6的块作用域和 let、const 之前，我们先来看看ES5以前的 var 关键字。

var 关键字用于定义一个变量，通常我们会将其与变量的赋值合并为一条语句，就像下面这样（例1）：

```
var age = 30;
```

但实际情况是有些微妙的。

在JavaScript中，变量的定义与否，虽然不像强类型语言那样重要，但也还是有所不同的。

变量未定义，是一种未捕获类型的错误，输出的结果是变量未定义，同时终止后续脚本的执行，示例如下（例2）：

```
console.log(age);
```

输出：

```
age is not defined.
```

已经定义但未赋值的变量，输出的结果是undefined（这个undefined不是未定义，是指变量的值为undefined，这是JavaSript中的一个特殊的值）。示例如下（例3）：

```
var age;
console.log(age);
```

输出：

```
undefined
```

var 定义变量语句，在JavaScript中会被编译器提升到整个作用域内有效，无论它写在脚本的哪个位置。示例如下（例4）：

```
console.log(age);
var age;
```

输出

```
undefined
```

这个待遇只有 var 定义语句都有，赋值语句是不会被提升的。当我们在代码中写下一条定义的同时赋值的 var 语句时，编译器会将其拆分为定义语句和赋值语句，并将定义语句提升，而赋值语句仍然只对其后的代码有效。

请看下面的例子（例5）：

```
console.log(age);
var age = 30;
console.log(age);
```

它与如下例子的输出是一致的（例6）：

```
var age;
console.log(age);
age = 30;
console.log(age);
```

输出结果：

```
undefined
30
```

我们还知道，JavaScript中变量可以不定义就使用，比如（例7）：

```
age = 30;
console.log(age);
```

输出：

```
30
```

这种直接赋值的语句，与定义的同时赋值，还是有所不同。参见下面的例子（例8）

```
console.log(age);
age = 30;
console.log(age);
```

输出结果：

```
ReferenceError: age is not defined
```

之所以一例5有这么大的区别，在于这里只有变量赋值，没有 var 定义部分。编译器不会自动补充一个 var 定义并提升（或者说编译器补充了一个隐含的 var 定义，但是这个隐含的 var 定义不会被提升），因此第一个console.log语句会直接报出变量未定义的错误，之后的变量赋值和输出语句不被执行。

为避免误用未定义的变量，在使用变量前，要养成在使用变量前用 var 进行定义的好习惯:\)。除非在函数作用域内要使用父作用域的变量，这也是闭包存在的根源。

如果可能，可以使用严格模式，即在脚本作用域的第一行，写下 "use strict"（注意要带着又引号哦）。这样做的一个好处是，WebStorm 这样的 IDE 会在为未定义的变量赋值时，直接给出警告。

### 块作用域

块作用域是指用一对花括号括住的语句块使用域。在ES5以前，是没有块作用域的，它只有函数作用域，因此 var 在非函数作用域的括号内，与在括号外，没有区别。为了保持向下兼容，在ES6中，var 在块作用域中仍然保持其原有性状。如下例（例9）：

```
console.log(age);
{
    var age = 30;
}
console.log(age);
```

未定义就赋值的变量，也与原来的表现一致（例10）

```
{
    age = 30;
}
console.log(age);
```

要使块作用域别有不一样的意义，就需要 let 和 const，这两个关键字了。

 let 关键字与 var 的作用类似，都是定义变量，也都可以在定义变量的同时，为变量赋值。不同之处有以下两点：

第一，let 可以定义（仅在）块作用域内（有效）的变量。如下例所示（例11）：

```
{
   let age = 30;
}
console.log(age);
```

输出结果为：

```
ReferenceError: age is not defined
```

对照例9，可以看出区别所在。

第二，let 定义变量语句，不会被提升到整个作用域有效。见下例（例12）：

```
console.log(age);
let age = 30;
```

输出结果为：

```
ReferenceError: age is not defined
```

对照例4，可以看出区别所在。

还有一个细节，let 和 var 都有在子作用域内定义与父作用域同名变量时，屏蔽父作用域同名变量的作用。

见如下两例：

（例13）

```
var age = 20;
(function () {
    var age = 30;
    console.log(age);
})();
console.log(age);
```

（例14）

```
var age = 20;
{
    let age = 30;
    console.log(age);
}
console.log(age);
```

输出结果均为：

```
30
20
```

区别在于，如果在子作用域中，定义同名变量之前就使用该变量，对于 var 而言，由于其会被提升，因此得到的是undefined。对于let，不会被提升，却也不会使用父作用域的变量，而是报出引用错误。示例如下：

（例15）

```
var age = 20;
(function () {
    console.log(age);
    var age = 30;
})();
```

输出：

```
undefined
```

（例16）

```
var age = 20;
{
    console.log(age);
    let age = 30;
}
```

输出：

```
ReferenceError: age is not defined
```

 还要说明一点，在ES6中，所有具有语句块含义花括号括住的部分都构成块作用域，包括单纯的语句块、if语句块、while语句块、for语句块、break语句块和函数语句块。

在 for 的循环变量定义，会在每次迭代中执行一遍，就相当于每次迭代定义一个变量，迭代结束销毁，循环多少次，生成多少个不同的循环变量。 这个特点的实际意义，可以用下面的两个示例说明。

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

可能会预期其输出结果为：

```
0
1
2
```

实际输出结果为：

```
3
3
3
```

（例18）

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

输出结果为：

```
0
1
2
```

在没有块作用域和let关键字之前，要实现同样效果，需要利用闭包（例19）

```
var i;
var fn  = [];
for (i = 0; i < 3; i++) {
    fn.push((function (i) {
        return function () {
            console.log(i);
        }
    })(i));
}
fn[0]();
fn[1]();
fn[2]();
```

附带说一句：for 语句块的循环变量赋值部分的let定义，作用域很特殊，是循环体作用域的父作用域。

例20：

```
for (let i = 0; i < 3; i++) {
    let i = "s";
    console.log(i);
}
```

输出结果为：

```
s
s
s
```

输出三个 s ，说明循环变量 i 保持了从 0 到 3 的自增，而循环体内的 i 屏蔽了循环变量 i ，所以会输出 s。

由于 let 相较于 var 优势明显，又没有明显的缺点，在ES6兼容环境下，可以完全抛弃 var，而用 let 代替。

《你不知道的JavaScript》中，提倡C语言风格的 let 使用方法，即在作用域的起始处使用 let 定义作用域内的所有变量，这样可以避免变量未定义就使用的情况发生。

个人觉得还是按现代编程语言就近声明的惯例更合适，可以避免无意识的误用，而且在阅读时也不需要在代码间跳跃以寻找变量定义和赋值。不过如果能保持代码块的简短，又能坚持同一变量在同一作用域内不作两用的原则，其实放在哪儿都无关紧要。

对于 const，其实没有太多需要说明的，常量定义而已。

PS：

还有一个细节：变量在使用时，可能处于以下四种状态：未声明、已声明未初始化、已初始化、已赋值。这里的初始化，指的是默认赋值undefined。

var 和 let 语句，在声明变量的同时，如果没有明确给变量赋值，则被初始化为undefined，如下两例可以证明：

（例21）

```
var age;
console.log(age);
```

（例22）

```
let age;
console.log(age)
```

两例均输出：

```
undefined
```

前面提到 var 语句会被提升，指的就是初始化这个功能，let 的初始化功能不会被提升。

但是 var 和 let 的变量声明功能，则是在整个作用域内生效的。即不管 var 和 let 处于作用域内什么位置，所声明的变量在整个作用域内都是已声明状态。没有使用 var 和 let 声明直接赋值的变量，则没有此待遇，在其赋值前，变量未声明。

下面举例说明：

未声明（例23）

```
console.log(age);
age = 30;
console.log(age);
```

输出结果为：

```
ReferenceError: age is not defined
```

已声明未初始化（例24）：

```
console.log(age);
let age = 30;
console.log(age);
```

firefox 输出：

    ReferenceError: can't access lexical declaration `a' before initialization

chrome 的输出比较含混：

```
ReferenceError: age is not defined
```

已初始化（例25）

```
console.log(age);
var age = 30;
console.log(age);
```

输出：

```
undefined

30
```

对于已声明未初始化和未声明，可以用 typeof 运算符来做区分，未声明的变量 typeof 的结果为 undefined，而对于已声明未初始化的变量，typeof 则会抛出 ReferenceError。

未声明（例26）

```
console.log(typeof age);
age = 30;
console.log(typeof age);
```

输出结果为：

```
undefined
number
```

已声明未初始化（例27）

```
console.log(typeof age);
let age = 30;
console.log(typeof age);
```

chrome 输出：

```
ReferenceError: age is not defined
```

firefox 输出：

    ReferenceError: can't access lexical declaration `a' before initialization

对于未声明和已声明且已初始化为 undefined 的变量，typeof 就力不从心了，因为 typeof undefined 的结果也是 undefined，和未声明变量的 typeof 结果相同。见下例（例28）

```
console.log(typeof age);
age = 30;
console.log(typeof name);
var name;
```

输出：

```
undefined
undefined
```

还想区别，就用如下方式（例29）：

```
console.log(typeof age && undefined === age);
```

对于已声明且已初始化为 undefined 的变量，输出结果为 true。

否则，chrome 抛出 ReferenceError: age is not undefined，firefox 抛出 ReferenceError: can't access lexical declaration \`a' before initialization。

