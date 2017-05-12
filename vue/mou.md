## 自己造一个vuejs （1）

> 深刻理解一个框架的最有效的方法是去模仿它。通过模仿它，才能理解它的原理与设计，才能更好地使用它的功能啊！


### 我们要做什么？

vuejs最重要，最基本的概念是数据绑定的原理。比如下面的代码：

html代码：

<pre>
<code>
&lt;div id="app">
  hello
  &lt;p> {{ title }} &lt;/p>
  &lt;p> {{ content}} &lt;/p>
&lt;/div>
</code>
</pre>

js代码：

<pre>
<code>

var options = {
  el: '#app',
  data:{
    title:'This is a title',
    content:'This is content!!!',
  }
}
var mou = new Mou(options);
</code>
</pre>

options的data属性会保存到mou的data属性。我们要将title与content填进id为app的元素对应的位置里面。并且当title与content更新时，html也会更新。这就是数据绑定的概念。详细代码可以看：[代码地址](https://github.com/zhoudaxia2016/mou)


### 我们要知道什么概念？

如果要理解下面的内容的话，要清楚：

1. [js闭包](http://maozzhou.cn/detail/19)。一个函数里面定义了一个内层函数，且内层函数访问了这个函数变量，然后返回内层函数，则形成了闭包，被访问的变量不会在这个函数执行后消失。
2. js正则表达式。
3. ES5的[Object.defineProperty](http://www.cnblogs.com/weiqu/p/5860945.html)函数。
4. Function构造函数

如果上面的概念都能理解的话，你肯定也可以理解vuejs的原理的（当然你还要读一下vuejs的官方文档，要不然不知道为什么要做这些）！

### 我们要怎么做？

如果要完成第一个功能，将title填进html里面，其实很简单，只要将元素的innerHTML正则匹配{{prop}}，然后替换掉就可以了,其实就是初始化html内容（html内容称为模板比较专业），可以将该步骤称为编译。但是如果修改了prop的话，其依赖的html是不会改变的，因为我们还没有将属性设置为响应式的。

怎么设置为响应式的？看代码：

<pre>
<code>
function defineReactive(data,key,val){
  /**
   * 响应式属性设置的主要函数
   *
   * @param {Object} data
   * @param {String} key
   * @param {-} val
   */

  var dep = new Dep();
  Object.defineProperty(data,key,{
    enumerable: true,
    configurable: false,
    get: function(){
      if (Watcher.tmp){
        console.log(Watcher.tmp.update);
        dep.addCb(Watcher.tmp);
      }
      return val;
    },
    set: function(newVal){
      val = newVal;
      dep.notify();
    }
  });
  if (val && typeof val === 'object'){
    Object.keys(val).forEach(function(k){
      defineReactive(val,k,val[k]);
    });
  }
}

</code>
</pre>

data即我们实例的属性。我们会递归的调用这个函数，使得data属性所有的属性都是响应式的。我们看到在函数开头声明一个一个变量dep，是用来保存该属性的更新函数的。我们调用了defineProperty来设置属性的get属性与set属性。get与set属性其实是一个函数，它们访问了外层函数的dep变量，所以这里利用了闭包，使得在执行完defineReactive后，每一个属性在内存中都有一个dep变量与之对应，不用我们维护它们的关系。

基本的原理就是：

1. 先执行defineReactive，设置属性的get与set属性，每一个属性都有一个dep。
2. 然后调用编译函数（后面讲到），编译的时候如果模板出现{{prop}}，比如上面的{{title}},然后就会定义一个函数，更新该节点的innerText。在这个过程中会访问title属性，调用了title属性的get，将该函数丢进dep里面（get不能设置参数，get怎么访问这个函数参数？）。
3. 当我们从新设置title属性，会调用title的set，set会调用dep里面所有更新函数。

上面就是基本的原理，不过还是有点问题，下面来讨论。

### 如何编译

编译就用到了Function构造函数。编译有两种方法：

1. 直接替换元素的innerHTML
2. 替换元素的非子节点的text，然后递归下去。

我选择了第二种方法，因为这样每一个节点可以得到一个更新函数，而不是只有根节点有一个更新函数。

怎么构造一个节点的更新函数呢？通过拼接代码字符串，利用字符串的replace函数，如果是{{prop}}就替换成+prop+，然后利用Function构造函数构造一个函数。这里也利用了闭包，因为这个函数得访问该节点的一些内容。最后通过一个全局变量Watcher.tmp将这个函数传递到get里面。

<pre>
<code>
Watcher.tmp = watcher; //全局变量保存
watcher.update(); // 调用更新函数，如果模板有{{prop}}肯定会调用prop的get函数，
// 然后就能通过全局变量Wacther.tmp保存更新函数了
Watcher.tmp = null; // 不要它了
</code>
</pre>

wacther是一个对象，wacther.update就是更新函数，因为可能还要扩展update的功能，所以将它保存到了一个对象里面。

### 为什么defineReactive的参数要一个val参数？

这应该是defineProperty的问题。虽然好像data[key]应该等于val，可是这会在get属性里面不停地调用get，造成死循环（因为需要return data[key]）。利用一个val保存，因为get与set都会访问它，又形成了闭包，保存该属性的值。所以说defineProperty劫持了属性的访问，通过外层函数的val变量来保存该属性的值。

### 总结

最最基础的功能实现了，以后就是慢慢扩展它的功能啦。
