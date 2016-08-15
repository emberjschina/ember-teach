在[Ember](http://emberjs.com)应用中适配器决定了数据保存到后台的方式，比如URL格式和请求头部。Ember Data默认的适配器是内置的[REST API回调](http://jsonapi.org/)。

实际使用中经常会扩展默认的适配器。Ember的立场是应该通过扩展适配器来添加不同的功能，而非添加标识。这样可以使得代码更加容易测试、更加容易理解，同时也降低了可能需要扩展的适配器的代码。

如果你的后端使用的是Ember约定的规则那么可用使用适配器`adapters/application.js`。适配器`application`优先级比默认的适配器高，但是比指定的模型适配器优先级低。模型适配器定义规则是：`adapter-modelName.js`。比如下面的代码定义了一个模型适配器`adapter-post`。
```js
//  app/adapters/adapter-post.js

import Ember from 'ember';
export default DS.JSONAPIAdapter.extend({
	namespace: 'api/v1'
});
```
此时适配器的优先级次序为：`JSONAPIAdapter` > `application `> 默认内置适配器；

Ember内置的是配有如下几种：

1. [DS.Adapter](http://emberjs.com/api/data/classes/DS.Adapter.html) 这个适配器是最基础的适配器，它不包含任何功能。如果需要创建一个与Ember适配器有根本性区别的适配器可以使从这个适配器入手。
2. [DS.JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html) 这个适配器是默认适配器，并且是遵循JSON API规范，用于与HTTP服务器之间通过XHR交互JSON数据。
3. [DS.RESTAdapter](http://emberjs.com/api/data/classes/DS.RESTAdapter.html) 这个适配器与第二个适配器功能一样，并且在Ember Data2.0之前是作为默认的适配器。

## 1，自定义JSONAPIAdapter适配器

`JSONAPIAdapter`适配器通常用于扩展非标准的后台接口。

#### 1，URL规范

`JSONAPIAdapter`适配器非常智能，它能够自动确定URL链接是那个模型。比如，如果你需要通过`id`获取`post`：
```js
this.store.find('post', 1).then(function(post) {
    //  处理post
});
```
`JSONAPIAdapter`会自动发送`get`请求到`/post/1`。

下表是Action、请求、URL三者的映射关系（由于本站markdown解析器不支持表格所以直接使用截图替代了）。

![映射关系截图](/content/images/2016/04/177.png)

比如在`action`中执行`find()`方法，会发送`get`请求，`JSONAPIAdapter`会自动解析成形如`/posts/1`的URL。

#### 2，多元化定制

为了适配复数名字的模型属性名称，可以是使用[Ember Inflector](https://github.com/stefanpenner/ember-inflector)绑定别名。
```js
let inflector = Ember.Inflector.inflector;
inflector.irregular('formula', 'formulae');
inflector.uncountable('advice');
```
这样绑定之后目的是告诉`JSONAPIAdapter`。你可以使用`/formulae/1`代替`/formulas/1`。但是目前我还没搞清楚这个设置是什么意思？又有什么用？如果读者知道请指教。

#### 3，断点路径定制

使用属性`namespace`可以设置URL的前缀。
```js
app/adapters/application.js
import Ember from 'ember';
export default DS.JSONAPIAdapter.extend({
  namespace: 'api/1'
});
```
请求`person`会自动转发到`/api/1/people/1`。

#### 4，自定义主机

默认情况下适配器会转到当前域名下。如果你想让URL转到新的域名可以使用属性host设置。
```js
app/adapters/application.js
import Ember from 'ember';
export default DS.JSONAPIAdapter.extend({
  host: 'http://api.example.com'
});
```
**注意**：如果你的项目是使用自己的后台数据库这个设置特别重要！！！属性`host`所指的就是你服务器接口的地址。

请求`person`会自动转发到`http://api.example.com/people/1`。

#### 5，自定义路径

默认情况下Ember会尝试去根据复数的模型类名、中划线分隔的模型类名生成URL。如果需要改变这个默认的规则可以使用属性`pathForType`设置。
```js
// app/adapters/application.js
import Ember from ‘ember’;
export default DS.JSONAPIAdapter.extend({
  pathForType: function(type) {
    return Ember.String.underscore(type);
  }
});
```
修改默认生成路径规则为下滑线分隔。比如请求`person`会转向`/person/1`。请求`user-profile`会转向`/user_profile/1`（默认情况转向`user-profile/1`）。

#### 6，自定义请求头

有些请求需要设置请求头信息，比如提供`API`的`key`。可以以键值对的方式设置头部信息，Ember Data会为每个请求都加上这个头信息设置。
```js
app/adapters/application.js
import Ember from 'ember';
export default DS.JSONAPIAdapter.extend({
  headers: {
'API_KEY': 'secret key',
'ANOTHER_HEADER': 'some header value'
  }
});
```
更强大地方是你可以在`header`中使用计算属性，动态设置请求头信息。下面的代码是将一个`session`对象设置到适配器中。
```js
app/adapters/application.js
import Ember from ‘ember’;
export default DS.JSONAPIAdapter.extend({
  session: Ember.inject.service(‘session’);
  headers: Ember.computed(‘session.authToken’, function() {
‘API_KEY’: this.get(‘session.authToken’),
‘ANOTHER_HEADER’: ‘some header value’  
  });
});
```
对于`session`应该非常不陌生，特别是在控制用户登录方面使用非常广泛，另外又一个非常好用的插件，专门用于控制用户登录的，这个插件是[Ember Simple Auth](https://github.com/simplabs/ember-simple-auth)，另外有一篇博文是介绍如何使用这个插件实现用户登录的，请看[使用ember-simple-auth实现Ember.js应用的权限控制](http://blog.ddlisting.com/2015/11/20/ember-application-authority-control/)的介绍。

你还可以使用`volatile()`方法设置计算属性为非缓存属性，这样每次发送请求都会重新计算`header`里的值。
```js
// app/adapters/application.js
import Ember from ‘ember’;
export default DS.JSONAPIAdapter.extend({
  //  ……
  }).volatile();
});
```

更多有关于适配器的信息浏览下面的网址：

* [Ember Observer](http://emberobserver.com/categories/data)
* [GitHub](https://github.com/search?q=ember+data+adapter&ref=cmdform)
* [Bower](http://bower.io/search/?q=ember-data-)

对于适配器主要掌握`JSONAPIAdapter`足矣，如果你需要个性化定制URL或者请求的域名可以在适配中配置。不过大部分情况下都是使用默认设置。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！