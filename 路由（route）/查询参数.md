查询参数是在URL的问号（？）右边部分，通常是键值对形式出现。
```
http://example.com/articles?sort=ASC&page=2
```
比如这个URL的查询参数有两个，一个是`sort`，一个是`page`，它们的值分别是`ASC`和`2`。

### 1，指定查询参数	

查询参数通常是声明为`controller`类中。比如在当前活动路由`articles`下，你需要根据文章的类型`category`过滤，此时你必须要在`controller`内声明过滤参数`category`。

使用[Ember CLI](http://ember-cli.com/user/guide)新建一个`controller`、`route`：
```
ember g controller article；
ember g route articles;
```
```js
//  app/controllers/articles.js

import Ember from 'ember';

export default Ember.Controller.extend({
	queryParams: ['category'],
	category: null
});
```
绑定一个查询参数到URL，并且参数的值为`null`。当你进入路由`articles`时，如果参数`category`的值发生变化会自动更新到`controller`中的`category`；反之亦然。你可以设置一个默认值，比如把`category`设置为`Java`。可以在模板上获取这个值。
```html
<!--  app/templates/articles.hbs  -->
{{outlet}}
category = {{category}}
```
执行[http://localhost:4200/articles](http://localhost:4200/articles)，页面会显示出	`category = Java`。如果执行[http://localhost:4200/articles?category=PHP](http://localhost:4200/articles?category=PHP)，那么页面会显示`category = PHP`。

下面代码演示了怎么使用查询参数：
```js
//  app/controllers/articles.js

import Ember from 'ember';

export default Ember.Controller.extend({
    queryParams: ['category'],
    category: null,

    //  定义一个返回数组的计算属性，可以直接在模板上遍历
    filteredArticles: Ember.computed('category', 'model', function() {
	    var category = this.get('category');
	    var articles = this.get('model');

	    if (category) {
	        return articles.filterBy('category', category);
	    } else {
	        return articles;
	    }
    })
});
```
创建一个计算属性，这个计算属性是一个数组类型。由于是计算属性，并且这个计算属性关联了另外两个属性`category`和`model`，只要这两个属性其中之一发生改变都会导致`filteredArticles`发生改变，所以返回的数组元素也会跟着改变。

在`route`初始化测试数据。
```js
//  app/routes/article.js

import Ember from 'ember';

export default Ember.Route.extend({

    model(params) {
	    
	    return [
	    	{ id: 1, title: 'Bower: dependencies and resolutions new', body: "In the bower.json file, I see 2 keys dependencies and resolutionsWhy is that so? I understand Bower has a flat dependency structure. So has it got anything to do with that ?", category: 'java' },
	    	{ id: 2, title: 'Highly Nested JSON Payload - hasMany error', body: "Welcome to the Ember.js discussion forum. We're running on the open source, Ember.js-powered Discourse forum software. They are also providing the hosting for us. Thanks guys! Please use this space for discussion abo… read more", category: 'php' },
	    	{ id: 3, title: 'Passing a jwt to my REST adapter new ', body: "This sets up a binding between the category query param in the URL, and the category property on controller:articles. In other words, once the articles route has been entered, any changes to the category query param in the URL will update the category property on controller:articles, and vice versa.", category: 'java' }
	    ];
    }
});
```
下面看看怎么在模板显示数据，并且根据`category`显示不同数据。
```html
<!--  app/templates/articles.hbs  -->

<div class="col-md-4 col-xs-4">

<ul>
    <!-- 输入的值会动态更新到controller  -->
	输入分类：{{input value=category placeholder ='查询的分类'}}
</ul>

<ul>
	<!-- 关键点：这里使用的是filteredArticles而不是从route获取的model -->
	{{#each filteredArticles as |item|}}
		<li>
			{{#link-to 'articles.article' item}} {{item.title}}--{{item.category}} {{/link-to}}
		</li>	
	{{/each}}
</ul>

</div>

<div class="col-md-8 col-xs-8">
{{outlet}}
</div>
```
精彩的时刻到了！！先执行[http://localhost:4200/articles](http://localhost:4200/articles)，此时显示的是所有类型的数据。如下图：

![result](/content/images/2016/03/75.png)

接着你就可以做点好玩的事了，直接在输入框输入分类名。由于计算属性的特性会自动更新数组`filteredArticles`。所以我们可以看到随着你输入字符的变化显示的数据也在变化！这个例子也说明了[Ember](http://emberjs.com/)计算属性自动更新变化的强大！！用着确实爽啊！！
官网教程没有说怎么在模板中使用，讲得也不是很明白，就给了一句
>Now we just need to define a computed property of our category-filtered array that the articles template will render:”
也有可能是我看不懂，反正摸索好一阵子才知道要这么用！！

### 2，使用link-to指定查询参数	

`link-to`助手使用`query-params`子表达式直接指定查询参数，需要注意的是这个表达式需要放在括号内使用，切记别少了这个括号。
```html
<!--  app/templates/articles.hbs  -->
……
<ul>
	{{#link-to 'articles' (query-params category='java')}} java {{/link-to}}
	<br>
	{{#link-to 'articles' (query-params category='php')}} php {{/link-to}}
	<br>
	{{#link-to 'articles' (query-params category='')}} all {{/link-to}}
</ul>
……
```
在显示数据的ul标签后面新增上述两个`link-to`助手。它们的作用分别是指定分类类型为java、php、全部。但用户点击三个连接直接显示与连接指定类型匹配的数据（并且查询的输入框也变为链接指定的类型值）。比如我点击了第一个链接，输入显示如下图：

![result](/content/images/2016/03/76.png)

### 3，路由转换	

`route`对象的`transitionTo`方法和`controller`对象的`transitionToRoute`方法都可以接受`final`类型的参数。并且这个参数是一个包括一个`key`为`queryParams`的对象。

修改前面已经创建好的路由`posts.js`。
```js
//  app/routes/posts.js
import Ember from 'ember';
export default Ember.Route.extend({

	beforeModel: function(params) {

		//  转到路由articles上，并且传递查询参数category，参数值为Java
		this.transitionTo('articles', { queryParams: { category: 'java' }});
	}
});
```
执行[http://localhost:4200/posts](http://localhost:4200/posts)后，可以看到路由直接跳转到[http://localhost:4200/articles?category=java](http://localhost:4200/articles?category=java)，实现了路由切换的同时也指定了查询的参数。界面显示的数据我就不截图了，程序不出错，显示的都是`category`为`java`的数据。

另外还有三种切换路由的方式。
```js
//  可以传递一个object过去
this.transitionTo('articles', object, { queryParams: { category: 'java' }});

// 这种方式不会改变路由，只是起到设置参数的作用，如果你在
//路由articles中使用这个方法，你的路由仍然是articles，只是查询参数变了。
this.transitionTo({ queryParams: { direction: 'asc' }});

//  直接指定跳转的URL和查询参数
this.transitionTo('/posts/1?sort=date&showDetails=true');
```
上面的三种方式请读者自己编写例子试试吧。光看不练假把式……

### 4，选择进入一个完整的路由	

`transitionTo`和`link-to`提供的参数仅会改变查询参数的值，而不会改变路由的层次结构，这种路由的切换被认为是不完整的，这也就意味着比如`model`和`setupController`回调方法就不会被执行，只是使得`controller`里的属性值为新的查询参数值以及更新URL。

但是有些情况是查询参数改变需要从服务器重新加载数据，这种情况就需要一个完整的路由切换了。为了能在查询参数改变的时候切换到一个完整的路由你需要在`controller`对应的路由中配置一个名为`queryParams`哈希对象。并且需要设置一个名为`refreshModel`的查询参数，这个参数的值为`true`。
```js
queryParams: {
	category: {
		refreshModel: true
	}
},
model: function(params) {
    return this.store.query('article', params);
}
```
关于这段代码演示实例请查看官方提供的代码！

### 5，使用replaceState更新URL

默认情况下，[Ember](http://emberjs.com/)使用`pushState`更新URL来响应`controller`类中查询参数属性的变化，但是如果你想使用`replaceState`来替换`pushState`你可以在`route`类中的`queryParams`哈希对象中设置`replace`为`true`。设置为`true`表示启用这个设置。
```js
queryParams: {
category: {
    replaceState:true
}
}
```
### 6，将控制器的属性映射到不同的查询参数键值

默认情况下，在`controller`类中指定的查询属性`foo`会绑定到名为`foo`的查询参数上。比如：`?foo=123`。你也可以把查询属性映射到不同的查询参数上，语法如下：
```js
//  app/controllers/articles.js
import Ember from 'ember';
export default Ember.Controller.extend({
    queryParams: {
    	category: 'articles_category'
    }
    category: null
});
```
这段代码就是把查询属性`category`映射到查询参数`articles_category`上。
对于有多个查询参数的情况你需要使用数组指定。
```js
//  app/controllers/articles.js
import Ember from 'ember';
export default Ember.Controller.extend({
	queryParams: ['page', 'filter', { category: 'articles_category' }],
    category: null,
    page: 1,
    filter: 'recent'
});
```
上述代码定义了三个查询参数，如果需要把属性映射到不同名的参数需要手动指定，比如`category`。

### 7，默认值与反序列化

```js
export default Ember.Controller.extend({
    queryParams: 'page',
    page: 1
});
```
在这段代码中设置了查询参数`page`的默认值为`1`。

这样的设置会有两种默认的行为：

1.查询的时候查询属性值会根据默认值的类型自动转换，所以当用户输入[http://localhost:4200/articles?page=1](http://localhost:4200/articles?page=1)的时候`page`的值`1`会被识别成数字`1`而不是字符`'1'`，应为设置的默认值`1`是数字类型。
2.当查询的值正好是默认值的时候，该值不会被序列化到URL中。比如查询值正好是`?page=1`这种情况URL可能是`/articles`，但是如果查询值是`?page=2`，URL肯定是`/articles?page=2`。

### 8，粘性的查询参数值

默认情况下，在[Ember](http://emberjs.com/)中查询参数是“粘性的”，也就是说如果你改变了查询参数或者是离开页面又回退回来，新的查询值会默认在URL上，而不会自动清除（几乎所见的URL都差不多是这种情况）。这是个很有用的默认设置，特别是当你点击后退回到原页面的时候显示的数据没有改变。	

此外，粘性的查询参数值会被加载的`route`存储或者回复。比如，包括了动态段`/:post_id`的路由`posts`，以及路由对应的`controller`包含了查询属性`filter`。如果你导航到`/badgers`并且根据`reookies`过滤，然后再导航到`/bears`并根据`best`过滤，然后再导航到`/potatose`并根据`lamest`过滤。如下面的链接：
```html
<ul>
	{{#link-to 'posts' 'badgers'}}Badgers{{/link-to}}<br>
	{{#link-to 'posts' 'bears'}}Bears{{/link-to}}<br>
	{{#link-to 'posts' 'potatoes'}}Potatoes{{/link-to}}<br>
</ul>
```
模板编译之后得到如下HTML代码：
```html
<ul>
    <a href="/badgers?filter=rookies">Badgers</a>
    <a href="/bears?filter=best">Bears</a>
<a href="/potatoes?filter=lamest">Potatoes</a>
</ul>
```
可以看到一旦你改变了查询参数，查询参数就会被存储或者是关联到`route`所加载的`model`上。如果你想重置查询参数你有如下两种方式处理：

1.在`link-to`或者`transitionTo`上显式指定查询参数的值；
2.使用`Route.resetController`回调设置查询参数的值并回退到切换之前的路由或者是改变`model`的路由。

下面的代码片段演示了一个查询参数在`controller`中重置为`1`，同时作用于切换前`ActiclesRoute`的`model`。结果就是当返回到当前路由时查询值已经被重置为`1`。
```js
//  app/routes/article.js

import Ember from 'ember';

export default Ember.Route.extend({

	resetController(controller, isExiting, transition) {
		//  只有model发生变化的时候isExiting才为false
		if (isExiting) {
			//  重置查询属性的值
			controller.set('page', 1);
		}
	}
});
```
某些情况下，你不想是用查询参数值限定路由模式，而是让查询参数值改变的时候路由也跟着改变并且会重新加载数据。这时候你可用在对应的`controller`类中设置`queryParams`哈希对象，在这对象中配置一个参数`scope`为`controller`。如下：
```js
queryParams: [{
    showMagnifyingGlass: {
		scope: 'controller'
	}
}]
```
粘性的查询参数值这个只是点理解起来好难的说，看下一遍下来都不知道这个有何用！！！现在还是学习阶段还没真正在项目中使用这个特性，所以我也不知道怎么解释更容易理解，建议直接看官网教程吧！！

**说明**：本文是基于官方2.0参考文档缩写，相对于其他版本内容会有出入。

以上的内容就是有关查询参数的全部了，主要是理解了查询参数的设置使用起来也就没什么问题。有点遗憾的是没能写出第4点的演示实例！能力有限只能遇到或者明白其使用的时候再补上了！！

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！