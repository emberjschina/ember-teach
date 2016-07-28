在前面[Ember.js 入门指南之三十八定义模型](http://blog.ddlisting.com/2016/04/08/ding-yi-mo-xing/)中介绍过模型之前的关系。主要包括一对一、一对多、多对多关系。但是还没介绍两个有关联关系模型的更新、删除等操作。

为了测试新建两个模型类。
```
ember g model post
ember g model comment
```
## 1，创建关系记录

```js
//  app/models/post.js

import DS from 'ember-data';

export default DS.Model.extend({
  comments: DS.hasMany('comment')
});

//  app/model/comment.js

import DS from 'ember-data';

export default DS.Model.extend({
  	post: DS.belongsTo('post')
});
```
设置关联，关系的维护放在多的一方`comment`上。
```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.createRecord('comment', {
  post: post
});
comment.save();
```
保存之后`post`会自动关联到`comment`上（保存`post`的`id`属性值到`post`属性上）。

当然啦，你可以在从`post`上设置关联关系。比如下面的代码：
```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.createRecord('comment', {
	//  设置属性值
});
//  手动吧对象设置到post数组中。（post是多的一方，comments属性应该是保存关系的数组）
post.get('comments').pushObject(comment);
comment.save();
```
如果你学过Java里的hibernate框架我相信你很容易就能理解这段代码。你可以想象，`post`是一的一方，如果它要维护关系是不是要把与其关联的`comment`的`id`保存到`comments`属性（数组）上，因为一个`post`可以关联多个`comment`，所以`comments`属性应该是一个数组。

## 2，更新已经存在的记录

更新关联关系与创建关联关系几乎是一样的。也是首先获取需要关联的模型在设置它们的关联关系。
```js
let post = this.store.peekRecord('post', 100);
let comment = this.store.peekRecord('comment', 1);
comment.set('psot', post);  //  重新设置comment与post的关系
comment.save();  //  保存关联的关系
```
假设原来`comment`关联的`post`是`id`为`1`的数据，现在重新更新为`comment`关联`id`为`100`的`post`数据。

如果是从`post`方更新，那么你可以像下面的代码这样：
```js
let post = this.store.peekRecord('post', 100);
let comment this.store.peekRecord('comment', 1);
post.get('comments').pushObject(comment);  // 设置关联
post.save();  //  保存关联
```

## 3，删除关联关系

既然有新增关系自然也会有删除关联关系。
如果要移除两个模型的关联关系，只需要把关联的属性值设置为`null`就可以了。
```js
let comment = this.store.peekRecord('comment', 1);
comment.set('post', null);  //解除关联关系
comment.save();
```
当然你也可以从一的一方移除关联关系。
```js
let post = this.store.peekRecord('post', 1);
let comment = this.store.peekRecord('comment', 1);
post.get('comments').removeObject(comment);  // 从关联数组中移除comment
post.save();
```
从一的一方维护关系其实就是在维护关联的数组元素。
	
只要Store改变了Handlebars模板就会自动更新页面显示的数据，并且在适当的时期Ember Data会自动更新到服务器上。

有关于模型之间关系的维护就介绍到这里，它们之间关系的维护只有两种方式，一种是用一的一方维护，另一种是用多的一方维护，相比来说，从一的一方维护更简单。但是如果你需要一次性更新多个纪录的关联时使用第二种方式更加合适（都是针对数组操作）。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
