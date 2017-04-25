## JS事件机制

>因为听说在addEventListener里有个布尔参数来确定事件是在事件冒泡还是事件捕获阶段触发，让我以为事件只有在这两种情况下才会调用，所以一直以来对JS的事件机制的理解都处于模糊不清的阶段。所以今天就来攻克这个难点吧！

### 事件阶段

事件的触发会有三个阶段：
1. 事件捕获阶段
2. 事件目标阶段
3. 事件冒泡阶段

为什么需要三个阶段？只有一个目标阶段，比如鼠标点击事件，点到哪个元素，就触发它的点击事件就可以了。这样的话，事情就比较简单了，就不会有这篇文章了。但是实际上事情并没有那么简单。

问题是被触发的目标的父元素算不算被点击了？因为子元素也是父元素的一部分，实际上可能也会出现这样的需求。如果父元素也算被点击的话，那么它们绑定的事件处理函数谁先被执行呢？所以就有了事件捕获阶段与事件冒泡阶段。

1. 事件捕获阶段是从触发目标元素最外层的父元素的事件处理函数，直到目标元素。
2. 事件冒泡阶段正好相反。

这样我们使用事件就能够更加灵活。

### 事件绑定

绑定在冒泡阶段的函数会在冒泡阶段执行，绑定在捕获阶段的函数会在捕获阶段执行。如果是目标阶段，不管是绑定于冒泡阶段还书捕获阶段，都会执行该函数。

事件的绑定有两种方法：
<pre>
<code>
function cb(){} // 绑定的函数
target.onclick = cb; // 第一种方法
target.addEventListener('click',cb,false); // 第二种方法 false表示非捕获阶段，即绑定于冒泡阶段
</code>
</pre>

第一种方法比较简单，可是默认是绑定在冒泡阶段，不能修改。并且新绑定的函数会覆盖掉原来绑定的函数。

第二种方法的可以有一个布尔值参数来确定是绑定于哪个阶段。多个函数被绑定于同一个事件上面会顺序执行这些绑定的函数。

>第二种方法不会覆盖掉第一种方法绑定的函数。

### 事件委托

了解事件委托可以更加深刻理解事件机制。

当我们需要在许多子元素中绑定差不多的事件函数时，我们可以这样写：

<pre>
<code>
var lis = document.getElementsByTagName('li');
for (var i=0; i<lis.length; i++){
  lis[i].onclick = cb;
}
</code>
</pre>

这样需要一个循环遍历，当我们添加多一个子元素的时候又得在绑定一次。而事件委托就是为了解决这样的问题。我们可以将事件委托给它们的父元素的冒泡阶段来处理（在捕获阶段会又什么问题？）。

<pre>
<code>
document.getElementsByTagName('ul')[0].onclick = function(e){
  if(e.target) {
    do something;
  }
</code>
</pre>

我们可以通过筛选得到我们需要触发的目标。这里我再一次了解到在事件处理函数中event.target与this是不同的，this是触发该事件的元素。event.target是触发整个事件阶段的元素，即目标阶段的元素。

### 用法

那它有什么灵活之处呢？

当点击目标元素时，我们只希望目标元素被触发：
<pre>
<code>
target.onclick = function(e){
  // do something
  e.stopPropagation(); //阻止事件冒泡，即事件没有冒泡阶段了
}
</code>
</pre>

当点击目标元素时，我们希望它父元素在它之后被触发(当然，前提是父元素也注册了点击事件）：
<pre>
<code>
parent.onclick = function(e){}
target.onclick = function(e){
  // do something
}
</code>
</pre>

当点击目标元素时，我们希望其父元素在它之前被触发：
<pre>
<code>
parent.addEventListener('click',function(e){
  // do something
},true);
target.onclick = function(e){
  // do something
  e.stopPropagation();
}

</code>
</pre>

当点击目标元素时,我们希望先触发父元素事件，然后再到它绑定的事件，然后再触发父元素另外一个绑定的事件函数。

<pre>
<code>
parent.addEventListener('click',function(e){
  // do something
},true);
parent.addEventListener('click',function(e){
  // do something
},false);
target.onclick = function(e){
  // do something
}
</code>
</pre>
