路由其中一个很重要的职责就是加载适合的`model`，初始化数据，然后在模板上显示数据。

### 1，普通model关联

```javascript
//  app/router.js

//  ……

Router.map(function() {
    this.route('posts');
});

export default Router;
```
对于`posts`这个路由如果要加载名为`post`的`model`要怎么做呢？代码实现很简单，其实在前面的代码也已经写过了。你只需要重写`model`回调，在回调中返回获取到的`model`即可。
```javascript
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		// return Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');
		// 加载post（是一个model）
		return this.store.query('post');
	}
});
```
`model`回调可以返回一个[Ember Data](http://emberjs.com/api/data/)记录，或者返回任何的[promise](http://liubin.org/promises-book/)对象（[Ember Data](http://emberjs.com/api/data/)也是`promise`对象），又或者是返回一个简单的javascript对象、数组都可以。但是需要等待数据加载完成才会渲染模板，所以如果你是使用`Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');`获取远程数据页面上会有一段时间是空白的，其实就是在加载数据，不过这样的用户体验并不好，不过你也不需要担心这个问题，Ember已经提供了解决办法。还记得上一篇的截图的路由表吗？是不是每个路由都有一个`xxx_loading`路由，这个路由就是在数据加载的时候执行的，有关`xxx_loading`更多详细的信息在后面的博文介绍。

### 2，动态model

一个`route`有时候只加载同一个`model`，比如路由`/photos`就通常是加载模型`photo`。如果用户离开或者是重新进入这个路由模型也不会改变。
然而，有些情况下路由所加载的`model`是变化的。比如在一个图片展示的APP中，路由`/photos`会加载一个`photo`模型集合并渲染到模板`photos`上。当用户点击其中一幅图片的时候路由只加载被点击的`model`数据，当用户点击另外一张图片的时候加载的又是另外一个`model`并且渲染到模板上，而且这两次加载的`model`数据是不一样的。

在这种情形下，访问的URL就包含了很重要的信息，包括路由和模型。

在Ember应用中可以通过定义动态段实现加载不同的模型。有关动态段的知识在前面的[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)和[Ember.js 入门指南之二十路由定义](http://blog.ddlisting.com/2016/03/25/ember-js-ru-men-zhi-nan-zhi-er-shi-lu-you-ding-yi/)已经做过介绍。

一旦在路由中定义了动态段Ember就会从URL中提取动态段的值作为`model`回调的第一个参数。
```javascript
//  app/router.js
// ……

Router.map(function() {
    this.route('posts', function() {
        this.route('post', { path: '/:post_id'});
    }); 
});

export default Router;
```
这段代码定义了一个动态段`:post_id`，记得动态段是以`:`”开头。然后在`model`回调中使用动态段获取数据。
```javascript
//  app/routes/posts.js

import Ember from 'ember';
export default Ember.Route.extend({
	model: function(params) {
		return this.store.findRecord('post', params.post_id);
	}
});
```
可以看到在`model`回调中也是使用在路由中定义的动态段，并把这个动态段作为参数传递给Ember的方法`findRecord`，Ember会把URL对应位置上的数据解析到这个动态段上。

**注意**：在`model`中的动态段只在通过URL访问的时候才会被解析。如果你是通过其他方式（比如使用`link-to`进入路由）转入路由的，那么路由中`model`回调方法里的动态不会被解析，所请求的数据会直接从上下文中获取（你可以把上下文想象成ember的缓存）。下面的代码将为你演示这个说法：

#### 1，首先创建一个model：ember g model post

```javascript
import DS from 'ember-data';

export default DS.Model.extend({
	title: DS.attr('string'),
	body: DS.attr('string'),
	timestamp: DS.attr('number')
});
```
定义了3个属性，`id`属性不需要显示定义，ember会默认加上。

#### 2，在posts下增加一个子路由

```javascript
Router.map(function() {
    this.route('posts', function() {
        this.route('post', { path: '/:post_id'});
    }); 
});
```
然后用[Ember CLI](http://ember-cli.com/user-guide)命令（`ember g route posts/post`）在创建路由`post`，同时也会自动创建出子模板`post.hbs`。创建完成之后会得到如下两个文件：
```
1.app/routes/posts/post.js
2.app/templates/posts/post.hbs
```
修改路由`posts.js`，在`model`回调中返回设定的数据。
```javascript
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {
		return [
		    {
		    	"id":"-JzySrmbivaSSFG6WwOk",
		        "body" : "testsssss",
		        "timestamp" : 1443083287846,
		        "title" : "test"
		    },
		    {
		    	"id":"-JzyT-VLEWdF6zY3CefO",
		        "body" : "33333333",
		        "timestamp" : 1443083323541,
		        "title" : "test33333"
		    },
		    {
		    	"id":"-JzyUqbJcT0ct14OizMo" ,
		        "body" : "body.....",
		        "timestamp" : 1443083808036,
		        "title" : "title1231232132"
		    }
		];
	}
});
```
修改`posts.hbs`，遍历显示所有的数据。
```html
<ul>
	{{#each model as |item|}}
		<li>
			{{#link-to 'posts.post' item}}{{item.title}}{{/link-to}}
		</li>	
	{{/each}}
</ul>

<hr>
{{outlet}}
```
修改子路由`post.js`，使得子路由根据动态段返回匹配的数据。
```javascript
// app/routes/posts/post.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {
		console.log('params = ' + params.post_id);
		return this.store.findRecord('post', params.post_id);
	}
});
```
注意打印信息语句`console.log()`;，然后接着修改子模板`post.hbs`。
```html
<!-- app/templates/posts/post.hbs -->

<h2>{{model.title}}</h2>
<p>{{model.body}}</p>
```
到此，全部所需的测试数据和代码已经编写完毕。下面执行[http://localhost:4200/posts](http://localhost:4200/posts)，可以看到界面上显示了所有在路由posts的model回调中设置的测试数据。查看页面的HTML代码：

![html](/content/images/2016/03/60.png)

可以看到每个连接的动态段都被解析成了数据的id属性值。
注意：随便点击任意一个，注意看浏览器控制台打印的信息。
我点击了以第一个连接，浏览器的URL变为

![html](/content/images/2016/03/61.png)

看浏览器的控制台是不是并没有打印出`params = -JzySrmbivaSSFG6WwOk`，在点击其他的连接结果也是一样的，浏览器控制台没有打印出任何信息。

下面我我们直接在浏览器地址栏上输入：[http://localhost:4200/posts/-JzyUqbJcT0ct14OizMo](http://localhost:4200/posts/-JzyUqbJcT0ct14OizMo)然后按enter执行，注意看浏览器控制台打印的信息！！！此时打印了`params = -JzyUqbJcT0ct14OizMo`，你可以用同样的方式执行另外两个链接的地址。同样也会打印出`params = xxx`（`xxx`为数据的`id`值）。

我想这个例子应该能很好的解释了Ember提示用户需要的注意的问题。
只有直接用过浏览器访问才会执行包含了动态段的`model`回调，否则不会执行包含有动态段的回调；如果没有包含动态段的`model`回调不管是通过URL访问还是通过`link-to`访问都会执行。你可以在路由`posts`的`model`回调中添加一句打印日志的代码，然后通过点击首页上的`about`和`posts`切换路由，你可以看到控制台打印出了你在`model`回调中添加的日志信息。


### 3，多模型

对于在一个`mode`l回调中同时返回多个模型的情况也是时常存在的。对于这种情况你需要在`model`回调中修改返回值为`Ember.RSVP.hash`对象类型。比如下面的代码就是同时返回了两个模型的数据：一个是`song`，一个是`album`。

```javascript
//  app/routes/favorites.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		return Ember.REVP.hash({
			songs: this.store.find('song'),
			albums: this.store.find('slbum')
		});
	}
});
```
然后在模板`favorites.hbs`中就可以使用`{{#each}}`把两个数据集遍历出来。遍历的方式与普通的遍历方式一样。
```html
<!--  app/templates/favorites.hbs -->

<h2>Song list</h2>
<ul>
{{#each model.songs as |item|}}
	<li>{{item.name}}</li>
{{/each}}
</ul>
<hr>
<h2>Album list</h2>
<ul>
{{#each model.albums as |item|}}
	<li>{{item.name}}</li>
{{/each}}
</ul>
```
到此所有路由的`model`回调的情况介绍完毕，`model`回调其实就是把模型绑定到路由上。实现数据的初始化，然后把数据渲染到模板上显示。这也是Ember推荐这么做的——就是把操作数据相关的处理放在`route`而不是放在`controller`。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
