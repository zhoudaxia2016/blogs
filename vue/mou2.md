## 自己造一个vuejs（2）

> 上次讲到vuejs基本的数据绑定原理不过还没有详细讲怎么编译一个更新的函数出来，这次我们来认真看看这个函数是怎么构建的。

上一篇：[传送门](http://maozzhou.cn/detail/27)

代码： [地址](http://github.com/zhoudaxia2016/mou)

我们再重新疏理一下这个更新函数：

1. 编译模板时每一个节点的编译都会产生一个更新函数，用来更新该节点。
2. 将该函数存于全局变量**Watcher.tmp**。
3. 调用该函数，创建该节点的内容。
4. 若该函数访问了某个变量（data的属性），将**Watcher.tmp** 保存于dep。这里利用了get属性。
5. 之后若重新设置data的某个属性，利用dep保存的更新函数重新计算某个节点。这里利用了set属性。

首先我们将{{ prop }}替换为+prop+：

<pre>
<code>
var p = /{{([^}]*)}}/g;
var text = node.nodeValue.replace(p,function(m,exp){
  return "`+(" + exp + ")+`";
});
</code>
</pre>

这样我们就能将prop与前后两个字符串连接起来。

不过最大的问题出现了，比如我们想访问data.prop，我们会这样写{{ prop }},直接替换成prop当然是访问不到data.prop的。我们有两个方法：

1. 在prop添加data.
2. 添加with(data)语句。

我使用的是第二种方法。

我们将以上替换后的字符串相加的式子在with(data)下面执行：

<pre>
<code>
var code = ["var tmp = '';"];
code.push("with(data){tmp = tmp + `");
code.push(text + "`};");
code.push("return tmp;");
code = code.join('');
</code>
</pre>

然后拼接成了一个函数代码，最后再用Function构造函数：

<pre>
<code>
var fn = new Function('data',code);
new Watcher(function(){node.nodeValue = fn(data);});
</code>
</pre>

为什么要用Function构造函数？因为我们的代码原来就是字符串（**innerText**)，只能将它重新替换成字符串代码，然后执行它。

### 结语

好了，最基本的内容都已经讲完了，大家也可以动手自己尝试一下吧！下回还有什么可以研究呢？

1. v-指令:for,bind,if,show,on
2. 表单双向绑定
