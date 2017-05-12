## 令人疑惑的 ==

> JS中有两个比较是否相等的运算符，==和===。===是不会进行隐士的类型转换的，若类型不同则必定会返回false。而==会进行类型转换，当类型不同的变量比较时，会出现我们意想不到的结果。所以来看一下==的详细比较规则吧。

当比较的类型一样时，==会按照===的方式比较，不用过多考虑它。我们下面来看一下当类型不一样时候该怎么比较。要理解==比较的规则，我觉得可以将比较的变量的类型重新分为三种：

1. null,undefined
2. string,number,boolean
3. object

为什么这样分呢？因为如果两个变量都是相同类型（这里的类型指上面分成的三种类型,下面的类型都是这个意思），它们的比较规则是一样的。这样我们就好分类的，由计数原理可得共有6种情况。其实还有一种出现NaN的情况，不过出现NaN就会返回false，太简单了，不用管它，记住就行了。

下面就是这6种情况：

### 1. 同种类型比较

若都是第一种类型(undefined与null），则返回true
比如：

<pre>
<code>
var a; // a为undefined
var b = null;
console.log(a == b) // true 
</code>
</pre>

若都是第二种类型（string，number，boolean），则将它们用Number转为number再比较。

<pre>
<code>
console.log(1 == '1',Number('1')) // true 1
console.log(0 == '',Number('')) // true 0
console.log(1 == '1a',Number('1a')) // false NaN
</code>
</pre>

若都是第三种类型，因为对象的比较是根据地址来判断的，任何对象都不会相等，除非两个变量引用同一个对象,才会相等。所以这种情况返回false，只要不是引用同一个对象。

### 2. 一个变量是第二种类型，另一个变量是第三种类型

这是比较难以理解的一部分了，也是会引起我们疑惑的情况，比如：

<pre>
<code>
![] == [] // true
![] == 1 // false
![] == 0 // true
!{} == {} // false
!{} == 1 // false
!{} == 0 // false
</code>
</pre>

看完以下部分肯定能让你理解上面的代码的，不行的话用评论来砸我！

上面第二种类型互相比较的时候会使用Number函数将非number类型的转换为number类型，对象与第二种类型应该是不能直接比较的，那是否也有将对象转换为第二种类型的方法呢？答案就是有，就是valueOf和toString，下面的例子可以验证：

<pre>
<code>
var a = [1,2,3];
console.log(a == 'Hello'); // false
a.valueOf =  function(){return 'Hello';}
console.log(a == 'Hello'); // true 

var b = [1,2,3];
console.log(b == 'Hello'); // false
b.toString =  function(){return 'Hello';}
console.log(b == 'Hello'); // true 
</code>
</pre>

> 我们也不能看到==的真实情况是怎么比较的，除了去看标准，我们可以自己做一些试验来验证别人说法。

上面的两个例子都是在修改了valueOf或toString方法成返回字符串Hello的函数之后，然后a，b两个数组对象都与字符串Hello相等了。这说明了数组对象在和字符串比较的时候，肯定是通过了这两个方法，至少与这两个方法返回的结果比较是其中一个步骤。得到这样的结果就算验证了数组对象与字符串比较的时候，会调用数组的valueOf与toString方法得到一个结果再与字符串比较。我们还可以用其他对象和number或者boolean类型来比较，大家不相信可以再试验其他例子。

第二个我们需要验证首先会先调用哪个方法。

<pre>
<code>
var a = [1,2,3];
a.valueOf = function(){console.log('Invoking valueOf');return 'valueOf';}
a.toString = function(){console.log('Invoking toString');return 'toString';}
console.log(a == 'valueOf',a == 'toString') // Invoking valueOf Invoking valueOf true false

var a = {};
a.valueOf = function(){console.log('Invoking valueOf');return 'valueOf';}
a.toString = function(){console.log('Invoking toString');return 'toString';}
console.log(a == 'valueOf',a == 'toString') // Invoking valueOf Invoking valueOf true false

function Obj(){}
var a = new Obj();
a.valueOf = function(){console.log('Invoking valueOf');return 'valueOf';}
a.toString = function(){console.log('Invoking toString');return 'toString';}
console.log(a == 'valueOf',a == 'toString') // Invoking valueOf Invoking valueOf true false

var a = new Date();
a.valueOf = function(){console.log('Invoking valueOf');return 'valueOf';}
a.toString = function(){console.log('Invoking toString');return 'toString';}
console.log(a == 'valueOf',a == 'toString') //Invoking toString Invoking toString  false true
</code>
</pre>

从上面可以非常明星地看到除了Date对象，其他对象都是调用了valueOf来比较。对于Date对象，我们可以将它的valueOf方法为备胎方法，对于其他对象，toString为其备胎方法。换一句话来说，什么时候，才能用到备胎方法呢？答案是第一个调用的方法返回对象的时候：

<pre>
<code>
var a = {};
a.valueOf = function(){console.log('Invoking valueOf');return {};}
a.toString = function(){console.log('Invoking toString');return 'toString';}
console.log(a == 'valueOf',a == 'toString') // Invoking valueOf Invoking toString Invoking valueOf Invoking toString false true
</code>
</pre>

从上面的例子可以看到，当valueOf方法返回对象时，使用==运算符时，顺序调用了valueOf，toString，即备胎方法派上用场了！我们也可以用一个词语来表述当valueOf返回什么值时不用再调用toString方法——原始值。原始值可以为：null，undefined，number，boolean，string

从上面的试验，我们总结一下结论：

1. 对象与number，boolean，string比较时，会调用valueOf方法或者toString方法得到了结果再与number，boolean，string比较。
2. 首先调用其中一个方法，若返回了非对象的值，则直接用此值比较。否则再调用备胎方法，用备胎方法返回的值进行比较。

### 第二种类型或者第三种类型与第一种类型比较

简单，返回false，不用比较。

可以用下面的例子来说明不会调用valueOf与toString方法：

<pre>
<code>
var a = {};
a.valueOf = function(){console.log('Invoking valueOf');return null;}
a.toString = function(){console.log('Invoking toString');return '123';}
console.log(a == null) // false 
</code>
</pre>

### 总结

规则应该就是这么多了，通过这次的学习，让我明白了多试验代码才能理解JS的标准啊！
