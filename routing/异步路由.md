本文将为你介绍路由的高级特性，这些高级特性可以用于处理项目复杂的异步逻辑。

>关于单词promises，直译是承诺，但是个人觉得还是使用原文吧。读起来顺畅点。

### 1，promises（承诺）

[Ember](http://emberjs.com/)的路由处理异步逻辑的方式是使用[Promise](http://liubin.org/promises-book/)。简而言之，[Promise](http://liubin.org/promises-book/)就是一个表示最终结果的对象。这个对象可能是`fulfill`（成功获取最终结果）也可能是`reject`（获取结果失败）。为了获取这个最终值，或者是处理[Promise](http://liubin.org/promises-book/)失败的情况都可以使用`then`方法，这个方法接受两个可选的回调方法，一个是[Promise](http://liubin.org/promises-book/)获取结果成功时执行，一个是[Promise](http://liubin.org/promises-book/)获取结果失败时执行。如果promises获取结果成功那么获取到的结果将作为成功时执行的回调方法的参数。相反的，如果[Promise](http://liubin.org/promises-book/)获取结果失败，那么最终结果（失败的原因）将作为[Promise](http://liubin.org/promises-book/)失败时执行的回调方法的参数。比如下面的代码段，当[Promise](http://liubin.org/promises-book/)获取结果成功时执行`fulfill`回调，否则执行`reject`回调方法。
```js
//  app/routes/promises.js

import Ember from 'ember';

export default Ember.Route.extend({
	beforeModel: function() {
		console.log('execute model()');

		var promise = this.fetchTheAnswer();
		promise.then(this.fulfill, this.reject);
	},

	//  promises获取结果成功时执行
	fulfill: function(answer) {
	  console.log("The answer is " + answer);
	},

	//  promises获取结果失败时执行
	reject: function(reason) {
	  console.log("Couldn't get the answer! Reason: " + reason);
	},

	fetchTheAnswer: function() {
		return new Promise(function(fulfill, reject){
			return fulfill('success');  //如果返回的是fulfill则表示promises执行成功
			//return reject('failure');  //如果返回的是reject则表示promises执行失败
		});
	}
});
```
上述这段代码就是promises的一个简单例子，promises的`then`方法会根据promises的获取到的最终结果执行不同的回调，如果promises获取结果成功则执行`fulfill`回调，否则执行`reject`回调。

promises的强大之处不仅仅如此，promises还可以以链的形式执行多个`then`方法，每个then方法都会根据promises的结果执行`fulfill`或者`reject`回调。
```js
//  app/routes/promises.js

import Ember from 'ember';

export default Ember.Route.extend({

	beforeModel() {
		// 注意Jquery的Ajax方法返回的也是promises
		var promiese = Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');
		promiese.then(this.fetchPhotoOfUsers)
				.then(this.applyInstagramFilters)
				.then(this.uploadThrendyPhotAlbum)
				.then(this.displaySuccessMessage, this.handleErrors);

	},
	fetchPhotoOfUsers: function(){
		console.log('fetchPhotoOfUsers');
	},
	applyInstagramFilters: function() {
		console.log('applyInstagramFilters');
	},
	uploadThrendyPhotAlbum: function() {
		console.log('uploadThrendyPhotAlbum');
	},
	displaySuccessMessage: function() {
		console.log('displaySuccessMessage');
	},
	handleErrors: function() {
		console.log('handleErrors');
	}
});
```
这种情况下会打印什么结果呢？？

在前的文章已经使用过`Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');`获取数据，是可以成功获取数据的。所以promises获取结果成功，应该执行的是获取成功对应的回调方法。浏览器控制台打印结果如下：
```
fetchPhotoOfUsers
applyInstagramFilters
uploadThrendyPhotAlbum
displaySuccessMessage
```
但是如果我把`Ember.$.getJSON('https://api.github.com/repos/emberjs/ember.js/pulls');`改成一个不存在的URL，比如改成`Ember.$.getJSON('https://www.my-example.com');`执行代码之后控制台会提示出`404`错误，并且打印'handleErrors'。说明promises获取结果失败，执行了then里的reject回调。为了验证每个回调的`reject`方法再修改修改代码，如下：
```js
//  app/routes/promises.js

import Ember from 'ember';

export default Ember.Route.extend({

	beforeModel() {
		// 注意Jquery的Ajax方法返回的也是promises
		var promiese = Ember.$.getJSON(' https://www.my-example.com ');
		promiese.then(this.fetchPhotoOfUsers, this.fetchPhotoOfUsersError)
				.then(this.applyInstagramFilters, this.applyInstagramFiltersError)
				.then(this.uploadThrendyPhotAlbum, this.uploadThrendyPhotAlbumError)
				.then(this.displaySuccessMessage, this.handleErrors);

	},
	fetchPhotoOfUsers: function(){
		console.log('fetchPhotoOfUsers');
	},
	fetchPhotoOfUsersError: function() {
		console.log('fetchPhotoOfUsersError');
	},
	applyInstagramFilters: function() {
		console.log('applyInstagramFilters');
	},
	applyInstagramFiltersError: function() {
		console.log('applyInstagramFiltersError');
	},
	uploadThrendyPhotAlbum: function() {
		console.log('uploadThrendyPhotAlbum');
	},
	uploadThrendyPhotAlbumError: function() {
		console.log('uploadThrendyPhotAlbumError');
	},
	displaySuccessMessage: function() {
		console.log('displaySuccessMessage');
	},
	handleErrors: function() {
		console.log('handleErrors');
	}
});
```

![result](/content/images/2016/03/77.png)

由于promises获取结果失败故执行其对应的失败处理回调。这种调用方式有点类似于`try……catch……`，但是本文的重点不是讲解promises，更多有关promises的教材请读者自行Google或者百度吧，在这里介绍一个js库[RSVP.js](https://github.com/tildeio/rsvp.js)，它可以让你更加简单的组织你的promises代码。
在附上几个promises的参考网站：
1. [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
2. [http://liubin.github.io/promises-book/](http://liubin.github.io/promises-book/)
3. [http://www.zhangxinxu.com/wordpress/2014/02/es6-javascript-promise-%E6%84%9F%E6%80%A7%E8%AE%A4%E7%9F%A5/](http://www.zhangxinxu.com/wordpress/2014/02/es6-javascript-promise-%E6%84%9F%E6%80%A7%E8%AE%A4%E7%9F%A5/)

极力推荐看第二个网站的教材，这个网站可以直接运行js代码。还有源码和PDF。非常棒！！！

###2，promises中止路由

当发生路由切换的时候，在`model`回调（或者是`beforeMode`、`afterModel`）中获取的数据集会在切换完成的时候传递到路由对应的`controller`上。如果model回调返回的是一个普通的对象（非promises对象）或者是数组，路由的切换会立即执行，但是如果model回调返回的是一个promises对象，路由的切换将会被中止直到promises执行完成（返回`fulfill`或者是`reject`）才切换。

**路由器任务任何一个包含了then方法的对象都是一个promises。**

如果promises获取结果成功则会从被中止的地方继续往下执行或者是执行路由链的下一个路由，如果promises返回的依然是一个promises，那么路由依然再次被中止，等待promises的返回结果，如果是`fulfill`则从被中止的地方开始往下执行，以此类推，一直到获取到`model`回调所需的结果。

传递到每个路由的`setupController`回调的值都是promises返回`fulfill`时的值。如下代码：
```js
//  app/routes/tardy.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		return new Ember.RSVP.Promise(function(resolver) {
			console.log('start......');
			Ember.run.later(function() {
				resolver({ msg: 'Hold your horses!!'});
			}, 3000);
		});
	},
	setupController(controller, model) {
		console.log('msg = ' + model.msg);
	}
});
```
一进入路由`tardy`，`model`回调就会被执行并且返回一个延迟3秒才执行的promises，在这期间路由会中止。当promises返回`fulfill`路由会继续执行，并将`model`返回的对象传递到`setupController`方法中。

虽然这种中止的行为会影响响应速度但是这是非常必要的，特别是你需要保证`model`回调得到的数据是完整的数据的时候。

### 3，promises获取结果失败

文章前面主要讲的是promises获取结果成功的情况，但是如果是获取结果失败的情况又是怎么处理呢？？

默认情况下，如果`model`回调返回的是一个promises对象并且此promises返回的是`reject`，此时路由切换将被终止，也不会渲染对应的模板，并且会在浏览器控制台打印出错误日志信息，例子`promises-ret-reject.js`会演示。

你可以自定义处理出错信息的逻辑，只要在`route`的`actions`哈希对象中配置即可。当promises获取结果失败的默认情况下会执行一个名为`error`的处理事件，否则会执行你自定义的处理事件。
```js
//  app/routes/promises-ret-reject.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		//  为了测试效果直接返回reject
		return Ember.RSVP.reject('FAIL');
	},
	actions: {
		error: function(reason) {
			console.log('reason = ' + reason);

			//  如果你想让这个事件冒泡到顶级路由application只需要返回true
			//  return true;
		}
	}
});
```
如果没有不允许事件冒泡打印结果仅仅是`reason = FAIL`。并且页面上什么都不显示（不渲染模板）。

如果去掉最后一行代码的注释，让事件冒泡到顶级路由`application`中的默认方法处理，那么结果又是什么呢？

![result](/content/images/2016/03/78.png)

结果是先打印了处理结果，然后再打印出提示错误的日志信息。并且页面上什么都不显示（不渲染模板）。

### 4，恢复promises的reject状态

在前面第3点介绍了promises获取结果失败时会终止路由转换，但是如果`model`返回是一个promises链呢？程序能到就这样死了！！！显然是不行的，做法是把model回调中返回的`reject`转换为`fulfill`。这样就可以继续执行或者切换到下一个路由了！
```js
//  app/routes/funky.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		var promises = Ember.RSVP.reject('FAIL');
		//  由于已经知道promises返回的是reject，所以fulfill回调直接写为null
		return promises.then(null, function() {
			return { msg: '恢复reject状态：其实就是在reject回调中继续执行fulfill状态下的代码。' };
		});
	}
});
```
为了验证`model`回调的结果，直接在模板上显示`msg`。
```html
<!--  app/templates/funky.hbs  -->

funky模板
<br>
{{model.msg}}
```
执行URL：[http://localhost:4200/funky](http://localhost:4200/funky)，得到如下结果：

![result](/content/images/2016/03/79.png)

*说明model回调进入到reject回调中，并正确返回了预期结果。*

到本文为止有关路由这以整章的内容也全部介绍完毕了！！难点在[Ember.js 入门指南之二十六查询参数](http://blog.ddlisting.com/2016/03/29/ember-js-ru-men-zhi-nan-zhi-er-shi-liu-cha-xun-can-shu/)这一篇。能力有限没有把这篇的内容讲明白，暂时搁下待日后完善！

总的来说路由主要职责是获取数据，根据逻辑处理数据。有点MVC架构的dao层，专门做数据的CRUD操作。当然另外一个重要职责就是路由的切换，以及切换的时候参数的设置问题。

结束完这一章下一章接着介绍组件（Component）。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！

