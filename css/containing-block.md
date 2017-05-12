## CSS核心概念之包含块

> 如果你用多了百分比属性值或者绝对定位，你才会遇到这个概念。当我们使用绝对定位时，top的值时相对于哪里的？当我们使用100%作为width的值，是相对于谁的宽度？这些问题的解决就需要我们去了解包含块的概念了。

### 包含块是什么

包含块是一个矩形，每一个元素形成的盒子（盒子模型不知道？看我另一篇博客吧：[盒子模型](http://maozzhou.cn/detail/23))都有一个包含块。当我们使用top时，就是以这个矩形的上面的一条边作为基准，top的值为a就是说距离这条边向下a的距离来放这个盒子。其他的bottom，left，right也是类似的。

包含块除了提供了位置的计算，还提供了尺寸的计算。比如一个元素的width的值设置为50%，其实就是设置为该包含块宽度的50%。

### 包含块的确定

总共有四种情况：

#### 1. 定位是static或者relative

若父元素非行内元素，则包含块就是该父元素的内容区域了。否则继续找父元素的父元素，直到找到初始包含块，也就是下面要讲的概念。

#### 2. 初始包含块

根元素即html元素的包含块是一个和视口（大概是浏览器显示网页内容的整个界面）一样大的矩形，且大小不会改变。

#### 3. 定位为fixed

包含块为视口形成的矩形。

#### 4. 定位为absolute

这个就点难理解了。它也是向上找祖先，并且是一个非static定位的祖先。包含块就由它决定，根据该祖先是块级元素还是行内元素，可以分为两种情况：

1. 祖先为块级元素：包含块是该祖先的内容区域加上padding区域。
2. 祖先为行内元素： 因为是行内元素，祖先的内容区域是由一个一个内容框形成的。如果祖先的内容是文字，则一行文字会形成一个框。若祖先的direction属性值为ltr（默认是这个属性，文字会从左到右排列），则包含块的上边界与左边界为祖先内容区域的第一个框的上边界与左边界，包含块的下边界与右边界为最后一个框的下边界与右边界。若direction为rtl，则包含块的上边界与右边界为祖先内容区域的第一个框的上边界与右边界，包含块的下边界与左边界为最后一个框的下边界与左边界。（还包含padding区域）

第二点我也是由我的理解来修改了一点点，是根据[这篇文章](http://www.w3help.org/zh-cn/kb/008/)来理解的。

其实第二点也可以这么理解：与第一点是统一的，只是行内元素的内容区域是一行一行排列的，将一行一行首尾拼接起来，就相当于一行内容了。所以如果拼接起来的话，left当然是第一行的left，right当然是最后一行的right拉（文字从左到由排列）。

如果不理解文字说明的话，可以看以下的图片说明。

### 测试最后一点

我在opera浏览器测试的结果与上面说明是一样的，可是在火狐浏览器的结果却不是这样的。

html代码:
<pre>
<code>
&lt;!-- tl类表示top属性值为0，left属性值为0，br，tr，bl类似 -->
&lt;!-- left to right -->
direction:ltr
  &lt;span class="ltr">
  Lorem dolores ea debitis iure maiores. Amet magni totam culpa vero corrupti veritatis Fugiat iusto eaque id voluptatem beatae? Quasi est cum aliquam in officiis Minima esse expedita similique optio pariatur! Unde saepe fuga placeat nihil consequatur consequatur non vel Doloribus in ipsam officia quo ut ab Doloremque dicta perferendis!
    <lt;div class="tl">tl<lt;/div>
    <lt;div class="br">br<lt;/div>
  &lt;/span>
&lt;br>&lt;hr>
<!-- right to left -->
direction:rtl
  &lt;span class="rtl">
  Elit itaque suscipit harum officiis quas, dignissimos sint Reprehenderit quaerat id impedit id accusamus debitis Doloribus pariatur eum minima laborum fugit itaque Debitis beatae dolores ipsa recusandae accusamus laudantium fuga, expedita. Explicabo vero odit eos similique autem. Cum beatae mollitia officia sequi eveniet. Accusamus esse molestias reprehenderit aliquam harum laborum
    <lt;div class="tr">tr<lt;/div>
    <lt;div class="bl">bl<lt;/div>
  &lt;/span>
</code>
</pre>

css代码：

<pre>
<code>
*{margin: 0; padding: 0;}
body{
  width: 300px;
  margin: auto;
}
.rtl,
.ltr{
  background: green;
  position: relative;
}
.ltr{
  direction: ltr;
}
.rtl{
  direction: rtl;
}
.tr,
.bl,
.tl,
.br{
  position: absolute;
  width: 50px;
  height: 50px;
  opacity: 0.4;
  color: #fff;
  font-size: 30px;
  font-weight: bold;
  line-height: 50px;
  text-align: center;
}
.tl{
  background: blue;
  top: 0;
  left: 0;
}
.tr{
  background: blue;
  top: 0;
  right: 0;
}
.br{
  background: red;
  bottom: 0;
  right: 0;
}
.bl{
  background: red;
  bottom: 0;
  left: 0;
}
</code>
</pre>

在opera浏览器中的结果：

<img src="/static/blog/images/ctbo.png">
上面的代码中，我们要寻找tl,tr,bl,br的包含块，来确定它们绝对定位的位置。它们的包含块都是由其父元素行内元素组成的。

我们只看tl类这个元素，它的父元素direction为ltr。它的top与left刚好为0，即紧贴包含块的上边界与左边界，在图中它刚好贴在span的第一行的最上与最左边。与上面的描述一样！其他大家可以自己理解看看。

可是在火狐浏览器中结果是这样的：

<img src="/static/blog/images/ctbf.png">

与描述不符合，它的包含块是第一行的区域。。。不知道大家的测试结果是如何，若有其他想法，可以留言讨论！
