## JS重要的基础概念之作用域

> 作用域最基本的含义就是我们定义的变量在哪里使用才能有效，这个概念应该在所有程序语言都有的，可是实现的方式，遵循的方法可能都不一样，我们现在来了解一下JS是怎么实现的和我们如何利用它吧。

### 作用域

作用域应该是离我们最接近的概念。在JS中，有全局作用域和函数作用域（在ES6中加入了块作用域）。在全局作用域中的变量即在程序的任何地方都可以使用，即使在函数里面也是可以的。而函数作用域里面的变量只能在该函数中使用。

当然这个概念并不能帮助我们什么，我们得理解具体的实现机制才有意义，我们会从执行上下文来详细了解。

### 执行上下文

作用域是一个抽象的让我们好理解的概念，而执行上下文则是一个实际的概念了，通过它我们才能利用作用域实现我们的程序。

执行上下文可以理解为对应一个执行环境的对象，这个环境可以为全局环境(首次进入程序与程序执行完毕）与函数环境(每一次函数调用），或者是**with** 形成的环境。这个对象为我们提供了作用域的服务，还有**this** 是指向哪里的问题，也是它确定的。执行上下文对象包括：

1. 变量对象（VO）
2. 作用域链
3. this的绑定

#### 变量对象

变量对象帮我们把在该作用域下可定义的变量保存起来。它有两个阶段：

1. 变量初始化阶段
2. 代码执行阶段

在变量初始化阶段，会扫描我们的代码，填充我们在该作用域下定义的变量。这是在程序执行之前就完成的，也把JS的作用域称为静态作用域也是这个原因。这些变量包括：

1. 函数参数，arguments对象（如果是在一个函数里面）
2. 函数声明
3. 变量声明

变量声明如果没有被赋值，会等于**undefined** ，而没有声明的变量引用会出现**error**。

在这里有一个变量声明提升的概念，比如：

<pre>
<code>
function f(){
  console.log(a); //输出 undefinded
  var a = 1;
  console.log(a); //输出 1
}
f();
</code>
</pre>

为什么会这样呢？因为声明的变量会提前到函数顶部进行声明，而声明的函数会把函数的定义也会提前，比如：

<pre>
<code>
function f(){
  ff();
  function ff(){};
}
f();
</code>
</pre>

这样的代码不会出错，因为函数的声明会提前的开头。而变量只会将声明提前，不会将定义也提前。（这其中还有函数声明与函数表达式的区别，有点难，所以暂且不深入下去了）。

这些都是在代码执行之前就会完成的了，即每一个函数，全局环境，with形成的环境都会有一个变量对象。

#### 作用域链

作用域链是当前执行环境的变量对象再加上上一级的作用域链形成的，可以将作用域链看成一个数组。在搜索变量时，先搜索当前的变量对象，然后再搜索上一级的变量对象，直到找到全局变量对象，若还是找不到，就会抛出错误了。比如下面的代码：

<pre>
<code>
var a = 1;
function inner(){
  console.log(a);
}
function f(){
  var a = 2;
  console.log(a); //输出2
  inner(); //输出1
}
f();
</code>
</pre>

f函数执行时，形成一个执行上下文对象，它的作用域链的变量对象有：[f的变量对象，全局变量对象]，它在它的变量对象中找到**var a=2**，所以输出了2。f函数里的inner函数执行时，又形成了一个执行上下文对象，它的作用域链为：[inner的变量对象，全局变量对象]。在inner的变量对象中没有a，所以往上寻找，在全局变量对象中找到了a为1。所以在这里也可以理解到静态作用域这个概念，因为inner虽然在f里面执行，并不会影响到inner的作用域链，因为inner在定义的时候它的作用域链已经决定了。

#### this

this是用来实现面向对象的作用的，通过它我们可以引用到当前的对象的变量，这和变量对象里面的变量是完全不一样的。比如用new运算符执行构造函数，this绑定的是新建立的这个对象，然后通过this来修改这个对象的数据。this的绑定原理也可以讲到很多内容，所以以后在讲它了。

### 注意点

在我的理解中，我认为**eval** 与**with** 都创造了新的执行上下文（找了很多资料，相关的东西很少，所以自己做了一点不知道正确与否的总结，如果你有不同的想法一定要告诉我啊！

#### eval函数

eval是一个函数，所以当然也能创建一个执行上下文。不过它的作用域链与变量对象就是调用它的上下文（所以可以修改当前上下文的变量），而this的指向也是一样的。而window.eval的作用域为全局作用域，this当然也是指向window了。

#### with语句

with语句就有点神奇了，新创建的执行上下文的变量对象可以由我们指定，this的指向与上级的this是一样的。作用域链就是该变量对象再加上上级的作用域链了。

### 总结

我们来总结一下这几个概念吧：

1. 作用域：规定变量的访问范围，是抽象的概念。
2. 执行上下文：执行一段代码时的一个用来提供一个变量访问控制的对象，是实际存在的概念。
3. 变量对象：全局环境，函数环境，with环境都有与之对应的一个保存该环境的变量的对象。
4. 作用域链：当前变量对象与上级作用域链形成的一个数组，存在于执行上下文对象中。
5. this：存在于执行上下文对象中，表示执行该环境的对象。

在作用域这个概念中，我们可以引出this以及闭包的概念，所以在这之后我们再一起探讨这两个概念吧。
