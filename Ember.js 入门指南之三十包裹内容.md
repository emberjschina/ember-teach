准备工作：
```
ember g route wrapping-content-in-component-route
ember g component wrapping-content-in-component	
```
有些情况下，你需要定义一个包裹其他模板提供的数据的组件。比如下面的例子：
```html
<!--  app/templates/components/wrapping-content-in-component.hbs  -->

<h1>{{title}}</h1>
<div class='body'>{{body}}</div>
```
上述代码定义了一个普通的组件。
```html
<!--  app/templates/wrapping-content-in-component-route.hbs  -->

{{wrapping-content-in-component title=model.title body=model.body}}
```
调用组件，传入两个指定名称的参数，更多有关组件参数传递问题请看[Ember.js 入门指南之二十九属性传递](http://blog.ddlisting.com/2016/04/07/ember-js-ru-men-zhi-nan-zhi-er-shi-jiu-shu-xing-chuan-di/)。

下面在`route`中增加一些测试数据。
```js
//  app/routes/wrapping-content-in-component-route.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		return { id: 1, title: 'test title', body: 'this is body ...', author: 'ubuntuvim' };
	}
});
```
如果程序代码没写错，界面应该会显示如下信息。

![结果截图](/content/images/2016/04/123.png)

在上述例子中组件正常显示出`model`回调中初始化的数据。但是如果你定义的组件需要包含自定义的HTML内容呢？？

除了上述这种简单的数据传递之外，[Ember](http://emberjs.com)还支持使用`block form`（块方式），换句话说你可以直接传递一个模板到组件中，并在组件中使用`{{yield}}`助手显示传入进来的模板。

为了能使用块方式传递模板到组件中，在调用组件的时候必须使用`#`开始的方式（两种调用方式：`{{component-name}}`或者`{{#component-name}}……{{/component-name}}`），注意一定要有关闭标签！

稍加改造前面的例子，这时候不只是传递一个简单的数据，而是传入一个包含HTML标签的简单模板。
```html
<!--  app/templates/components/wrapping-content-in-component.hbs  -->

<h1>{{title}}</h1>
<!--  模板编译渲染之后{{yield}}助手会被组件标签wrapping-content-in-component包含的内容替换掉 -->
<div class='body'>{{yield}}</div>
```
注意此时`div`标签内使用的是`{{yield}}`助手，而不是直接使用`{{body}}`。

下面是调用组件的模板。
```html
<!--  app/templates/wrapping-content-in-component-route.hbs  -->

{{!wrapping-content-in-component title=model.title body=model.body}}
<!--  调用组件的方式必须是以标签的形式，
	模板编译渲染之后small标签和{{body}}这两行的内容会渲染到组件wrapping-content-in-component的{{yield}}助手上  -->
{{#wrapping-content-in-component title=model.title}}
	{{model.body}}
	<small>by {{model.author}}</small>
{{/wrapping-content-in-component}}
```
页面加载之后效果如下：

![页面效果](/content/images/2016/04/125.png)


查看页面HTML源代码，可以看到在`<div class="body">`这个标签内的内容确实是调用组件`wrapping-content-in-component`传入进来的简单HTML模板。你可以把`{{#wrapping-content-in-component}}……{{/wrapping-content-in-component}}`中间的内容当做是一个参数理解。

![页面效果](/content/images/2016/04/125-1.png)

到此组件包裹内容的知识点介绍完毕，内容很少也比较简单！如果有疑问请给我留言或者直接看[官方教程](guides.emberjs.com/v2.0.0/components/wrapping-content-in-a-component/)。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
