## CSS核心概念之bfc

> 终于到讲一个重量级的概念了，bfc在CSS的概念中应该是算相当重要的一个。bfc全称叫做block formatting context，翻译过来就是块级格式上下文。

我对bfc的理解就是浏览器渲染页面的一种环境，在该环境下还有对应的布局规则。所以我们要着重理解bfc是怎么创造的，bfc的布局规则。最后当然还要问知道这个有什么用处拉。

### bfc创建

1. overflow不为visiable
2. float不为none
3. position不为static或者relative
4. display为inline等等（后面的不熟悉，所以不说了）

### bfc里面的元素布局规则

1. 块级元素盒子垂直排列
2. 子元素尽量靠着包含块的左上排列（对于从左到右的环境）
3. 相邻块级元素盒子的margin会折叠
4. bfc里面的元素与外部元素不会互相影响
5. bfc的高度计算会包括float元素
6. bfc区域不会与float元素重叠


上面的概念可能有些不能理解，不过我们可以从下面的运用了解它们。

### bfc利用

#### 解决相邻元素margin折叠

看以下代码：

<pre>
<code>
&lt;div style="margin-bottom:50px">&lt;/div>
&lt;div style="margin-top:60px">&lt;/div>
</code>
</pre>

则这两个元素的border-box（不包括margin的形成的box）会相距60px而不是50+60px。即会取两个元素最大margin（第3个规则）。有时候这会非常方便，可是有时候这不是我们想要的结果，所以我们得想办法防止margin折叠（我当然不会说只设置其中一个元素的margin这么不负责任的话，那相当于没有这个问题而不是解决这个问题）。

因为浮动元素与inline-block元素margin不会被折叠，所以可以将其中一个元素设置为inline-block或者将最后一个元素设置浮动。

#### 解决子元素的margin-top影响到父元素的margin-top

如果父元素没有设置padding-top和border-top的话，子元素的margin-top就会影响到父元素，比如：

html代码：

<pre>
<code>
&lt;div class="p">
  &lt;div class="c">&lt;/div>
&lt;/div>
</code>
</pre>

css代码：

<pre>
<code>
*{margin: 0; padding: 0;}
div{
  height: 100px;
  width: 100px;
  background: #aaa;
}
.c{
  background: #aaf;
  margin-top: 100px;
}
.p{
  background: #afa;
}
</code>
</pre>

结果：

<img src="/static/blog/images/mc.png" alt="margin坍塌例子">

本来它们之间应该没有距离才对，可是两个div想距了第二个元素的子元素的margin-top的值。这肯定不是我们想要的结果了，子元素的margin应该在父元素的区域下起作用才合理。

我们可以利用第4条规则让第二个元素创建一个新的bfc环境，然后它里面的子元素就不会影响到父元素的margin了。将css代码改为：

<pre>
<code>
*{margin: 0; padding: 0;}
div{
  height: 100px;
  width: 100px;
  background: #aaa;
}
.c{
  background: #aaf;
  margin-top: 100px;
}
.p{
  float: left;
  background: #afa;
}
</code>
</pre>

结果为：

<img src="/static/blog/images/mcf.png" alt="margin坍塌解决">

#### 高度坍塌

当我们设置一个元素浮动时，它的父元素如果没有其他元素了，父元素的高度就为0了。因为float元素脱离了文档流，父元素相当于没有它这个儿子。

这个例子之前讲过了,可以去这篇博客详细了解：[高度坍塌的解决方法](http://maozzhou.cn/detail/22)

基本原理是：我们可以利用规则第5点，在bfc中高度计算包括float元素。在父元素中使用适合的方法创建一个bfc，然后父元素的高度就会包括float的子元素了。

####  自适应两栏布局

代码：

<pre>
<code>
&ltdiv style="float:left;width:100px;height:50px;background:#ffa">
&lt;/div>
&lt;div style="overflow:hidden">
Consectetur maiores laudantium perspiciatis dicta minima Nemo dicta eius et vero autem. 
Obcaecati fuga quibusdam saepe cumque assumenda. Veritatis voluptate 
minima ex tenetur dolorem culpa Nemo et quas praesentium aspernatur
&lt;/div>
</code>
</pre>

因为设置了左浮动，文字就会围绕在浮动元素周围，这是浮动的特性。有时候我们需要把它们分开，所以上面的代码在文字所在的元素加了overflow属性为hidden，触发了bfc，所以由于规则的第6点，浮动元素不能与bfc重叠，所以文字就不会围绕在浮动元素周围了，结果为：


<img src="/static/blog/images/adapt.png" alt="利用bfc的自适应布局">

这样的话我们可以只设置浮动元素的宽度，剩下的元素会自动填满剩下的宽度，形成了自适应的效果。
