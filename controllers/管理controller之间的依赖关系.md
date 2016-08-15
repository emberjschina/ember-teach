在有路由嵌套的情况下，你可能需要在两个不同的`controller`之间通信。
按照惯例先做准备工作：
```
ember g route post
ember g route post/comments
ember g model post
```
比如下面的路由设置：
```js
//  router.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('blog-post');
  this.route('post', { path: '/posts/:post_id' }, function() {
    this.route('comments');
  });
});

export default Router;
```
对于这个路由配置生成的路由表请看[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)。
	
如果用户访问`/posts/1/comments`。模型`post`就会加载到`postController`，并不会直接加载到`commentsController`。然后如果你想在一篇`post`中显示`comment`信息呢？

为了实现这个功能，可以把`postController`注入到`commentController`中。
```js
//  app/controllers/comments.js

import Ember from 'ember';

export default Ember.Controller.extend({
	postController: Ember.inject.controller('post')
});
```
一旦`comments`路由被访问，`postController`就会获取控制器对应的`model`，并且这个`model`是只读的。为了能获取到模型`post`还需要增加一个引用`postController.model`。
```js
//  app/controllers/comments.js

import Ember from 'ember';

export default Ember.Controller.extend({
	postController: Ember.inject.controller('post'),
	post: Ember.computed.reads('postController.model')
});
```
最后可以直接在`comment`模板中显示模型`post`和`comment`的信息。
```html
<h1>Comments for {{post.title}}</h1>
<ul>
	{{#each model as |comment|}}
		<li>{{comment.text}}</li>
	{{/each}}
</ul>
```
有关更多别名的介绍请移步这里查看[API文档](emberjs.com/api/#method_computed_alias)的介绍。如果你想了解更多关于注入的问题请看[这里](emberjs.com/api/#method_computed_alias)的教程（新版官网已经没有这个地址的文档了）。

`controller`这章的内容到此也全部介绍完毕了，只有寥寥的2篇教程，可见`controller`在[Ember](http://emberjs.com)未来版本会被组件替代已成必然。

那么下一章将为大伙介绍模型，模型对于[Ember](http://emberjs.com)来说是一块非常重要的内容，内容也比较多！我回用9篇文章来给你介绍模型，从定义到其使用等等内容。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！


