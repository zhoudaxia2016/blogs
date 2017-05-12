## 捕获JS错误小精灵

> 程序员必备之一的技能有一项叫作调试代码，可是这真的是比较困难的事情啊。所以今天我们来看看最基础的调试的技能——捕获异常。

异常就是用来装b的词语而已，实际上异常就是程序执行的出现的错误。不过这个错误已经在JS中包装成了一个对象，名字叫做异常。下面说到的异常与错误都是一个同一个东西。

如果什么都管我们的程序，当出现错误时候我们就无所适从，虽然知道错误是什么，在哪里出现的，可是实际上解释器帮我们做的这些工作可能并不是非常准确。我们有时候是知道会出现异常的，还想利用异常来干一些事情。我们还是有办法来捕获这些错误的小精灵的，捕获之后就为所欲为吧！

### 精灵球 try catch finally

要捕获小精灵，当然需要一个精灵球，而在JS中，下面就是我们的精灵球：

<pre>
<code>
function print(e){
  console.log('name',e.name);
  console.log('message',e.message);
}

try{
  a = 1;
  console.log('我会被执行，如果不出错误的话');
}
catch(err){
  print(err);
}
finally{
  console.log('我必须执行！');
}

// 结果:
// 我会被执行，如果不出错误的话
// 我必须执行！ 
</code>
</pre>

try语句就是我们甩出来的精灵球啦，当try语句里面出现错误的时候，就会被捕获到，保存到err里面，在catch语句里面，我们可以做需要做的操作。而finally语句无论是否出现错误，都会被执行的。所以上面的try语句与finally语句里面的代码会被执行。

当我们将try里面的代码改为：

<pre>
<code>
try{
  a = b;
  console.log('我会被执行，如果不出错误的话');
}
</code>
</pre>

结果为：

<pre>
<code>
name ReferenceError
message b is not defined
我必须执行！
</code>
</pre>

finally里代码果然还是执行了，而try最后的代码没有被执行，因为出现错误会终止下面代码的执行。这也是捕获异常的一个作用，比如当我们打开一个文件，在操作过程中如果出现错误，即使最后有代码让文件关闭也不会被执行，使用try语句，将关闭文件的代码放在finally代码块中就可以保证文件会关闭了。

### 错误小精灵的属性与类别

上面的print函数就是输出了错误的两个属性,name与message。当我们使用异常对象的**toString** 方法时，就会返回： name:message。

异常小精灵的类型一共有6个。

#### 1.SyntaxError

中文名字：语法错误。比较出名的小精灵，非常强大，一般的错误都是此类型。表示出现了不符合JS的语法的代码。
我们来见识一下吧：

<pre>
<code>
a = ‘1’ // SyntaxError: illegal character 用中文字符会出现这种错误
1a = '1' // SyntaxError: identifier starts immediately after numeric literal 标识符有一套规则，不能以数字开头。
</code>
</pre>

至今我还没有捕获得到这个小精灵，因为我使用try来捕获时，浏览器都是直接抛出错误，执行不了我的catch语句与finally语句。不知道是我抓小精灵的能力不够还是它真的很强大，没有人可以捕获得到。

#### 2.TypeError

中文名字：类型错误。当我们用错类型就会出现这个小精灵：

<pre>
<code>
try{
  var a = 1;
  a();
}
catch(err){
  print(err);
}
finally{
  console.log('我必须执行！');
}

// 结果：
// name TypeError
// message a is not a function
// 我必须执行！
</code>
</pre>

print函数你可以随便定义来处理你捕获的小精灵。因为a不是一个函数，我将它当作函数来调用，所以会出现类型错误，a不是函数类型。

#### 3.ReferenceError

中文名字：引用错误。当我们引用一个不存在的东西时，就会遇到它：

<pre>
<code>
try{
  var a = b;
}
catch(err){
  print(err);
}
finally{
  console.log('我必须执行！');
}

// 结果：
// name ReferenceError
// message b is not defined
// 我必须执行！
</code>
</pre>

因为b并没有声明，所以引用它就会出现这个错误。

#### 4. RangeError

中文名字：超出有效范围的错误。这通常出现在数组上面。比如：
<pre>
<code>
var a = new Array(-1);
// 或者
var a = [1,2,3];
a.length = -1;
</code>
</pre>

因为-1不能当作长度，所以出现错误。而以-1为数组作为下标，或者以比数组长度还长的数为小标不会出现错误，JS太宽容了，如果这样操作的话，有时候会出现意想不到的错误。一句话来说RangeError是当数组的长度的类型不对时候就会出现该错误。

#### URIError

encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数的使用时候，如果参数不对就会出现此错误。暂时不了解这些函数，应该还不会遇到它们。

#### EvalError

还没遇见过，也不会捕获它。

### 创造小精灵

如果我们需要自定义异常怎么办？我们可以使用它们的构造函数来创造它们，构造函数的名字就是错误类型的名字。加上它们的祖先Error共有7个异常构造函数。可以传入一个参数，表示错误的message。

<pre>
<code>
var err = new Error('error!');
// 使用err，就像平时遇到错误一样
throw err
</code>
</pre>

### catch语句的作用域

具体可参考讲with作用域的[这篇文章](http://www.cnblogs.com/zz334396884/p/4951042.html)。我觉得with语句与catch语句的作用域的原理是一样的。都是在该执行环境下加了一个对象在作用域链的头部，这个对象只能被访问，不能被扩展。比如上面那些catch语句都是把一个含有err变量的对象放在了catch语句的执行环境的顶部。当我们访问err的时候，会在顶部的这个对象找到。当我们查找其他变量，在这个对象找不到，会在上一级作用域链去找。当我们声明定义一个值时候，会定义在上一级的变量对象。

### 总结

当我们拥有了捕获了这些小精灵的强大能力，也不能乱用噢，只能用在适合的地方，不可滥用。
