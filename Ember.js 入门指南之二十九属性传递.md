## 1，传递参数到组件上	

每个组件都是相对独立的，因此任何组件所需的数据都需要通过组件的属性把数据传递到组件中。

比如上篇[Ember.js 入门指南之二十八组件定义](httpblog.ddlisting.com20160407ember-js-ru-men-zhi-nan-zhi-er-shi-ba-zu-jian-ding-yi)的第三点`{{component item.pn post=item}}`就是通过属性post把数据传递到组件`foo-component`或者`bar-component`上。如果在`index.hbs`中是如下方式调用组件那么渲染之后的页面是空的。
`{{component item.pn}}`
请读者自己修改`index.hbs`的代码后演示效果。

传递到组件的参数也是动态更新的，当传递到组件上的参数变化时组件渲染的HTML也会随之发生改变。

## 2，位置参数 

传递的属性参数不一定要指定参数的名字。你可以不指定属性参数的名称，然后根据参数的位置获取对应的值，但是要在组件对应的组件类中指定位置参数的名称。比如下面的代码：

准备工作：
```
ember g route passing-properties-to-component
ember g component passing-properties-to-component
```
调用组件的模板，传入两个位置参数，分别是`item.title`、`item.body`。
```html
!-- apptemplatespassing-properties-to-component.hbs  --

{{#each model as item}}
	!-- 传递到组件blog-post第一个参数为数据的title值，第二个为body值 --
	{{passing-properties-to-component item.title item.body}}
{{each}}
```

准备需要显示的数据。
```js
  approutespadding-properties-to-component.js

import Ember from 'ember';

export default Ember.Route.extend({

	model function() {
		 return [
	    	{ id 1, title 'Bower dependencies and resolutions new', body In the bower.json file, I see 2 keys dependencies and resolutionsWhy is that so  },
	    	{ id 2, title 'Highly Nested JSON Payload - hasMany error', body Welcome to the Ember.js discussion forum. We're running on the open source, Ember.js-powered Discourse forum software.  },
	    	{ id 3, title 'Passing a jwt to my REST adapter new ', body This sets up a binding between the category query param in the URL, and the category property on controllerarticles.  }
	    ];
	   
	}
});
```
在组件类中指定位置参数的名称。
```js
  appcomponentspadding-properties-to-component.js

import Ember from 'ember';

export default Ember.Component.extend({
	 指定位置参数的名称
	positionalParams ['title', 'body']
});
```
注意：属性positionalParams指定的参数不能在运行期改变。

组件直接使用组件类中指定的位置参数名称获取数据。
```html
!--  apptemplatescomponentspassing-properties-to-component.hbs  --

article
	h1{{title}}h1
	p{{body}}p
article
```
注意：获取数据的名称必须要与组件类指定的名称一致，否则无法正确获取数据。
显示结果如下：

![结果](contentimages201604121.png)

[Ember](httpemberjs.com)还允许你指定任意多个参数，但是组件类获取参数的方式就需要做点小修改。比如下面的例子：

调用组件的模板
```html
!-- apptemplatespassing-properties-to-component.hbs  --

{{#each model as item}}
	!-- 传递到组件blog-post第一个参数为数据的title值，第二个为body值 --
	{{passing-properties-to-component item.title item.body 'third value' 'fourth value'}}
{{each}}
```
指定参数名称的组件类，获取参数的方式可以[Ember.js 入门指南之三计算属性](httpblog.ddlisting.com20160317ember-js-ru-men-zhi-nan-ji-suan-shu-xing-compute-properties)这章。
```js
  appcomponentspadding-properties-to-component.js

import Ember from 'ember';

export default Ember.Component.extend({
	 指定位置参数为参数数组
	positionalParams 'params',
	title Ember.computed('params.[]', function() {
		return this.get('params')[0];  获取第一个参数
	}),
	body Ember.computed('params.[]', function() {
		return this.get('params')[1];  获取第二个参数
	}),
	third Ember.computed('params.[]', function() {
		return this.get('params')[2];  获取第三个参数
	}),
	fourth Ember.computed('params.[]', function() {
		return this.get('params')[3];  获取第四个参数
	})
});
```
下面看组件是怎么获取传递过来的参数的。
```html
!--  apptemplatescomponentspassing-properties-to-component.hbs  --

article
	h1{{title}}h1
	p{{body}}p
	pthird {{third}}p
	pfourth {{fourth}}p
article
````
显示结果如下：

![结果截图](contentimages201604122-1.png)

到此组件参数传递的内容全部介绍完毕。总的来说没啥难度。Ember中参数的传递与获取方式基本是相似的，比如[link-to助手](httpblog.ddlisting.com20160322ember-js-ru-men-zhi-nan-zhi-shi-san-link-to)、[action助手](httpblog.ddlisting.com20160322ember-js-ru-men-zhi-nan-zhi-shi-wu-action)。

br
博文完整代码放在[Github](httpsgithub.comubuntuvimmy_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
