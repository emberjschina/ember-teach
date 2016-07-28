不得不说，[Ember](http://emberjs.com)的更新是在是太快了！！本教程还没写到一半就又更新到`v2.1.0`了！！！！不过为了统一还是使用官方`v2.0.0`的参考文档！！

从本篇开始进入新的一章——组件。这一章将用6篇文章介绍Ember的组件，从它的定义开始知道它的使用方式，我将为你一一解答！
	
**准备工作**：
	本章代码统一访问项目chapter4_components下，项目代码可以在以下网址上找到：
[https://github.com/ubuntuvim/my_emberjs_code](https://github.com/ubuntuvim/my_emberjs_code)


与之前的文章一样，项目仍然是使用[Ember CLI](http://www.ember-cli.com/user-guide/)命令创建项目和各个组件文件。

创建项目并测试运行，首先执行如下四条命令，最后在浏览器执行：[http://localhost:4200/](http://localhost:4200/)。
```
ember new chapter4_components
cd chapter4_components
ember server
```
如果你能在页面上看到**Welcome to Ember**说明项目框架搭建成功！那么你可以继续往下看了，否则想搭建好项目再往下学习~~~

## 1，自定义组件及使用

创建组件方法很简单：`ember generate component my-component-name`。一条命令即可，但是需要注意的是组件的名称必须要包含中划线`-`，比如`blog-post`、`test-component`、`audio-player-controls`这种格式的命名是合法，但是`post`、`test`这种方式的命名是不合法的！其一是为了防止用户自定义的组件名与W3C规定的元素标签名重复；其二是为了确保[Ember](http://emberjs.com)能自动检测到用户自定义的组件。

下面定义一个组件，`ember g component blog-post`。[Ember CLI](http://www.ember-cli.com/user-guide/)会自动为你创建组件对应的的模板，执行这条命令之后你可以在`app/components`和`app/templates/components`下看到创建的文件。
```html
<!--  app/templates/components/blog-post.hbs  -->

<article class="blog-post">
	<h1>{{title}}</h1>
	<p>{{yield}}</p>
	<p>Edit title: {{input type="text" value=title}}</p>
</article>
```
为了演示组件的使用需要做些准备工作：
`ember g route index`
```js
//  app/routes/index.js

import Ember from 'ember';

export default Ember.Route.extend({
	
	model: function() {
		 return [
	    	{ id: 1, title: 'Bower: dependencies and resolutions new', body: "In the bower.json file, I see 2 keys dependencies and resolutionsWhy is that so? I understand Bower has a flat dependency structure. So has it got anything to do with that ?", category: 'java' },
	    	{ id: 2, title: 'Highly Nested JSON Payload - hasMany error', body: "Welcome to the Ember.js discussion forum. We're running on the open source, Ember.js-powered Discourse forum software. They are also providing the hosting for us. Thanks guys! Please use this space for discussion abo… read more", category: 'php' },
	    	{ id: 3, title: 'Passing a jwt to my REST adapter new ', body: "This sets up a binding between the category query param in the URL, and the category property on controller:articles. In other words, once the articles route has been entered, any changes to the category query param in the URL will update the category property on controller:articles, and vice versa.", category: 'java'}
	    ];
	   
	}
});
```
```html
<!-- app/templates/index.hbs  -->

{{#each model as |item|}}
	<!-- 使用自定义的组件blog-post -->
	{{#blog-post title=item.title}}
		{{item.body}}
	{{/blog-post}}
{{/each}}
```
在这段代码中，使用了自定义的组件来显示数据。最后页面显示如下：

![运行结果截图](/content/images/2016/04/99.png)

![渲染后的HTML代码](/content/images/2016/04/100-1.png)

自定义的组件被渲染到了模板`index.hbs`使用`blog-post`的地方。并且自定义组件的HTML标签没有变化。
到这里大概应该知道怎么去使用组件了，至于它是怎么就渲染到了使用组件的地方，以及它是怎么渲染上去的。别急~~后面的文章会为你一一解答。

**说明**：默认情况下，自定义的组件会被渲染到`div`标签内，当然这种默认情况也是可以修改的，比较简单在此不过多介绍，请自行学习，网址：[customizing-a-components-element](http://guides.emberjs.com/v2.0.0/components/customizing-a-components-element/)。

### 2，自定义组件类

用户自定义的组件类都需要继承`Ember.Component`类。

通常情况下我们会把经常使用的模板片段封装成组件，只需要定义一次就可以在项目任何一个模板中使用，而且不需要编写任何的javascript代码。比如上述第一点“自定义组件及使用”中描述的一样。

但是如果你想你的组件有特殊的行为，并且这些行为是默认组件类无法提供的（比如：改变包裹组件的标签、响应组件模板初始化某个状态等），那么此时你可以自定义组件类，但是要继承`Ember.Component`，如果你自定义的组件类没有继承这个类，你自定义的组件就很有可能会出现一些不可预知的问题。

[Ember](http://emberjs.com)所能识别的自定义组件类的名称是有规范的。比如，你定义了一个名为`blog-post`的组件，那么你的组件类的名称应该是`app/components/blog-post.js`。如果组件名为`audio-player-controls`那么对应的组件类名为`app/components/audio-player-controls.js`。即：组件类名与组件同名，这个是`v2.0`的命名方法，请区别就版本的[Ember](http://emberjs.com)，旧版本的组件命名规则是驼峰式的命名规则。

举个简单的例子，在第一点“自定义组件及使用”中讲过，组件默认会被渲染到`div`标签内，你可以在组件类中修改这个默认标签。
```js
//  app/components/blog-post.js

import Ember from 'ember';

export default Ember.Component.extend({
	tagName: 'nav'
});
```
这段代码修改了包裹组件的标签名，页面刷新后HTML代码如下：

![渲染后的HTML代码](/content/images/2016/04/120.png)

可以看到组件的HTML代码被包含在`nav`标签内。

## 3，动态渲染组件

组件的动态渲染与Java的多态有点相似。`{{component}}`助手会延迟到运行时才决定使用那个组件渲染页面。当程序需要根据数据不同渲染不同组件的时，这种动态渲染就显得特别有用。可以使你的逻辑和试图分离开。

那么要怎么使用呢？非常简单，只需要把组件名作为参数传递过去即可，比如：使用`{{component 'blog-post'}}`与`{{blog-post}}`结果是一致的。我们可以修改第一点“自定义组件及使用”实例中模板`index.hbs`的代码。
```html
<!-- app/templates/index.hbs  -->

{{#each model as |item|}}
	<!-- 使用自定义组件的另一种方式（动态渲染组件方式） -->
	{{component 'blog-post' title=item.title}}
	{{item.body}}
{{/each}}
```
页面刷新之后，可以看到结果是一样的。


下面为读者演示如何根据数据不同渲染不同的组件。

按照惯例，先做好准备工作，使用Ember CLI命令创建2个不同的组件。
```
ember g component foo-component
ember g component bar-component
```
```html
<!-- app/templates/components/bar-component.hbs -->

<h1>Hello from bar</h1>
<p>{{post.body}}</p>
```
为何能用`post`获取数据，因为在使用组件的地方传递了参数。在模板`index.hbs`中可以看到。
```html
<!-- app/templates/components/foo-component.hbs -->

<h1>Hello from foo</h1>
<p>{{post.body}}</p>
```
修改显示的数据，注意数据的最后增加一个属性`pn`，`pn`的值就是组件的名称。
```js
//  app/routes/index.js

import Ember from 'ember';

export default Ember.Route.extend({

	model: function() {
		 return [
	    	{ id: 1, title: 'Bower: dependencies and resolutions new', body: "In the bower.json file, I see 2 keys dependencies and resolutionsWhy is that so? I understand Bower has a flat dependency structure. So has it got anything to do with that ?", pn: 'bar-component' },
	    	{ id: 2, title: 'Highly Nested JSON Payload - hasMany error', body: "Welcome to the Ember.js discussion forum. We're running on the open source, Ember.js-powered Discourse forum software. They are also providing the hosting for us. Thanks guys! Please use this space for discussion abo… read more", pn: 'foo-component' },
	    	{ id: 3, title: 'Passing a jwt to my REST adapter new ', body: "This sets up a binding between the category query param in the URL, and the category property on controller:articles. In other words, once the articles route has been entered, any changes to the category query param in the URL will update the category property on controller:articles, and vice versa.", pn: 'bar-component'}
	    ];
	   
	}
});
```
修改调用组件的模板`index.hbs`。
```html
<!-- app/templates/index.hbs  -->

{{#each model as |item|}}
	<!-- 根据组件名渲染不同的组件，第一个参数是组件名，第二个参数为传递到组件上显示的数据 -->
	{{component item.pn post=item}}
{{/each}}
```
模板编译之后会得到形如`{{component foo-component post}}`的组件调用代码。

相信你应该了解了动态渲染组件是怎么回事了！自己动手试试吧~~

到此组件的定义与使用介绍完毕了，不知道你有没有学会呢？如果你有疑问请给我留言或者直接看官方教程学习。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对
我来说是最大的动力！！