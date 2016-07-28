路由的另一个重要职责是渲染同名字的模板。

比如下面的路由设置，`posts`路由渲染模板`posts.hbs`，路由`new`渲染模板`posts/new.hbs`。
```javascript
Router.map(function() {
     this.route('posts', function() {
     this.route('new');
  });
});
```
每一个模板都会渲染到父模板的`{{outlet}}`上。比如上面的路由设置模板`posts.hbs`会渲染到模板`application.hbs`的`{{outlet}}`上。`application.hbs`是所有自定义模板的父模板。模板posts/new.hbs会渲染到模板`posts.hbs`的`{{outlet}}`上。

如果你想渲染到另外一个模板上也是允许的，但是要在路由中重写`renderTemplate`回调方法。
```javascript
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({
	//  渲染到favorites模板上
	renderTemplate: function() {
	    this.render('favorites');
	}
});
```
模板的渲染这个知识点比较简单，内容也很少，在前面的[Ember.js 入门指南之十四番外篇，路由、模板的执行、渲染顺序](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-si-fan-wai-pian-lu-you-mo-ban-de-zhi-xing-xuan-ran-shun-xu/)已经介绍过相关的内容。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
