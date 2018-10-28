---

title: this的解析.md
date: 2018-10-27 18:09:28
tags:
---

最近在看《你不知道的JavaScript（上卷）》，觉得书中关于this的知识点讲解的比较透彻易懂，在此做一些整理分享出来。

this在JS中很重要，程序员平时写代码时，尤其是在ES6出现之前，基本上都离不开this，但往往我们在使用this的时候并没有深入理解this究竟指向的是什么，所以当逻辑复杂之后，很容易就造成this指向与自己预期不同而导致结果出错，这种情况在查bug时也比较费时费力，因此掌握this，了解this究竟指代了什么对于前端开发人员来说是有必要的，即使是在ES6已经普及了的今天。

主要介绍以下：

+ ### 为什么要使用this

+ ### this的认知误区

+ ### this的四种绑定方式及优先级



#### 为什么要使用this 

this一直都是JS种比较复杂不易理解的一个机制，往往对于简单的几行实例代码，我们可能很容易就能找到this指代的对象，但是当项目越来越大，代码越来越多时，多数人就会用猜测或者试错的方式来判断代码中的this究竟是指向什么。我们付出了大量时间来学习复杂的this，真的值得吗，它能够给我们带来什么优势和便利呢？

我们先来看一段代码：

``` javascript
function foo(context){
    return context.name.toUpperCase();
}
function bar(context){
    var greeting = "Hello, I'm"+ foo(context);
    console.log(greeting);
}
var me = {
    name :"jeff";
}
var you ={
    name :"zoe";
}
foo(you); // 
bar(me);  // Hello, I'm JEFF
```

随着使用的模式越来越复杂，显示传递长下文对象会让代码变得越来越混乱，使用this则不会这样。this 提供了一种更优雅的方式来隐式“传递”一个对象引用，可以方便我们将API设计的更加简便易于复用。

#### this的常见误区

往往新手程序员甚至一些经验丰富的老程序员都会对this的指向理解有误：

1. this指向自身

   很容易理解成this是指向函数自身的，这个从某些场景或者说大部分场景下是正确的，但是从真正的引用关系上来看是不对的，举个例子：

   ``` javascript
   function foo(num){
       console.log("foo:"+ num);
       this.count++;
   }
   foo.count = 0;
   var i;
   
   for (i=0;i<10;i++){
       if(i>5){
           foo(i);
       }
   }
   
   //foo被调用了多少次
   console.log(foo.count); // 0 
   ```

   如果按照this指向自身来理解的话，foo.count应该调用了4次才对，但是结果却是0，说明this.count并不指向foo.count。有些人此时就会避开this，声明一个新的变量来存储count属性。

   ```javascript
   function foo(num){
       console.log("foo:"+ num);
       data.count++;
   }
   var data={
       count:0
   }
   var i;
   
   for (i=0;i<10;i++){
       if(i>5){
           foo(i);
       }
   }
   
   //foo被调用了多少次
   console.log(foo.count); // 4
   ```

   这种做法虽然解决了问题，但是却避开了this的含义和工作原理，使用了我们比较熟悉的词法作用域技术。

2. this指向函数的作用域

   this在任何情况下都不指向函数的词法作用域，例子：

   ```javascript
   function foo(){
       var a = 2;
       this.bar();
   }
   function bar(){
       console.log(this.a);
   }
   foo(); // 在严格模式下会抛出错误 a is not defined; 非严格模式下，输出undefined
   ```

   显然，this指向的也不是函数的作用域。

3. this到底是什么

   排除这些错误理解后，我们来看看this到底是什么机制。

   this是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。



首先，在理解this的绑定过程之前，要先理解调用位置：调用位置就是函数在代码中被调用的位置，而非声明位置。通常来说寻找调用位置就是寻找函数被调用的位置，但是在逻辑较为复杂或者使用了某些设计模式的项目中，调用位置可能会被隐藏起来。

例子：

``` javascript
function baz(){
    //当前调用栈是：baz
    //因此，当前调用位置是全局作用域
    console.log("baz");
    bar();//bar的调用位置
}

function bar(){
    //当前调用栈是baz->bar
    //因此，当前调用位置在baz中
    
    console.log("bar");
    foo();//foo的调用位置
}

function foo(){
    //当前调用栈是baz -> bar->foo
    //因此，当前调用位置在bar中
    console.log("foo");
}
baz(); //baz的调用位置
```

可能这种寻找调用位置的方法较为繁琐，现在很多浏览器的开发者工具中都集成了定位调用位置的功能。比如chrome中，就可以在debugger处，使用开发者工具找到调用栈，找到栈中第二个元素，这就是调用位置。



#### this的四种绑定方式

我们看看this在执行过程中调用位置如何决定this的绑定对象。

**默认绑定**

结论：this指向全局对象(非严格模式)、this为undefined(严格模式下)

首先介绍的是最常用的调用类型：独立函数调用。



```javascript
function foo(){
    console.log(this.a);
}
var a = 2;
foo(); //2
```

此时的a其实是声明后的全局对象，此时调用foo函数的也是全局对象window。这就遵循了this绑定的默认规则，this指向全局对象。这是在非严格模式下，如果我们使用了"use ditrict"的严格模式此时执行foo()就会报错“this is undefined”。这说明在严格模式下，this默认绑定到undefined。

这里需要注意的是，如果在严格模式前使用this，而调用位置出现在严格模式之内，这种情况this还是会指向全局对象，但不建议在代码中出现两种模式，除非是引用的第三方库中使用了不同的模式。



**隐式绑定**

根据调用位置是否有上下文对象，或者是否被对象所拥有或包含。

```javascript
function foo(){
    console.log(this.a);
}
var obj = {
    a:2,
    foo:foo
}
obj.foo();
```

当函数引用有上下文对象时，它就遵循了隐式绑定的规则，this会隐式的将绑定在这个上下文对象上。所以此时obj.a就等于this.a。只有最接近函数的一层或者说最后一层上下文对象在调用中起作用。在隐式绑定时，我们必须在一个对象内部包含一个指向函数的属性，并通过这个属性简介引用函数，从而把this绑定到这个对象上。

例子：

```javascript
function foo(){
    console.log(this.a)
}
var obj2={
    a:3,
    foo:foo
}
var obj3={
    a:4,
    foo:foo
}
obj3.obj2.foo(); //3
```

在隐式绑定this的时候，有些情况下this会丢失，转而遵循默认规则绑定到全局对象或者undefined。

例子：

```javascript
function foo(){
    console.log(this.a)
}
var obj = {
    a:2,
    foo:foo
}
var bar = obj.foo;
var a = 333;
bar(); //333
```

此时bar只是obj.foo的引用，它引用的是foo的本身，因此bar()此时是一个不带任何修饰的函数调用，因此指向全局。

在我们经常使用的setTimeout函数中经常会犯的一个错误就是this的隐式绑定丢失。

```javascript
function setTimeout(fn,delay){
    //等待delay毫秒
    fn();//调用位置
}
var a = 123;
var obj={
    a:2,
    foo:foo
}
function foo(){
    console.log(this.a);
}
setTimeout(obj.foo,100);//123
```

当我们给该函数传递一个函数的引用作为参数时，在内部调用函数时，this的绑定就会丢失转而绑定到全局或者undefined。默认和隐式的绑定都无法主动的控制this的绑定对象，下面我们介绍两种可以主动绑定this的方式。



**显示绑定**

如果我们想在某个对象上强制调用函数，应该怎么实现。我们可以使用js中函数的原型属性call()和apply()。它们接收的第一个参数是一个对象，是给this准备的。接着在调用函数时将其绑定到this。

``` javascript
function foo(){
    console.log(this.a);
}
var obj = {
    a:2
}
foo.call(obj); // 2
```

通过foo.call(obj)在调用foo时，把this强行绑定到obj上。

如果传入了一个原始值作为第一个参数，如string，boolean等，就会转换为该原始值的对象形式（new String(), new Boolean()等）。也就是我们所说的“装箱”。一旦给一个函数显式绑定了this，之后无论怎么调用该函数，塔总会手动在绑定的this上调用函数。

```javascript
function foo(){
    console.log(this.a);
}
var obj = {
    a:2
}
var bar = function(){
    foo.call(obj);
};
bar();//2
setTimeout(bar,200);//2
bar.call(window);//2

```

这也就解决了我们之前说过的绑定this丢失的问题，我们称之为硬绑定。该技术的主要应用场景就是创建一个包裹函数，负责接收参数并返回值。

```javascript
function foo(something){
    console.log(this.a,something);
    return this.a + something;
}
var obj = {
    a:2
}
var bar = function(){
    return foo.apply(obj,arguments);
}
var b = bar(3);//2 3 
console.log(b); //5
```

硬绑定是一种非常常用的模式，所以ES5提供了内置方法 Function.prototype.bind，用法如下：

```javascript
function foo(something){
    console.log(this.a, something);
    return this.a+ something;
}
var obj = {
    a:2
}
var bar = foo.bind(obj);
var b= bar(3); //2 3
console.log(b); //5
```

bind 会返回一个硬编码的新函数，它会把你指定的参数设置为this的上下文并调用原始函数。

API 调用的上下文，很多内置API中的方法都类似bind，确保回调函数使用指定的this。

```javascript
function foo(el){
    console.log(el,this.id);
}
var obj ={
    id:"awesome"
};
[1,2,3].foreach(foo,obj); // 1 awesome 2 awesome 3 awesome
```

实际上foreach的第二个参数就绑定了this。



#### new 绑定

在js中，构造函数只是一些使用new操作符时被调用的函数，它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。

使用new来调用函数，会自动执行下面的操作。

1. 创建一个全新的对象。
2. 这个对象会被执行[[Prototype]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

```javascript
function foo(a){
    this.a = a ;
}
var bar = new foo(2);
console.log(bar.a);//2
```

### 优先级

 new > 显式绑定>隐式绑定>默认绑定



#### 软绑定

之前介绍了硬绑定，但是它缺乏灵活性，在绑定了this的值之后无法修改。如何给默认绑定一个全局对象和undefined以外的值，就可以实现和硬绑定相同的效果，同时保留隐式和显式绑定的能力。这里就需要用到软绑定。

```javascript
if(!Function.prototype.softBind){
    Function.prototype.softBind = function(obj){
        var fn = this;
        //捕获所有curried参数
        var curried = [].slice.call(arguments,1);
        var bound = function(){
            return fn.apply((!this||this===(window||global))?obj:this,curried.concat.apply(curried,arguments));
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    }
}
```



