当你的应用启动的时候，路由器就会匹配当前的URL到你定义的路由上。然后按照定义的路由层次逐个加载数据、设置应用程序状态、渲染路由对应的模板。

### 1，基本路由	

在`app/router.js`的`map`方法里定义的路由会映射到当前的URL。当`map`方法被调用的时候方法体内的`route`方法就会创建路由。

下面使用[Ember CLI](http://ember-cli.com/user-guide)命令创建两个路由：
```
ember generate route about
ember generate route favorites
```
命令执行完之后你可在你的项目目录`app/routes`下面看到已经创建好的两个路由文件已经`app/templates`下面路由对应的模板文件。
此时在`app/router.js`的`map`方法中已经存在了刚刚创建的路由配置。这个是[Ember CLI](http://ember-cli.com/user-guide)自动为你创建了。
```javascript
// app/router.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {
  this.route('about');
  this.route('favorites');
});

export default Router;
```
现在分别修改`app/templates`下面的两个模板文件如下：
```html
<!-- app/templates/about.hbs -->

这个是about模板！<br>
{{outlet}}
```
```html
<!-- app/templates/favorites.hbs -->

这个是favorites模板！<br>
{{outlet}}
```
然后访问[http://localhost:4200/about](http://localhost:4200/about)或者[http://localhost:4200/favorites](http://localhost:4200/favorites)，如果你的程序没有问题你也会得到如下显示结果：

![run result](/content/images/2016/03/55.png)

如果你觉得`favorites`这个路由名字太长是否可以修改成其他名字呢？答案是肯定的，你只要修改`router.js`中`map`方法的配置即可。
```javascript
Router.map(function() {
  this.route('about');
  // 注意：访问的URL可以写favs但是项目中如果是使用route的地方仍然是使用favorites
  this.route('favorites', { path: '/favs' });
});
```
此时访问：[http://localhost:4200/favs](http://localhost:4200/favs)，界面显示的结果与之前是一样的。

**说明**：默认情况下访问的URL与路由名字是一致的，比如`this.route('about')`与`this.route('about', { path: ‘/about’ })`是同一个路由，如果URL与路由不同名则需要使用`{path: '/xxx'}`设置映射的URL。

在handlebars模板中可以使用`{{link-to}}`助手在不同的路由间切换，使用时需要在`link-to`助手内指定路由名称。比如下面的代码使用`link-to`助手实现在`about`和`favs`两个路由间切换。
为了页面能美观一点引入[bootstrap](http://www.bootcss.com/)，使用[npm](https://www.npmjs.com/)命令安装：`bower install bootstrap`，如果安装成功你可以在bower_components目录下看到[bootstrap](http://www.bootcss.com/)相关的文件。安装成功之后引入到项目中，修改`chapter3_routes/ember-cli-build.js`。在`return`语句前加入如下两行代码(作用就是引入[bootstrap](http://www.bootcss.com/)框架)：
```javascript
app.import("bower_components/bootstrap/dist/css/bootstrap.css");
app.import("bower_components/bootstrap/dist/js/bootstrap.js");
```
修改`application.hbs`，增加一个导航菜单。
```html
<!-- //app/templates/application.hbs -->

<nav class="navbar navbar-inverse navbar-fixed-top">
    <div class="container-fluid">
            <div class="navbar-header" href="#">
                <!--  <a class="navbar-brand" href="#">Blog</a> -->
                {{#link-to 'index' class="navbar-brand"}}Home{{/link-to}}
            </div>
            <ul class="nav navbar-nav">
                <li>{{#link-to 'about'}}about{{/link-to}}</li>
       			<li>{{#link-to 'favorites'}}favorites{{/link-to}}</li>
            </ul>
            <ul class="nav navbar-nav navbar-right">
                <li><a href="#">Login</a></li>
                <li><a href="#">Logout</a></li>
            </ul>
    </div>
</nav>

<div class="container-fluid" style="margin-top: 70px;">
<!-- 项目中其他所有的模板都是application的子模板，所以其他模板都会渲染到这里的 outlet上 -->
{{outlet}}
</div>
```
如果看到页面没有[bootstrap](http://www.bootcss.com/)效果请重新启动项目。如果运行项目后再浏览器控制台出现如下错误。

![run result](/content/images/2016/03/56.png)

如果出现上图错误需要在`config/environment.js`中加上一些安全策略设置代码，有关设置的解释请看下面网址的文章介绍。

1. [ember-cli-and-content-security-policy-cs](https://blog.justinbull.ca/ember-cli-and-content-security-policy-csp/)
2. [https://www.w3.org/TR/2015/CR-CSP2-20150721/](https://www.w3.org/TR/2015/CR-CSP2-20150721/)
```javascript
, contentSecurityPolicy: {
      'default-src': "'none'",
      'script-src': "'self' 'unsafe-inline' 'unsafe-eval' use.typekit.net connect.facebook.net maps.googleapis.com maps.gstatic.com",
      'font-src': "'self' data: use.typekit.net",
      'connect-src': "'self'",
      'img-src': "'self' www.facebook.com p.typekit.net",
      'style-src': "'self' 'unsafe-inline' use.typekit.net",
      'frame-src': "s-static.ak.facebook.com static.ak.facebook.com www.facebook.com"
    }
```

![配置截图](/content/images/2016/03/57.png)

上图是我的演示项目配置。然后点击`about`会得到如下界面

![result](/content/images/2016/03/58.png)

可以看到浏览器地址栏的URL变为`about`了，并且页面显示的内容也是`about`模板的内容。同理你点击`favorites`地址栏就变为[http://localhost:4200/favs](http://localhost:4200/favs)并且显示的内容是`favorites`的（为什么URL是`favs`而不是`favorites`呢，因为前面已经修改了`route`和URL的映射关系，路由`favorites`对应的URL是`favs`）。
上述演示的就是路由的切换！！！

可以看到浏览器地址栏的URL变为about了，并且页面显示的内容也是about模板的内容。同理你点击“favorites”地址栏就变为http://localhost:4200/favs并且显示的内容是favorites的（为什么URL是favs而不是favorites呢，因为前面已经修改了route和URL的映射关系，路由favorites对应的URL是favs）。
上述演示的就是路由的切换！！！

### 2，路由嵌套	

还记得在前面的[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)这篇文章的内容吗？在这篇文章中比较详细的介绍了路由的嵌套与怎么使用嵌套的路由。不妨回过头去看看。在这里打算就不讲了……如果有不明白的请看官网的教程。

### 3，application路由

`application`路由是默认的路由，是程序的入口，所有其他自定义的路由都先进过`application`才到自定义的路由。并且`application`路由对应的`application.hbs`模板是所有自定义模板的父模板，所有自定义的模板都会渲染到application.hbs模板的`{{outlet}}`上。有关于路由的执行顺序以及模板的渲染顺序在前面的[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)也讲过了，在此也不打算在做过多的介绍了。你可以回头看之前的文章或者到官网查看。

### 4，index路由

对于所有的嵌套的路由，包括最顶层的路由Ember会自动生成一个访问URL为`/`对应路由名称为`index`的路由。

比如下面的两种路由设置是等价的。
```javascript
//  app/router.js
// ……

Router.map(function() {
    this.route('about');
    // 注意：访问的URL可以写favs但是项目中如果是使用route的地方仍然是使用favorites
    this.route('favorites', { path: '/favs' });
});

export default Router;
```

```javascript
//  app/router.js
// ……

Router.map(function() {
	this.route('index', { path: '/' });
    this.route('about');
    // 注意：访问的URL可以写favs但是项目中如果是使用route的地方仍然是使用favorites
    this.route('favorites', { path: '/favs' });
});

export default Router;
```
`index`路由会渲染到`application.hbs`模板的`{{outlet}}`上。这个是Ember默认设置。当用户访问`/about`时Ember会把`index`模板替换为`about`模板。

对于路由嵌套的情况也是如此。
```javascript
//  app/router.js
//  ……
Router.map(function() {

    this.route('posts', function() {
    	this.route('new');
    });

});

export default Router;
```
```javascript
//  app/router.js

//  ……
Router.map(function() {
	this.route('index', { path: '/' });
    this.route('posts', function() {
    	this.route('index', { path: '/' });
    	this.route('new');
    });

});

export default Router;
```
两种设置方式都会得到如下图的路由表。打开浏览器的“开发者工具”点开“Ember”选项卡，在点开“/#Routes”你就可以看到如下路由表（显示是顺序有可能跟你的不一样）。

![路由渲染结果](/content/images/2016/03/59.png)

注：`loading`和`error`这两个路由是ember自动生成的，他们的用法会在后面的文章介绍。

当用户访问`/posts`时实际进入的路由是`posts.index`对应的模板是`posts/index.hbs`，但是实际中我并没有创建这个模板，因为Ember默认把这个模板渲染到`posts.hbs`的`{{outlet}}`上。由于这个模板不存在也就相当于什么都没做。当然你也可以创建这个模板。
<br>使用命令：`ember generate template posts/index`然后在这个模板中添加以下显示的内容：
```html
<!-- app/templates/posts/index.hbs  -->

<h2>这里是/posts/index.hbs。。。</h2>
```
再此访问[http://localhost:4200/posts](http://localhost:4200/posts)，是不是可以看到增加的内容了。

你可以这么理解对于每一个有子路由的路由都有一个名为`index`的子路由并且这个路由对应的模板为`index.hbs`，如果把有子路由的路由当做一个模块看待那么`index.hbs`就是这个模块的首页。特别是做过一些信息系统的朋友应该是很熟悉的，基本上没个子模块都会有一个首页，这个首页现实的内容就是一进入这个模块时就显示的内容。既然是子模板当然也不会例外它也会渲染到父模板的`{{outlet}}`上。比如上面的例子当用户访问[http://localhost:4200/posts](http://localhost:4200/posts)实际进入的是[http://localhost:4200/posts/](http://localhost:4200/posts/)（后面多了一个`/`，这个`/`对应的模板就是`index`），当用户访问的是[http://localhost:4200/posts/new](http://localhost:4200/posts/new)，那么进入的就是`posts/new.hbs`这个模板（也是渲染到`posts.hbs`的`{{outlet}}`上）。

### 4，动态段 

关于动态段在前面的[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)也介绍过了，在这里就再简单补充下。

路由最主要的任务之一就是加载`model`。

例如对于路由`this.route('posts');`会加载项目中所有的`posts`下的`model`。但是当你只想加载其中一个`model`的时候怎么处理呢？而且大多数情况我们是不需要一次性加载完全部数据的，一般情况都是加载其中一小部分。这个时候就需要动态段了！

动态段以`:`开头，并且后面接着`model`的`id`属性。

```javascript
//  app/router.js

//  ……

Router.map(function() {
    this.route('about');
    // 注意：访问的URL可以写favs但是项目中如果是使用route的地方仍然是使用favorites
    // this.route('favorites', { path: '/favs' });

    this.route('posts', function() {
    	this.route('post', { path: '/:post_id'});
    });
    
});

export default Router;
```
此时你可以访问[http://localhost:4200/posts/1](http://localhost:4200/posts/1)，不过我们还没创建`model`所以会报错。这个我们暂时不管，后面会有一章是介绍`model`的。现在只要知道你访问[http://localhost:4200/posts/1](http://localhost:4200/posts/1)就相当于获取`id`值唯一的`model`。

### 5，通配符/全局路由

Ember也同样运行你使用`*`作为URL通配符。有了通配符你可以设置多个URL访问同一个路由。
```javascript
this.route('about', { path: '/*wildcard' });
```
然后访问：[http://localhost:4200/wildcard](http://localhost:4200/wildcard)或者访问[http://localhost:4200/2423432ffasdfewildcard](http://localhost:4200/2423432ffasdfewildcard)或者[http://localhost:4200/2333](http://localhost:4200/2333)都是可以进入到`about`这个路由，但是[http://localhost:4200/posts](http://localhost:4200/posts)仍然进入的是`posts`这个路由。因为可以匹配到这个路由。

### 6，重置子路由的命名空间

在有路由嵌套的情况下，一般情况我们访问URL的格式都是`父路由名/子路由名`，Ember提供了一个`resetNamespace:true`选项可以用户重置子路由的命名空间，使用这个设置的路由可以直接这样访问`/子路由名`，不需要写父路由名。
```javascript
this.route('posts', function() {
    this.route('post', { path: '/:post_id'});
    this.route('comments', { resetNamespace: true}, function() {
    	this.route('new');
    });
});
```
此时如果你想问的`new`这个路由你可以直接不写`comments`。[http://localhost:4200/posts/new](http://localhost:4200/posts/new)，而不需要[http://localhost:4200/posts/comments/new](http://localhost:4200/posts/comments/new)，不过模板渲染的顺序没变，`new`模板仍然是渲染到`comments`的`{{outlet}}`上。

不过个人觉得还是不使用这个设置比较好，特别是在开发的时候你可以看到访问的URL的层次，对你调试代码还是很有帮助的。

以上的内容就是定义路由的全部内容。都是非常重要的知识，希望你能好好掌握，对于路由的嵌套请看之前的文章。如果有疑问请给我留言或者访问官网看原教程。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
