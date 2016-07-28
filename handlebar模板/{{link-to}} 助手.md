### 1，link-to助手常规使用
`link-to`助手表达式渲染之后就是一个`a`标签。而`a`标签的`href`属性的值是根据路由生成的，与路由的设置是息息相关的。并且每个设置的路由名称都是有着对应的关系的。
为了演示效果，用命令生成了一个`route`（或者手动创建文件）并获取测试数据。本文结合路由设置，随便串讲了一些路由方面的知识，如果你能看懂最好了，看不懂也不要紧后面会有一整章介绍路由。
#### 1, 增加子路由
```javascript
//  app/routers.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType	
});

Router.map(function() {
  this.route('posts', function() {
  	this.route('detail', {path: '/:post_id'});  //指定子路由，:post_id会自动转换为数据的id
  });
});

export default Router;
```
如上述代码，在`posts`下增加一个子路由`detail`。并且指定路由名为`/:post_id`，`:post_id`是一个动态字段，一般情况下默认为`model`的`id`属性。经过模板渲染之后会得到类似于`posts/1`、`posts/2`这种形式的路由。

#### 2, 在route初始化数据
```javascript
//  app/routes/posts.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		return Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');
	}
});
```
使用Ember提供的方法直接从远程获取测试数据。测试数据的格式可以用浏览器直接打开上面的URL就可以看到。

#### 3，添加显示的模板
```html
<!--  //  app/templates/posts.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-10 col-xs-10">
			<div style="margin-top: 70px;">
				<ul class="list-group">
				{{#each model as |item|}}
					<li class="list-group-item">
						<!--设置路由，路由的层级与router.js里定义的要一致 -->
                     {{#link-to 'posts.detail' item}}
							{{item.title}}
						{{/link-to}}
					</li>
				{{/each}}
				</ul>
			</div>
		</div>
	</div>
</div>
```
直接用`{{#each}}`遍历出所有的数据，并显示在界面上，由于数据比较多可能显示的比较慢，特别是页面刷新之后看到一片空白，请不要急着刷新页面，等待一下即可……。下图是结果的一部分：

![结果截图](/content/images/2016/03/24.png)

我们查看页面的源代码，可以看到`link-to`助手渲染之后的HTML代码，自动生成了URL，在`router.js`里配置的`post_id`渲染之后都被`model`的`id`属性值代替了。

![run result](/content/images/2016/03/25.png)

如果你没有测试数据，你还可直接把`link-to`助手的`post_id`写死，可以直接把数据的`id`值设置到`link-to`助手上。在模板文件的`ul`标签下一行增加如下代码，在`link-to`助手中指定`id`为`1`：
```html
<!-- 增加一条直接指定id的数据 -->
  <li class="list-group-item">
    {{#link-to 'posts.detail' 1}}增加一条直接指定id的数据{{/link-to}}
  </li>
```
渲染之后的HTML代码如下:
```html
<li class="list-group-item">
<a id="ember404" href="/posts/1" class="ember-view">增加一条直接指定id的数据
</a>
</li>
```
可以看到与前面使用动态数据渲染之后的`href`的格式是一样的。如果你想设置某个`a`标签是激活状态，你可以直接在标签上增加一个CSS类（`class=”active”`）。

### 2，link-to助手设置多个动态字段
开发中，路由的路径经常不是2层的（`post/1`）也有可能是多层次的（`post/1/comment`、`post/1/2`或者`post/1/comment/2`等等形式。），如果是这种形式的URL在`link-to`助手上又要怎么去定义呢？
老样子，在演示模板之前还是需要先构建好测试数据以及修改对应的路由设置，此时的路由设置是多层的，因为`link-to`助手渲染之后得到的href属性值就是根据路由生成的！！！这个必须要记得……

#### 1. 一个路由下有个多子路由
```javascript
//  app/routers.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType	
});

Router.map(function() {
  // this.route('handlebarsConditionsExpRoute');
  // this.route('handlebars-each');
  // this.route('store-categories');
  // this.route('binding-element-attributes');

  //  link-to实例理由配置
  // this.route('link-to-helper-example', function() {
  //  this.route('edit', {path: '/:link-to-helper-example_id'});
  // });

  this.route('posts', function() {
      //指定子路由，:post_id会自动转换为数据的id
  	this.route('detail', {path: '/:post_id'}, function() {
      //增加一个comments路由
      this.route('comments');
      // 第二个子路由comment，并且路由是个动态字段comment_id
      this.route('comment', {path: '/:comment_id'});
    });
  });
  
});

export default Router;
```

如果是上述配置，渲染之后得到的路由格式`posts/detail/comments`。由于获取远程数据比较慢直接注释掉`posts.js`里的`model`回调方法，就直接使用写死id的方式。
注意：上述配置中，在路由`detail`下有2个子路由，一个是`comments`，一个是`comment`，并且这个是一个动态段。由此模板渲染之后应该是有2种形式的URL。一种是`posts.detail.comments`（`posts/1/comments`），另一种是`posts.detail.comment`（`posts/1/2`）。如果能理解这个那`route`嵌套层次再多应该也能看明白了！
```html
<!--  //  app/templates/posts.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-10 col-xs-10">
			<div style="margin-top: 70px;">
				<ul class="list-group">
					<!-- 增加一条直接指定id的数据 -->
					<li class="list-group-item">
						<!—- 注意此时只有一个动态段，所以只有一个数字1，Ember会根据顺序自动匹配到动态段的位置上。 -->
						{{#link-to 'posts.detail.comments' 1 class='active'}}
						posts.detail.comments（posts/1/comments）形式
						{{/link-to}}
					</li>


					<li class="list-group-item">
						<!—-
						注意此时有2个动态段，所以有2个数字1，2，Ember会根据顺序自动匹配到动态段的位置上。
						第一个数字1会匹配到第一个动态段post_id上，第二个数字2会匹配到动态段comment_id上
						 -->
						{{#link-to 'posts.detail.comment' 1 2 class='active'}}
						posts.detail.comment（posts/1/2）形式
						{{/link-to}}
					</li>

				</ul>
			</div>
		</div>
	</div>
</div>
```
渲染之后的结果如下：

![run result](/content/images/2016/03/26.png)

如果是动态段的一般都是`model`的`id`代替，如果不是动态段的直接显示配置的路由名称。

#### 2, 多层路由嵌套

上面演示了多个子路由的情况，下面接着介绍一个路由有多个层次，并且是有个多个动态段和非动态段组成的情况。
首先修改路由配置，把`comments`设置为`detail`的子路由。并且在`comments`下在设置一个动态段的子路由`comment_id`。
```javascript
//  app/routers.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType	
});

Router.map(function() {
  this.route('posts', function() {
      //指定子路由，:post_id会自动转换为数据的id
    this.route('detail', {path: '/:post_id'}, function() {
      //增加一个comments路由
      this.route('comments', function() {
        // 在comments下面再增加一个子路由comment，并且路由是个动态字段comment_id
        this.route('comment', {path: '/:comment_id'});
      });   
    });
  });
});
export default Router;
```
模板使用路由的方式`posts.detail.comments.comment`。正常情况应该生成类似`posts/1/comments/2`这种格式的URL。
```html
<!--  //  app/templates/posts.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-10 col-xs-10">
			<div style="margin-top: 70px;">
				<ul class="list-group">

					<li class="list-group-item">
						<!--
							一共设置了4层路由
						 -->
						{{#link-to 'posts.detail.comments.comment' 1 2 class='active'}}
						posts.detail.comments.comment（posts/1/comments/2）形式
						{{/link-to}}
					</li>

				</ul>
			</div>
		</div>
	</div>
</div>
```
渲染之后得到的HTML如下：
```html
<ul class="list-group">
  <li class="list-group-item">
  <!-- 一共设置了4层路由 -->
    <a id="ember473" href="/posts/1/comments/2" class="active ember-view">						posts.detail.comments.comment（posts/1/comments/2）形式
    </a>
  </li> 
</ul>
```
结果正如我们预想的组成了4层路由（`/posts/1/comment/2`）。
补充内容。
对于上述第二点多层路由嵌套的情况你还可以使用下面的方式设置路由和模板，并且可用同时设置了`/posts/1/comments`和`/posts/1/comments/2`。
```javascript
  this.route('posts', function() {
      //指定子路由，:post_id会自动转换为数据的id
      this.route('detail', {path: '/:post_id'}, function() {
          //增加一个comments路由
          this.route('comments');
          // 路由调用：posts.detail.comment
          // 注意区分与前面的设置方式，detai渲染之后会被/:post_id替换，comment渲染之后直接被comments/:comment_id替换了，
          //会得到如posts/1/comments/2这种形式的URL
          this.route('comment', {path: 'comments/:comment_id'});
    });
  });
```
```html
<!--  //  app/templates/posts.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-10 col-xs-10">
			<div style="margin-top: 70px;">
				<ul class="list-group">

					<li class="list-group-item">
						<-- 设置的方式不变 -->
                    {{#link-to 'posts.detail.comments' 1 class='active'}}
						posts.detail.comments（/posts/1/comments）形式
						{{/link-to}}
					</li>

					<li class="list-group-item">
						<!--
							一共设置了4层路由，与前面的设置方式一样
						 -->
						{{#link-to 'posts.detail.comment' 1 2 class='active'}}
						posts.detail.comments.comment（posts/1/comments/2）形式
						{{/link-to}}
					</li>


				</ul>
			</div>
		</div>
	</div>
</div>
```

渲染之后的结果如下：

![渲染之后的结果](/content/images/2016/03/27.png)

两种方式定义的路由渲染的结果是一样的，看个人喜欢了，定义方式也是非常灵活的。第一种定义方式看着比较清晰，看代码就知道是一层层的。但是需要写更多代码。第二种定义方式更加简洁，不过看不出嵌套的层次。
	对于上述route的设置如果还不能理解也不要紧，后面会有一整章是介绍路由的，然而你能结合link-to助手理解了路由设置对于后面route章节的学习是非常有帮助的。

#### 3, 在link-to助手内增加额外属性

`handlebars`允许你直接在`link-to`助手增加额外的属性，经过模板渲染之后`a`标签就有了增加的额外属性了。
比如你可用为`a`标签增加CSS的`class`。

```html
{{link-to "show text info" 'posts.detail' 1 class="btn btn-primary"}}

{{#link-to "posts.detail" 1 class="btn btn-primary"}}show text info{{/link-to}}
```
上述两种写法都是可以的，渲染的结果也是一样的。渲染之后的HTML为：
```html
<a id="ember434" href="/posts/1" class="btn btn-primary ember-view">show text info</a>
```
注意：上述两种方式的写法所设置的参数的位置不能调换。但是官网没有看到这方面的说明，有可能是我的演示实例的问题，如果读者你的可用欢迎给我留言。
第一种方式，显示的文字必须放在最前面，并且中间的参数是路由设置，最有一个参数是额外的属性设置，如果你还要其他的属性需要设置仍然只能放在最后面。
第二章方式的参数位置也是有要求的，第一个参数必须是路由设置，后面的参数设置额外的属性。
对于渲染之后的HTML代码中出现标签`id`为`ember`，或者`ember-xxx`，这些属性都是Ember默认生成的，我们可以暂时不用理它。
综合，本来这篇是讲解`link-to`的，但是由于涉及到了`route`的配置就顺便讲讲，有点难度，主要在路由的嵌套这个知识点上，但是对于后面的`route`这一章的学习是很有帮助的，`route`的设置几乎都是为URL设置的。这两者是紧密关联的！
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！



