## JS 数据属性与访问器属性

> 访问一个对象的属性与设置一个对象的属性只能有一种固定的方法的话，不是很灵活。我觉得访问器属性就是为了解决这个问题。

对象的属性可以分为数据属性与访问器属性。数据属性就是我们平时最常使用的属性，如：

<pre>
<code>
var a = {b: 1};
a.b = 2; // 设置属性的值
console.log(a.b); // 访问属性的值
</code>
</pre>

其实属性并不是那么简单，属性自己也拥有自己的特殊属性，共有四个：

1. enumerable：boolean值，表示否可以被for in 遍历
2. configurable：boolean值，表示是否可以被删除配置
3. writable：boolean值，表示是否可以被设置
4. value：本身属性的值

我们可以通过Object.getOwnPropertyDesciptor(obj,'attr')来取得这些特殊属性组成的一个对象。

那么访问器属性又是什么呢？其实它只与数据属性有两点不一样。就是将上面第三点改为set属性，第四点改为get属性。这两个属性都是一个函数，用来表示当我们设置这个属性时和访问该属性时该做什么。我们可以通过ES5的函数defineProperty来设置：

<pre>
<code>
var a = {};
Object.defineProperty(x,'attr',{
  get: function(){return 2;}
  enumerable: true,
  configurable: true,
  set: function(newValue){this.x=newValue;},
})
</code>
</pre>

当我们访问a的属性x时会调用get定义的函数，当我们设置x新的值会以这个新的值为函数参数调用set定义的函数。

除了以上方法，我们还可以有两种方法来使用它，不过不能设置属性的特殊属性：

<pre>
<code>
// 直接在对象字面量里设置属性
var a = {
  get attr(){},
}

// 在原型对象中设置
function b(){}
b.prototype = {
  get attr(){},
}
</code>
</pre>

通过这两个特殊的属性，我们就可以拦截对某个属性的设置与访问操作，为所欲为了！
