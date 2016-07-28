>声明：对于transition这个词直译是“过渡”的意思，但是总觉得“路由的过渡”读起来总有那么一点别扭，想了下于是就用“切换”替代吧，如有不妥欢迎指正。

我们熟知的Java、PHP等语言都提供了URL的重定向，那么[Ember](http://emberjs.com/)的重定向又是怎么去实现的呢？

如果是从路由重定向到另外一个路由你可以调用`transitionTo`方法，如果是从`controller`重定向到一个`route`则调用`transitionToRoute`方法。`transitionTo`方法所实现的功能与`link-to`的作用是一样的，都可以实现路由的切换。
如果重定向之后的路由包含有动态段你需要解析`model`数据或者指定动态段的值。由于不是直接执行URL所以不会执行重定向之后的路由的`model`回调。

### 1，切换路由前获取model 

如果你想在路由切换的时候不加载`model`你可以调用`beforeModel`回调，在这个回调中实现路由的切换。
```js
beforeModel() {
	this.transitionTo('posts');
}
```

### 2，切换路由后获取model 

有些情况下你需要先根据`model`回调获取到的数据然后判断跳转到某个路由上。此时你可以使用`afterModel`回调方法。
```js
afterModel: function(model, transition) {
	if (model.get(‘length’) === 1) {
		this.transitionTo('post', model.get('firstObject'));
	}
}
```
切换路由，并初始化数据为`model`的第一个元素数据。

### 3，重定向到子路由 

```js
Router.map(function() {
    this.route('posts', function() {
        this.route('post', { path: '/:post_id'});
    }); 
});
```
子路由的重定向有些许不同，如果你需要重定向到上面这个段代码的子路由`posts.post`上，如果是使用`beforeModel`、`model`、`afterModel`回调重定向到`posts.post`父路由`posts`会重新在执行一次，再次执行父路由这种方式就显得有点多余了，特别父路由需要加载的数据比较多的时候，会影响到加载的效率。
如果是这种情况我们可以使用`redirect`回调，此回调不会再次执行父路由。仅仅是实现路由切换而已。
```js
redirect: function(model, transition) {
	if (model.get('length') === 1) {
		this.transitionTo('posts.post', model.get('firstObject'));
	}
}
```
重定向到子路由，解析之后会得到的类似于`posts/2`这种形式的URL。

以上就是全部路由的重定向方式，主要有4个回调：`beforeModel`、`model`、`afterModel`、`redirect`。前面三种使用场景差别不大，`redirect`主要用于重定向到子路由。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
