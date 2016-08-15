前一篇介绍了查询方法，本篇介绍新建、更新、删除记录的方法。
本篇的示例代码创建在上一篇的基础上。对于整合[firebase](http://www.firebase.com)、创建`route`和`template`请参看上一篇，增加一个controller：`ember g controller articles`。

## 1，新建记录

创建新的记录使用`createRecord()`方法。比如下面的代码新建了一个`aritcle`记录。修改模板，在模板上增加几个`input`输入框用于输入`article`信息。
```html
<!--  app/templates/articles.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-4 col-xs-4">
			<ul class="list-group">
			{{#each model as |item|}}
				<li class="list-group-item">
					<!--设置路由，路由的层级与router.js里定义的要一致 -->
                 	{{#link-to 'articles.article' item.id}}
						{{item.title}} -- <small>{{item.category}}</small>
					{{/link-to}}
				</li>
			{{/each}}
			</ul>


			<div>
				title：{{input value=title}}<br>
				body： {{textarea value=body cols="80" rows="3"}}<br>
				category: {{input value=category}}<br>
				<button {{ action "saveItem"}}>保存</button>
				<font color='red'>{{tipInfo}}</font>
			</div>
		</div>

		<div class="col-md-8 col-xs-8">
		{{outlet}}
		</div>
	</div>
</div>
```
页面的字段分别对应这模型`article`的属性。点击“保存”后提交到`controller`处理。下面是获取数据保存数据的`controller`。
```js
//   app/controllers/articles.js

import Ember from 'ember';

export default Ember.Controller.extend({

	actions: {

		//  表单提交，保存数据到Store。Store会自动更新到firebase
		saveItem: function() {
			var title = this.get('title');
			if ('undefined' === typeof(title) || '' === title.trim()) {
				this.set('tipInfo', "title不能为空");
				return ;
			}

			var body = this.get('body');
			if ('undefined' === typeof(body) || '' === body.trim()) {
				this.set('tipInfo', "body不能为空");
				return ;
			}
			var category = this.get('category');
			if ('undefined' === typeof(category) || '' === category.trim()) {
				this.set('tipInfo', "category不能为空");
				return ;
			}

			//  创建数据记录
			var article = this.store.createRecord('article', {
				title: title,
				body: body,
				category: category,
				timestamp: new Date().getTime()
			});
			article.save();  //保存数据的到Store
			//  清空页面的input输入框
			this.set('title', "");
			this.set('body', "");
			this.set('category', "");
		}
	}
});
```
主要看`createRecord`方法，第一个参数是模型名称。第二个参数是个哈希，在哈希总设置模型属性值。最后调用`article.save()`方法把数据保存到`Store`，再由`Store`保存到firebase。运行效果如下图：

![结果截图](/content/images/2016/04/170.png)

输入信息，点击“保存”后数据立刻会显示在列表”no form -- java”之后。然后你可以点击标题查询详细信息，body的信息会在页面后侧显示。

通过这里实例我想你应该懂得去使用`createRecord()`方法了！但是如果有两个模型是有关联关系保存的方法又是怎么样的呢？下面再新增一个模型。
```
ember g model users
```
然后在模型中增加关联。
```js
//   app/models/article.js

import DS from 'ember-data';

export default DS.Model.extend({
	title: DS.attr('string'),
	body: DS.attr('string'),
	timestamp: DS.attr('number'),
	category: DS.attr('string'),
	author: DS.belongsTo('user')  //关联user
});


//  app/models/user.js
import DS from 'ember-data';

export default DS.Model.extend({
  	username: DS.attr('string'),
  	timestamp: DS.attr('number'),
  	articles: DS.hasMany('article')  //关联article
});
```
修改模板`articles.hbs`在界面上增加录入作者信息字段。
```html
……省略其他代码
<div>
	title：{{input value=title}}<br>
	body： {{textarea value=body cols="80" rows="3"}}<br>
	category: {{input value=category}}<br>
	<br>
	author: {{input value=username}}<br>
	<button {{ action "saveItem"}}>保存</button>
	<font color='red'>{{tipInfo}}</font>
</div>
……省略其他代码
```
下面看看怎么在`controller`中设置这两个模型的关联关系。一共有两种方式设置，一种是直接在`createRecord()`方法中设置，另一种是在方法外设置。
```js
//   app/controllers/articles.js

import Ember from 'ember';

export default Ember.Controller.extend({

	actions: {

		//  表单提交，保存数据到Store。Store会自动更新到firebase
		saveItem: function() {
			//  获取信息和校验代码省略……

			// 创建user
			var user = this.store.createRecord('user', {
				username: username,
				timestamp: new Date().getTime()
			});
			//  必须要执行这句代码，否则user数据不能保存到Store，
			//  否则article通过user的id查找不到user
			user.save();

			//  创建article
			var article = this.store.createRecord('article', {
				title: title,
				body: body,
				category: category,
				timestamp: new Date().getTime(),
				author: user   //设置关联
			});

			article.save();  //保存数据的到Store
			//  清空页面的input输入框
			this.set('title', "");
			this.set('body', "");
			this.set('category', "");
			this.set('username', "");
		}
	}
});
```

![界面截图](/content/images/2016/04/171.png)

输入上如所示信息，点击“保存”可以在firebase的后台看到如下的数据关联关系。

![firebase数据截图](/content/images/2016/04/172.png)

注意点：与这两个数据的关联是通过数据的`id`维护的。
那么如果我要通过`article`获取`user`的信息要怎么获取呢？

直接以面向对象的方式获取既可。
```html
{{#each model as |item|}}
	<li class="list-group-item">
		<!--设置路由，路由的层级与router.js里定义的要一致 -->
     	{{#link-to 'articles.article' item.id}}
			{{item.title}} -- <small>{{item.category}}</small> -- <small>{{item.author.username}}</small>
		{{/link-to}}
	</li>
{{/each}}
```
注意看助手`{{ item.author.username }}`。很像EL表达式吧！！
前面提到过有两个方式设置两个模型的关联关系。下面的代码是第二种方式：
```js
//  其他代码省略……
//  创建article
var article = this.store.createRecord('article', {
	title: title,
	body: body,
	category: category,
	timestamp: new Date().getTime()
	// ,
	// author: user   //设置关联
});

// 第二种设置关联关系方法，在外部手动调用set方法设置
article.set('author', user);
//  其他代码省略……
```
运行，重新录入信息，得到的结果是一致的。甚至你可以直接在`createRecord`方法里调用方法来设置两个模型的关系。比如下面的代码段：
```js
var store = this.store;  // 由于作用域问题，在createRecord方法内部不能使用this.store
var article = this.store.createRecord('article', {
	title: title,
	// ……
	// ,
	// author: store.findRecord('user', 1)   //设置关联
});

// 第二种设置关联关系方法，在外部手动调用set方法设置
article.set('author', store.findRecord('user', 1));
```
这种方式可以直接动态根据user的id属性值获取到记录，再设置关联关系。新增介绍完了，接着介绍记录的更新。

##2，更新记录

更新相对于新增来说非常相似。请看下面的代码段：
首先在模板上增加更新的设置代码，修改子模板`articles/article.hbs`：
```html
<!--  app/templates/articles/article.hbs  -->

<h1>{{model.title}}</h1>
<div class = "body">
{{model.body}}
</div>

<div>
<br><hr>
更新测试<br>
title: {{input value=model.title}}<br>
body：<br> {{textarea value=model.body cols="80" rows="3"}}<br>
<button {{action 'updateArticleById' model.id}}>更新文章信息</button>
</div>
```
增加一个`controller`，用户处理子模板提交的修改信息。
```
ember g controller articles/article
```
```js
//  app/controllers/articles/article.js

import Ember from 'ember';

export default Ember.Controller.extend({
	actions: {
		// 根据文章id更新
		updateArticleById: function(params) {
			var title = this.get('model.title');
			var body = this.get('model.body');
			this.store.findRecord('article', params).then(function(art) {
				art.set('title', title);
				art.set('body', body);

				//  保存更新的值到Store
				art.save();
			});
		}
	}
});
```
在左侧选择需要更新的数据，然后在右侧输入框中修改需要更新的数据，在修改过程中可以看到被修改的信息会立即反应到界面上，这个是因为Ember自动更新Store中的数据（还记得很久前讲过的观察者（observer）吗？）。

![结果截图](/content/images/2016/04/173-1.png)

如果你没有点击“更新文章信息”提交，你修改的信息不会更新到firebase。页面刷新后还是原来样子，如果你点击了“更新文章信息”数据将会把更新的信息提交到firebase。

由于`save`、`findRecord`方法返回值是一个`promises`对象，所以你还可以针对出错情况进行处理。比如下面的代码：
```js
var user = this.store.createRecord('user', {
	//  ……
});
user.save().then(function(fulfill) {
	//  保存成功
}).catch(function(error) {
	//  保存失败
});

this.store.findRecord('article', params).then(function(art) {
	//  ……
}).catch(function(error) {
	//  出错处理代码
});
```
具体代码我就不演示了，请读者自己编写测试吧！！

##3，删除记录

既然有了新增那么通常就会有删除。记录的删除与修改非常类似，也是首先查询出要删除的数据，然后执行删除。
```js
//   app/controllers/articles.js

import Ember from 'ember';

export default Ember.Controller.extend({

	actions: {

		//  表单提交，保存数据到Store。Store会自动更新到firebase
		saveItem: function() {
			// 省略
		},
		//  根据id属性值删除数据
		delById : function(params) {

	    	//  任意获取一个作为判断表单输入值
	        if (params && confirm("你确定要删除这条数据吗？?")) {
	            //  执行删除
	            this.store.findRecord('article', params).then(function(art) {
	            	art.destroyRecord();
	            	alert('删除成功！');
	            }, function(error) {
	            	alert('删除失败！');
	            });
	        } else {
	            return;
	        }

		}
	}
});
```
修改显示数据的模板，增加删除按钮，并传递数据的`id`值到`controller`。
```html
<!--  app/templates/articles.hbs  -->

<div class="container">
	<div class="row">
		<div class="col-md-4 col-xs-4">
			<ul class="list-group">
			{{#each model as |item|}}
				<li class="list-group-item">
					<!--设置路由，路由的层级与router.js里定义的要一致 -->
                 	{{#link-to 'articles.article' item.id}}
						{{item.title}} -- <small>{{item.category}}</small> -- <small>{{item.author.username}}</small>
					{{/link-to}}
					<button {{action 'delById' item.id}}>删除</button>
				</li>
			{{/each}}
			</ul>
        // ……省略其他代码
	</div>
</div>
```

![截图](/content/images/2016/04/174.png)

结果如上图，点击第二条数据删除按钮。弹出提示窗口，点击“确定”之后成功删除数据，并弹出“删除成功！”，到firebase后台查看数据，确实已经删除成功。
然而与此关联的user却没有删除，正常情况下也应该是不删除关联的user数据的。
最终结果只剩下一条数据：

![截图](/content/images/2016/04/175.png)

到此，有关新增、更新、删除的方法介绍完毕。已经给出了详细的演示实例，我相信，如果你也亲自在自己的项目中实践过，那么掌握这几个方法是很容易的！
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
