模型也是一个类，它定义了向用户展示的属性和数据行为。模型的定义非常简单，只需要继承[DS.Model](http://emberjs.com/api/data/classes/DS.Model.html)类即可，或者你也可以直接使用[Ember CLI](http://ember-cli.com/user-guide)命令创建。比如使用命令模型 `ember g model person`定义了一个模型类`person`。
```js
//  app/models/person.js

import DS from 'ember-data';

export default DS.Model.extend({
  
});
```
这个是个空的模型，没有定义任何属性。有了模型类你就可以使用`find`方法查找数据了。

## 1，定义属性

上面定义的模型类`person`还没有任何属性，下面为这个类添加几个属性。
```js
//  app/models/person.js

import DS from 'ember-data';

export default DS.Model.extend({
	firstName: DS.attr(),
	lastName: DS.attr(),
	birthday: DS.attr()  
});
```
上述代码定义了3个属性，但是还未给属性指定类型，默认都是`string`类型。这些属性名与你连接的服务器上的数据`key`是一致的。甚至你还可以在模型中定义[计算属性](http://blog.ddlisting.com/2016/03/17/ember-js-ru-men-zhi-nan-ji-suan-shu-xing-compute-properties/)。
```js
//  app/models/person.js

import DS from 'ember-data';

export default DS.Model.extend({
	firstName: DS.attr(),
	lastName: DS.attr(),
	birthday: DS.attr(),

	fullName: Ember.computed('firstName', 'lastName', function() {
		return `${this.get('firstName')} ${this.get('lastName')}`;
	})
});
```
这段代码在模型类中定义了一个计算属性`fullName`。

## 2，指定属性类型与默认值

前面定义的模型类是没有指定属性类型的，默认情况下都是`string`类型，显然这是不够的，简单的模型属性类型包括：`string`，`number`，`boolean`，`date`。这几个类型我想不用我解释都应该知道了。

不仅可以指定属性类型，你还可以指定属性的默认值，在[attr()](http://emberjs.com/api/data/classes/DS.html#method_attr)方法的第二个参数指定。比如下面的代码：
```js
//  app/models/person.js

import DS from 'ember-data';

export default DS.Model.extend({
	username: DS.attr('string'),
	email: DS.attr('string'),
	verified: DS.attr('boolean', { defaultValue: false }),  //指定默认值是false
	//  使用函数返回值作为默认值
	createAt: DS.attr('string', { defaultValue(){ return new Date(); } })
});
```
正如代码注释所述的，设置默认值的方式包括直接指定或者是使用函数返回值指定。

## 3，定义模型的关联关系
	
[Ember](http://emberjs.com)的模型也是有类似于数据库的关联关系的。只是相对于复制的数据库[Ember](http://emberjs.com)的模型就显得简单很多，其中包括一对一，一对多，多对多关联关系。这种关系是与后台的数据库是相统一的。

#### 1，一对一

声明一对一关联使用[DS.belongsTo](http://emberjs.com/api/data/classes/DS.html#method_belongsTo)设置。比如下面的两个模型。
```js
//  app/models/user.js
import DS from 'ember-data';

export default DS.Model.extend({
  profile: DS.belongsTo(‘profile’);
});
```
```js
//  app/models/profile.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  user: DS.belongsTo(‘user’);
});
```

#### 2，一对多
	
声明一对多关联使用[DS.belongsTo](http://emberjs.com/api/data/classes/DS.html#method_belongsTo)（多的一方使用）和[DS.hasMany](http://emberjs.com/api/data/classes/DS.html#method_hasMany)（少的一方使用）设置。比如下面的两个模型。
```js
//  app/models/post.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  comments: DS.hasMany(‘comment’);
});
```
这个模型是一的一方。下面的模型是多的一方；
```js
//  app/models/comment.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  post: DS.belongsTo(‘post’);
});
```
这种设置的方式与Java 的hibernate非常相似。

#### 3，多对多

声明一对多关联使用[DS.hasMany](http://emberjs.com/api/data/classes/DS.html#method_hasMany)设置。比如下面的两个模型。
```js
//  app/models/post.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  tags: DS.hasMany(‘tag’);
});
```
```js
//  app/model/tag.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  post: DS.hasMany(‘post’);
});
```
多对多的关系设置都是使用[DS.hasMany](http://emberjs.com/api/data/classes/DS.html#method_hasMany)，但是并不需要“中间表”，这个与数据的多对多有点不同，如果是数据的多对多通常是通过中间表关联。

## 4，显式反转

[Ember Data](https://github.com/emberjs/data)会尽力去发现两个模型之间的关联关系，比如前面的一对多关系中，当`comment`发生变化的时候会自动更新到`post`，因为每一个`comment`只对应一个`post`，可以有`comment`确定到某个一个`post`。

然而，有时候同一个模型中会有多个与此关联模型。这时你可以在反向端用[DS.hasMany](http://emberjs.com/api/data/classes/DS.html#method_hasMany)的`inverse`选项指定其关联的模型：
```js
//  app/model/comment.js
import DS from 'ember-data';
export default DS.Model.extend({
  onePost: DS.belongsTo(‘post’),
  twoPost: DS.belongsTo(‘post’),
  redPost: DS.belongsTo(‘post’),
  bluePost: DS.belongsTo(‘post’)
});
```
在一个模型中同时与3个`post`关联了。
```js
//  app/models/post.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  comments: hasMany(‘comment’, { inverse: ‘redPost’ });
});
```
当`comment`发生变化时自动更新到`redPost`这个模型。

## 5，自反关系

#### 1，一对多

当你想定义一个自反关系的模型时（模型本身的一对一关系），你必须要显式使用`inverse`指定关联的模型。如果没有逆向关系则把`inverse`值设置为`null`。
```js
//  app/models/folder.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  children: DS.hasMany(‘folder’, { reverse: ‘parent’ });
  parent: DS.hasMany(‘folder’, { reverse: ‘children’ });
});
```
一个文件夹通常有父文件夹或者子文件夹。此时父文件夹和子文件夹与本身都是同一个类型的模型。此时你需要显式使用`inverse`属性指定，比如这段代码所示，“children……”这行代码意思是这个模型有一个属性`children`，并且这个属性也是一个`folder`，模型本身作为父文件夹。同理“parent……”这行代码的意思是这个模型有个属性`parent`，并且这个属性也是一个`folder`，模型本身是这个属性的子文件夹。比如下图结构：

![结构图](/content/images/2016/04/140.png)

这个有点像数据结构中的链表。你可以把`children`和`parent`想象成是一个指针。

如果仅有关联关系没有逆向关系直接把`inverse`设置为`null`。
```js
//  app/models/folder.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  parent: DS.belongsTo(‘folder’, { inverse: null });
});
```

#### 2，一对一

```js
//  app/models/user.js
import DS from ‘ember-data’;
export default DS.Model.extend({
  bestFriend: DS.belongsTo(‘folder’, { inverse: ‘bestFriend’ });
});v
```
这个关系与数据库设置设计中双向一对一很类似。

## 6，嵌套数据

有些模型可能会包含深层嵌套的数据对象，如果也是使用上述的关联关系定义那么将是个噩梦！对于这种情况最好是把数据定义成简单对象，虽然增加点冗余数据但是降低了层次。另外一种是把嵌套的数据定义成模型的属性（也是增加冗余但是降低了嵌套层次）。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！









