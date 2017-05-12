## vuejs 学习笔记——自定义项目模板

> 用着官方的项目模板总是不是很舒服，webpack模板有许多我根本还没有接触的东西，simple模板太简单了，webpack-simple模板里有我还不是很会用的es6的loader，webpack也还不会（只需要webpack -w就可以了），所以为了使用一个适合我这个初学者的模板，学习一下了如何自定义模板。


### vue脚手架介绍

vue提供工具**vue-cli**帮我们构建一个**vue**的开发环境，通过它，可以节省我们重复创建文件夹与文件的时间。

我们先下载**vue-cli**：

<pre>
<code>
npm install -g vue-cli
</code>
</pre>

使用它我们可以下载各种模板（在命令行中输入）：

<pre>
<code>
// 用法:vue init &lt;template-name> &lt;project-name>
// 例子
vue init webpack my-project // 官方模板
</code>
</pre>

本文是讲自定义的模板，所以自己定义的模板的用法是：

<pre>
<code>
vue init &lt;path-to-template-dir> &lt;my-project> // 本地模板
vue init username/repositoryname &lt;my-project> // github上面的模板 
</code>
</pre>

比如需要创建一个叫做**project**的项目，我的github帐号为**zhoudaxia2016**，在github新建一个叫做**vue-template**的仓库，就可以这样使用这个模板：

<pre>
<code>
vue init zhoudaxia2016/vue-template project
</code>
</pre>

### 怎么创建一个模板

现在来看看怎么创建一个模板吧。其实就是创建一个包括这么一些内容的文件夹：

1. meta.js 或者 meta.json // 一些配置
2. template文件夹 // 模板内容
3. README.md // github仓库都需要的拉

可以看最简单的官方模板[simple](https://github.com/vuejs-templates/simple)。

vue是一个node程序（用node执行的），vue init会根据后面的仓库名字来下载对应的模板文件夹，对**template**里面的文讲做一些处理然后复制到你新建的项目（大概就是做这些内容吧。。）

所以我们需要知道的是我们可以通过设置什么来定制对**template**文件夹的处理。下面我们来看看吧。

### 配置

我们可以通过编写meta文件来做一些配置。下面我说一下最基本的配置吧。脚手架的github地址:[vue-cli](https://github.com/vuejs/vue-cli)，下面的理解都是基于以上与官方模板。

#### 文件格式

文件格式可以是js也可以是json。若是json，则里面的内容是json格式的（这是一句废话）。而js文件需要导出数据：

<pre>
<code>
module.exports = {
  prompts: {}
  ...
}
</code>
</pre>

最重要的区别就是json文件里面的健值对都是字符串。有些非字符串的内容好像无法配置（可能我还没有完全理解）。

我们可以设置什么呢：

1. **prompts:** 创建项目时，问你一些问题
2. **filters:** 根据需要去掉一些文件
3. **completeMessage：**  成功创建项目后给你一些提示，是一个字符串
4. **helpers:** 一些函数
5. **complete：** 功能同上，不过是以一个函数形式而不是字符串
6. **metalsmith：** 不懂，很深奥

下面我们就只看看前面四个简单点的吧，因为后面两个我不是很理解。

#### Prompts

比如我想问一下这个项目叫什么名字呢？

<pre>
<code>
{
  "prompts": {
    "name": {
      "type": "string",
      "message": "Project name?"
    }
  }
}
</code>
</pre>

这样当使用vue-init，下载完模板后，会弹出：Project name？然后我们输入一个字符串就会保存在name中了。最终的目的当然不是为了保存这个数据，而是使用它。我们可以在**template**文件夹下的所有文件里面加入一句：

<pre>
<code>
{{ name }}
</code>
</pre>

然后这句就会编译成你输入的字符串了。

> 如果template只是一个简单的文件夹，那么你生成的项目就只是template的一个复制而已，这样还不如直接使用cp命令。

**type**即是这个变量的类型。我知道的还有两种类型：confirm，list

confirm就是会问你确认吗？然后你按回车或者Y会返回true，按n会返回false。那这个布尔值有什么用呢？比如：

<pre>
<code>
{
  "prompts": {
    "needAuthor": {
      "type": "confirm",
      "message": "Need author?"
    }
  }
}
</code>
</pre>

上面就是一个布尔值，可以用来：

1. 决定一个prompt有效还是无效
2. 在template中使用
3. 在filters中使用。

第一个的理解可以看下面的代码：

<pre>
<code>
{
  "prompts": {
    "needAuthor": {
      "type": "confirm",
      "message": "Need author?"
    },
    "author": {
      "when": "needAuthor",
      "type": "string",
      "message": "Author?"
  }
}
</code>
</pre>

author有一个when选项，当when的值为true时才会问：Author？

> author必须在它依赖的布尔值后面，因为问的问题是按照顺序来问的。

第二个的理解可以看：

<pre>
<code>
{
	"prompts": {
    "sass": {
       "type": "confirm",
       "message": "Use sass?",
       "default": false
    }
	}
}
</code>
</pre>

在template里面的package.json文件中：

<pre>
<code>
{{#sass}}
  "sass-loader": "^5.0.1",
{{/sass}}
</code>
</pre>

只有sass为true时才安装sass的loader。

与filters的使用在下面再说。

list类型有两种设置方法：

<pre>
<code>
{
	"prompts": {
		"lintConfig": {
			"type": "list",
			"message": "Pick a lint config",
			"choices": ["standard","airbnb","none"]
		},
		"color": {
			"type: "list",
			"message": "Pick a color",
			"choices" [
				{ 
					"name": "nameblack",
					"value": "black",
					"short": "bl"
				}, 
				{
					"name": "namered",
					"value": "red",
					"short": "r"
				},
			]
		}
	}
}
</code>
</pre>

choices就是问问题的时候列出来的选项。这样我们可以做选择题了。

### Filtes

使用方法：

<pre>
<code>
{
	"filters": {
		"src/index.js": "bool"
	}
}
</code>
</pre>

通过变量bool的true与false来确定编译template之后，template里面的src/index.js文件存与留。

### Helpers

一些函数，可以在template里面的文件中使用。比如：

<pre>
<code>
module.exports = {
	"helpers": {
		"lowercase": function (str) { return str.toLowerCase(); }
	}
}
</code>
</pre>

这里我使用了js文件的格式，因为json格式不知道如何写这个函数。在template的使用方法：

<pre>
<code>
{{ lowercase name }}
</code>
</pre>

### CompleteMessage

这个很简单，直接输入一个字符串就可以了：

<pre>
<code>
{
	"completeMessage": "OK!"
}
</code>
</pre>

然后项目文件夹创建后就会输出OK！如果你想输出项目文件夹的名字，可以用{{ destDirName }}来取得，只要写在上面的OK字符串里面就行了，它就会变成目标项目文件夹的名字。

### 小结

虽然上面都是非常基础的内容，可能还有一些细节内容没有了解到，不过根据上面的内容应该可以满足我这个初学者的要求了！
