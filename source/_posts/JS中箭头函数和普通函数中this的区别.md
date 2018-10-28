# 箭头函数与普通函数的区别
作为ES6中新加入的箭头函数语法，深受广大开发人员的喜爱，也是平时前端面试过程中经常会被提及问道的典型题目。它不仅简化了我们的代码，而且也让开发人员摆脱了“飘忽不定”的this指向，本文就箭头函数与普通函数的区别进行一些分析。

在我看来，面试官最关注的也是两者最关键的区别就是this指向的区别，普通函数中的this指向函数被调用的对象，因此对于不同的调用者，this的值是不同的。而箭头函数中并没有自己的this（同时，箭头函数中也没有其他的局部变量，如this，argument，super等），所以箭头函数中的this是固定的，它指向定义该函数时所在的对象。

## 普通函数
相信大家对普通函数的用法已经非常熟悉了，下面我们举一个简单的例子。
```
var a  = 3;
var obj = {
	a : 1,
	foo : function(){
		console.log(this.a);
	}
}
obj.foo(); //1
var bar = obj;
bar.a = 2;
bar.foo(); //2
var baz = obj.foo;
baz(); //3
```
上述代码中，出现了三种情况：
1. 直接通过obj调用其中的方法foo，此时，this就会指向调用foo函数的对象，也就是obj;
2. 将obj对象赋给一个新的对象bar，此时通过bar调用foo函数，this的值就会指向调用者bar；
3. 将obj.foo赋给一个新对象baz，通过baz()调用foo函数，此时的this指向window；

由此我们可以得出结论：
- 普通函数的this总是指向它的直接调用者。
- 在严格模式下，没找到直接调用者，则函数中的this是undefined。
- 在默认模式下（非严格模式），没找到直接调用者，则函数中的this指向window。

再考虑一下下面的情况：
```
var obj = {
	a : 1,
	foo : function(){
	 	setTimeout(
			function(){console.log(this.a),3000})
	}
}
obj.foo(); //undefined
```
你可能会认为此时的输出应该为1，但是结果却是undefined。因为此时this的指向是全局的window对象。
通过以上例子，可以得出以下总结：
- 对于方法（即通过对象调用了该函数），普通函数中的this总是指向它的调用者。
- 对于一般函数，this指向全局变量（非严格模式下）或者undefined（严格模式下）。在上例中setTimeout中的function未被任何对象调用，因此它的this指向还是window对象。因此，这也可以总结成：javascript 的this 可以简单的认为是后期绑定，没有地方绑定的时候，默认绑定window或undefined。

如果我们希望可以在上例的setTimeout函数中使用this要怎么做呢？在箭头函数出现之前，我们往往会使用以下两种方法：
1. 在setTimeout函数的外部，也就是上层函数foo内部通过将this值赋给一个临时变量来实现。
```
var obj = {
    a : 1,
    foo : function(){
        var that  = this;
        setTimeout(
            function(){console.log(that.a),3000})
    }
}
obj.foo(); //1
```
2. 通过bind()来绑定this。
```
var obj = {
    a : 1,
    foo : function(){
        setTimeout(
            function(){console.log(this.a),3000}.bind(this))
    }
}
obj.foo(); //1
```
这种现象在ES6引入箭头函数后得到了改善。

##箭头函数
箭头函数是ES6中引入的新特性，使用方法为：
```
()=>{console.log(this)}
```

其中()内是要带入的参数，{}内是要执行的语句。箭头函数是函数式编程的一种体现，函数式编程将更多的关注点放在输入和输出的关系，省去了过程的一些因素，因此箭头函数中没有自己的this，arguments，new target（ES6）和 super(ES6)。箭头函数相当于匿名函数，因此不能使用new来作为构造函数使用。
箭头函数中的this始终指向其父级作用域中的this。换句话说，箭头函数会捕获其所在的上下文的this值，作为自己的this值。任何方法都改变不了其指向，如call(), bind(), apply()。在箭头函数中调用 this 时，仅仅是简单的沿着作用域链向上寻找，找到最近的一个 this 拿来使用，它与调用时的上下文无关。我们用代码来解释一下。
```
var obj = {
    a: 10,
    b: () => {
      console.log(this.a); // undefined
      console.log(this); // Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, frames: Window, …}
    },
    c: function() {
      console.log(this.a); // 10
      console.log(this); // {a: 10, b: ƒ, c: ƒ}
    },
    d:function(){
        return ()=>{
            console.log(this.a); // 10
        }
    },
    e:function(){
      setTime
    }

  }
  obj.b(); 
  obj.c();
  obj.d()();
```
简单分析一下代码，obj.b（）中的this会继承父级上下文中的this值，也就是与obj有相同的this指向，为全局变量window。obj.c()的this指向即为调用者obj，obj.d().()中的this也继承自父级上下文中的this，即d的this指向，也就是obj。
通过这个例子，也就可以大概的让我们理解普通函数中的this和匿名函数中的this指向差别，从而更好的在工作中根据我们的需求正确合理地使用这两种函数。