在Ember中路由和模板的执行都是有一定顺序的，它们的顺序为：主路由->子路由1->子路由2->子路由3->……。模板渲染的顺序与路由执行顺序刚好相反，从最后一个模板开始解析渲染。

**注意**：模板的渲染是在所有路由执行完之后，从最后一个模板开始。关于这一点下面的代码会演示验证，官网教程有介绍，点击查看。

比如有一路由格式为`application/posts/detail/comments/comment`，此时路由执行的顺序为：`application/posts` -> `detail` -> `comments` -> `comment`，`application`是项目默认的路由，用户自定义的所有路由都是`application`的子路由（默认情况下），相对应的模板也是这样，所有用户自定义的模板都是`application.hbs`的子模板。如果你要修改模板的渲染层次你可以在`route`中重写`renderTemplate`回调函数，在函数内使用`render`方法指定要渲染的模板（如：`render('other')`，渲染到`other`这个模板上）更多有关信息请查看这里。并且它们对应的文件模板结构如下图：

![文件模板结构](/content/images/2016/03/28.png)

路由与模板是相对应的，所以模板的目录结构与路由的目录结构是一致的。
你有两种方式构建上述目录：

1. 手动创建
2. 使用命令，比如创建`comment.js`使用命令：`ember generate route posts/detail/comments/comment`，[Ember CLI](http://ember-cli.com/user-guide/)会自动为我们创建目录和文件。

创建好目录结构之后我们添加一些代码到每个文件。运行项目之后你就会一目了然了……。
下面我按前面讲的路由执行顺序分别列出每个文件的内容。
```javascript
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() { 
		console.log('running in posts...');
		return { id: 1, routeName: 'The route is posts'};
		// return Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');
	}
	
});
```

```javascript
import Ember from 'ember';

export default Ember.Route.extend({

	model: function(params) {
		console.log('params id = ' + params.post_id);
		console.log('running in detail....');

		return { id: 1, routeName: 'The route is detail..' };
	}
});	
```

```javascript
//  app/routes/posts/detail.js

import Ember from 'ember';

export default Ember.Route.extend({

	model: function(params) {
		console.log('params id = ' + params.post_id);
		console.log('running in detail....');

		return { id: 1, routeName: 'The route is detail..' };
	}
});	
```

```javascript
//  app/routes/posts/detail/comments.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		console.log('running in comments...');
		return { id: 1, routName: 'The route is comments....'};
	}
});
```

```javascript
//  app/routes/posts/detail/comments/comment.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {
		console.log('params id = ' + params.post_id);
		console.log('running in comment...');
		return { id: 1, routeName: 'The route is comment...'};
	}
});
```
下面是模板各个文件的内容。其列出才顺序与路由的顺序一致。
```html
<!--  //  app/templates/posts.hbs  -->

<!-- 所有的子路由对应的模板都会渲染到outlet上 -->
{{model.routeName}} >> {{outlet}}
```

```html
<!-- // app/templates/posts/detail.hbs -->

<!-- 所有的子路由对应的模板都会渲染到outlet上 -->
{{model.routeName}} >> {{outlet}}
```

```html
<!-- // app/templates/posts/detail/comments.hbs -->

<!-- 所有的子路由对应的模板都会渲染到outlet上 -->
{{model.routeName}} >> {{outlet}}
```
```html
<!-- // app/templates/posts/detail/comments/comment.hbs -->

<!-- 所有的子路由对应的模板都会渲染到outlet上 -->
{{model.routeName}} >> {{outlet}}
```

下图是路由执行的顺序，并且在执行的过程中渲染路由对应的模板。

![路由执行的顺序](/content/images/2016/03/29.png)

从上图中可用清楚的看到当你运行一个URL时，与URL相关的路由是怎么执行的。

1. 执行主路由（默认是`application`），此时进入到路由的`model`回调方法，并且返回了一个对象`{ id: 1, routeName: 'The route is application...' }`，执行完回调之后继续转到子路由执行直到最后一个路由执行完毕，所有的路由执行完毕之后开始渲染页面。
2. 页面的渲染则是从最后一个路由对应的模板开始，并沿着最近的父模板往回渲染。
为了验证是否是这样的执行顺序，下面修改`detail.js`和
`comments.js`。在代码中加入一个模拟休眠的操作。
```javascript
//  app/routes/posts/detail.js

import Ember from 'ember';

export default Ember.Route.extend({

	model: function(params) {
		console.log('params id = ' + params.post_id);
		console.log('running in detail....');

		//  执行一个循环，模拟休眠
		for (var i = 0; i < 10000000000; i++) {
			
		}
		console.log('The comment route executed...');


		return { id: 1, routeName: 'The route is detail..' };
	}
});	
```
```javascript
//  app/routes/posts/detail/comments.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {
		console.log('params id = ' + params.post_id);
		console.log('running in comment...'); 
		//  执行一个循环，模拟休眠
		for (var i = 0; i < 10000000000; i++) {
			
		}

		return { id: 1, routeName: 'The route is comment...'};
	}
});
```
刷新页面，注意查看控制台输出信息和页面显示的内容。
新开一个窗口，执行URL：[http://localhost:4200/posts/2/comments](http://localhost:4200/posts/2/comments)。

![run result](/content/images/2016/03/30.png)

控制台输出到这里时处理等待（执行`for`循环），此时已经执行了两个路由`application`和`posts`，并且正在执行`detail`，但是页面是空白的，没有任何HTML元素。

![run result](/content/images/2016/03/31.png)

在`detail`路由执行完成之后转到路由`comments`。然后执行到`for`循环模拟休眠，此时页面仍然是没有任何HTML元素。然后等到所有`route`执行完毕之后，界面才显示`model`回调里设置的信息。

![run result](/content/images/2016/03/33.png)

![run result](/content/images/2016/03/32.png)

每个子路由设置的信息都会渲染到最近一个父路由对应模板的`{{outlet}}`上面。

![run result](/content/images/2016/03/34.png)

1. 渲染`comment`
得到的内如为：“`comment`渲染完成”
2. 渲染`comment`最近的父模板`comments`
得到的内容为：“`comment`渲染完成 `comments`渲染完成”
3. 渲染`comments`最近的父模板`detail`
得到的内容为：“`comment`渲染完成 `comments`渲染完成 `detail`渲染完成”
4. 渲染`detail`最近的父模板`posts`
得到的内容为：“`comment`渲染完成 `comments`渲染完成 `detail`渲染完成 `posts`渲染完成”
5. 渲染`posts`最近的父模板`application`
得到的内容为：“`comment`渲染完成 `comments`渲染完成 `detail`渲染完成 `posts`渲染完成 `application`渲染完成”

只要记住一句话：**子模板的都会渲染到父模板的`{{outlet}}`上，最终所有的模板都会被渲染到`application`的`{{outlet}}`上。**

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
