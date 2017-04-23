## JS 重要的基本概念之this

> 上一节讲了[JS重要的基本概念之作用域](http://maozzhou.cn/detail/15),里面的this的概念只是简单带过，今天我们一起来看看它是如何绑定的吧。

我们来回顾一下this的概念吧。它是属于执行上下文的概念里面的，在一段代码执行之前，它就会绑定一个对象，其中的绑定规则就是我们需要了解的。

JS的作用域是静态作用域，在函数的定义的时候就确定的，而**this** 却不是这样的，它是由运行的时候函数所属于的对象来确定的。其中又许多情况，我先总结我知道的所有情况吧，以后遇到新的情况再来补充。

### 1. 普通函数的调用

绑定全局对象，在浏览器中即是**window**。

<pre>
<code>
function f(){this.a = 1;}
f();
console.log(a); // 输出1
</code>
</pre>

f()就是一个普通函数的调用，this.a即引用了全局对象的属性a，所以最后全局下的a为1。

### 2. 构造函数调用

this绑定到了新创建的对象，这就和c++差不多了。通过this我们可以设置对象的属性。

<pre>
<code>
function f(){this.a = 1;}
var o = new f();
console.log(this.a); // 输出undefined
console.log(o.a); // 输出1
</code>
</pre>

这次就是以构造函数的方式调用了，全局环境的a就是undefined了，而创建的对象通过this设置a的值，所以o.a为1。

> 在严格模式下，这种情况this为undefined

### 3. 在对象上调用

在对象上绑定一个函数，那么通过该对象调用这个函数的话，这个函数的this就会绑定这个对象。这样可能意义是能够让函数复用吧。

<pre>
<code>
function f(){console.log(this.x);}
var a = {x: 1,f};
var b = {x: 2,f};
a.f(); // 输出1
b.f(); // 输出2
</code>
</pre>

我们定义两个对象，两个对象都有同一个函数，可是它们的this是不一样的,所以输出的x也是不一样。

### 4. apply与call

apply与call函数可以重新绑定一个对象给this。

<pre>
<code>
function f(){console.log(this.a)}
var a = {x: 1};
var b = {x: 2};
f.apply(a); // 输出1
f.apply(b); // 输出2
</code>
</pre>

### 5. 箭头函数

这是ES6的内容，为了让this像绑定在定义它的时候的对象上面。

<pre>
<code>
var x = 'global';
var f1 = function(){console.log(this.x);}
var f2 = ()=>{console.log(this.x)}
var o = {x:1,f1,f2}
o.f1(); // 输出 1
o.f2(); // 输出 'global'
</code>
</pre>

从上面的例子看起来箭头函数的this绑定就像是JS的作用域的绑定一样，是在函数调用之前就确定的！f2是箭头函数，它的this在定义的时候是在全局环境下的，所以绑定到window上面。而f1不是箭头函数，它在执行o.f1()时才确定为o，所以才会有上面的结果。
