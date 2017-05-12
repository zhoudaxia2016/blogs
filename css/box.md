## CSS核心概念之盒子模型

> CSS的其中一个重要意义就是浏览器排版。那么浏览器排版的规则是什么？又是以什么为单位来排版的呢？想知道的话就让我为你慢慢道来。

在浏览器中怎么将一个一个元素排列呢？它们的位置又该如何确定呢？浏览器其实就是对元素形成的盒子以一定的规则排列而完成排版的。所以下面内容可分为盒子模型的构成与各种盒子的排列规则两部分。

### 盒子的构成

不幸的是有两种盒子模型需要我们去了解——border-box与content-box。这两个其实是bo-sizing的两个属性值。

#### 1. content-box

盒子模型由margin,border,padding,content组成，可以翻译为：外边距，边框，内补白，内容。margin可以用CSS属性margin设置宽度，由margin-left,margin-right,margin-top,margin-bottom四个值，分别表示左右上下的margin。border可以设置宽度，也有四个方向，设置类似于margin，还可设置颜色和形状。padding可以设置宽度,类似margin。而content由内容区域大小确定，也可以通过width与height属性决定（这些都是一般情况，可能会有些特殊情况）。content区域还可以用background设置背景色，而padding也会是content的颜色。

下面看看一张图片吧：

<img src="img/box.jpg">

这应该大部份浏览器都是这种模型吧。而IE则是将要说的第二种类型。我们总结一下：

1. 盒子宽度：margin-left+border-left+width+border-right+margin-right
2. 盒子高度：margin-top+border-top+height+border-bottom+margin-bottom

#### 2. border-box

border-box模型会将padding与border的区域也视为content区域。下面再来一张图片吧：

<img src="img/iebox.jpg">

这也就是说当我们设置width与height的时候，如果又设置了padding与border，实际上内容区域会减小，甚至会没有了。这也许会让人觉得更加直观（反正我已经习惯了content-box，倒也不觉得），但是有时会很麻烦。

总结一下爱：

1. 盒子宽度：margin-left+width+margin-right
2. 盒子高度：margin-top+height+margin-bottom
3. width： border-left+padding-left+内容区域宽度+padding-right+border-right
4. height: border-top+padding-top+内容区域高度+padding-bottom+border-bottom


### 盒子排列规则

说完了盒子模型的构成，可是这样我们还是不知道浏览器怎么排列它们啊，所以呢，我们还得了解盒子是怎样排列的。

到这里我们就要了解块级元素与行内元素了，它们形成的盒子的排列规则是不一样的。

1. 块级元素：宽度自动填满父级元素的空间，从上往下排列。子元素可为块级元素或者行内元素。
2. 行内元素：宽度不会自动填满父级元素的空间，从左到右排列。子元素只能为行内元素。

块级元素有：p,div

行内元素有：input,img,span

这上面只是举个例子。我们也没必要记得清清楚楚，用到的时候自然会明白。它们也可以相互转化，通过css的一些属性。比如display属性可以设置为inline，则该元素就像行内元素一样了。

### 行内元素分类

虽然盒子模型的基本内容已经差不多讲完了，可是引出了行内元素这个概念，所以还要讲一下啊行内元素这个概念。它可以分为替换元素与不可替换元素（块级元素基本是不可替换元素）。

替换元素就是字面上的意思，表明这个元素会被替换成其他东西。比如input元素，会根据type的设置来替换成不同的表单。而img也是替换元素，会根据src加载的图片来填充内容。而p啊，span啊，里面是什么内容它就显示什么内容，所以称作不可替换元素。

行内元素之不可替换元素：

1.不能设置行高，可以设置line-height
2. 可设置水平方向padding与margin
3. 设置垂直方向的padding与margin不会增加盒子padding与margin，但是显示效果。

第三个规则是什么意思呢？先来看看以下的代码：
html：
<pre>
<code>
&lt;div&rt;块级元素&lt;/div&rt;
&lt;span&rt;行内元素&lt;/span&rt;
</code>
</pre>

css:
<pre>
<code>
span{
  background: blue;
  padding-top: 50px;
}
div{
  background: green;
  height: 50px;
}
</code>
</pre>

大家可以测试一下，span元素的padding是会出现的，并且会遮住div元素的一部分，在浏览器中查看，span元素也的padding-top也是0。所以说padding是有显示的效果的，可是在排版中，它不会加入到盒子模型padding的计算，所以padding还是0，而且其他元素就像看不到它的padding一样。而margin也可以这样解释，因为它是没有颜色的。

行内元素之替换元素：

这个除了有行内元素之前所说的特性，还有：

1. 可以设置宽高
2. 可以设置padding与margin

### 总结

因为这是CSS的非常基本的概念，所以我首先讲盒子的概念，然后下次再接着讲包含块的概念，将CSS的基本概念都理解明白！
