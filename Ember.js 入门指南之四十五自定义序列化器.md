在[Ember](http://emberjs.com)应用中，序列化器会格式化与后台交互的数据，包括发送和接收的数据。默认情况下会使用[JSON API](http://jsonapi.org/)序列化数据。如果你的后端使用不同的格式，Ember Data允许你自定义序列化器或者定义一个完全不同的序列化器。

Ember Data内置了三个序列化器。[JSONAPISerializer](http://emberjs.com/api/data/classes/DS.JSONAPISerializer.html)是默认的序列化器，用与处理后端的JSON API。[JSONSerializer](http://emberjs.com/api/data/classes/DS.JSONSerializer.html)是一个简单的序列化器，用与处理单个JSON对象或者是处理记录数组。[RESTSerializer](http://emberjs.com/api/data/classes/DS.RESTSerializer.html)是一个复杂的序列化器，支持侧面加载，在Ember Data2.0之前是默认的序列化器。

## JSONAPISerializer规范

当你向服务器请求数据时，JSONSerializer会把服务器返回的数据当做是符合下列规范的JSON数据。

**注意**：特别是项目使用的是自定义适配器的时候，后台返回的数据格式必须符合[JSOP API](http://www.jsonapi.org)规范，否则无法实现数据的CRUD操作，Ember就无法解析数据，关于自定义适配器这点的知识请看上一篇[Ember.js 入门指南之四十四自定义适配器](http://blog.ddlisting.com/2016/04/17/zi-ding-yi-gua-pei-qi/)，在文章中有详细的介绍自定义适配器和自定义序列化器是息息相关的。

#### 1，JSON API文档

JSONSerializer期待后台返回的是一个符合JSON API规范和约定的JSON文档。比如下面的JSON数据，这些数据的格式是这样的：

1. type指定model的名称
2. 属性名称使用中划线分隔

比如请求`/people/123`，响应的数据如下：
```js
{
  "data": {
    "type": "people",
    "id": "123",
    "attributes": {
      "first-name": "Jeff",
      "last-name": "Atwood"
    }
  }
}
```
如果响应的数据有多条，那么`data`将是以数组形式返回。
```js
{
  "data": [
  	{
	    "type": "people",
	    "id": "123",
	    "attributes": {
	      "first-name": "Jeff",
	      "last-name": "Atwood"
	    }
	},{
	    "type": "people",
	    "id": "124",
	    "attributes": {
	      "first-name": "chen",
	      "last-name": "ubuntuvim"
	    }
	}
  ]
}
```
#### 2，拷贝数据

数据有时候并不是请求的主体，如果数据有链接。链接的关系会放在`included`下面。
```js
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON API paints my bikeshed!"
    },
    "links": {
      "self": "http://example.com/articles/1"
    },
    "relationships": {
      "comments": {
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  }],
  "included": [{
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```
从JSON数据看出，`id`为`5`的`comment`链接是`"self": http://example.com/comments/5`。`id`为`12`的`comment`链接是`"self": http://example.com/comments/12`。并且这些链接是单独放置`included`内。

#### 3，自定义序列化器

Ember Data默认的序列化器是JSONAPISerializer，但是你也可以自定义序列化器覆盖默认的序列化器。

要自定义序列化器首先要定义一个名为`application`序列化器作为入口。

直接使用命令生成：`ember g serializer application`
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({

});
```
甚至你还可以针对某个模型定义序列化器。比如下面的代码为`post`定义了一个专门的序列化器，在前一篇自定义适配器中介绍过如何为一个模型自定义适配器，这个两个是相关的。
```js
//  app/serializers/post.js
import DS from ‘ember-data’;
export default DS.JSONSerializer.extend({
});
```
如果你想改变发送到后端的JSON数据格式，你只需重写`serialize`回调，在回调中设置数据格式。

比如前端发送的数据格式是如下结构， 
```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "amount": 100,
      "currency": "SEK"
    },
    "type": "product"
  }
}
```
但是服务器接受的数据结构是下面这种结构：
```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "cost": {
        "amount": 100,
        "currency": "SEK"
      }
    },
    "type": "product"
  }
}
```
此时你可以重写`serialize`回调。
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({
	serialize: function(snapshot, options) {
		var json = this._super(...arguments);  // ??
		json.data.attributes.cost = {
			amount: json.data.attributes.amount,
			currency: json.data.attributes.currency
		};

		delete json.data.attributes.amount;
		delete json.data.attributes.currency;

		return json;
	}
});
```
那么如果是反过来呢。
如果后端返回的数据格式为：
```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "cost": {
        "amount": 100,
        "currency": "SEK"
      }
    },
    "type": "product"
  }
}
```
但是前端需要的格式是：
```js
{
  "data": {
    "attributes": {
      "id": "1",
      "name": "My Product",
      "amount": 100,
      "currency": "SEK"
    },
    "type": "product"
  }
}
```
此时你可以重写回调方法`normalizeResponse`或`normalize`，在方法里设置数据格式：
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({

	normalizeResponse: function(store, primaryModelClass, payload, id, requestType) {
		payload.data.attributes.amount = payload.data.attributes.cost.amount;
		payload.data.attributes.currency = payload.data.attributes.cost.currency;

		delete payload.data.attributes.cost;

		return this._super(...arguments);
	}
});
```
####4，数据ID属性

每一条数据都有一个唯一值作为`ID`，默认情况下Ember会为每个模型加上一个名为`id`的属性。如果你想改为其他名称，你可以在序列化器中指定。
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({
	primatyKey: '__id'
});
```
把数据主键名修改为`__id`。

#### 5，属性名

Ember Data约定的属性名是驼峰式的命名方式，但是序列化器却期望的是中划线分隔的命名方式，不过Ember会自动转换，不需要开发者手动指定。然而，如果你想修改这种默认的方式也是可以的，只需在序列化器中使用属性`keyForAttributes`指定你喜欢的分隔方式即可。比如下面的代码把序列号的属性名称改为以下划线分隔：
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({
  keyForAttributes: function(attr) {
    return Ember.String.underscore(attr);  
  }
});
```

#### 6，指定属性名的别名

如果你想模型数据被序列化、反序列化时指定模型属性的别名，直接在序列化器中使用`attrs`属性指定即可。
```js
//  app/models/person.js
export default DS.Model.extend({
  lastName: DS.attr(‘string’)
});
```
指定序列化、反序列化属性别名：
```js
//  app/serializers/application.js

import DS from 'ember-data';

export default DS.JSONSerializer.extend({
  attrs: {
    lastName: ‘lastNameOfPerson’
  }
});
```
指定模型属性别名为`lastNameOfPerson`。

#### 7，模型之间的关联关系

一个模型通过`ID`引用另一个模型。比如有两个模型存在一对多关系：
```js
//  app/models/post.js
export default DS.Model.extend({
  comments: DS.hasMany(‘comment’, { async: true });
});
```
序列化后JSON数据格式如下，其中关联关系通过一个存放`ID`属性值的数组实现。
```js
{
  "data": {
    "type": "posts",
    "id": "1",
    "relationships": {
      "comments": {
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    }
  }
}
```
可见，有两个`comment`关联到一个`post`上。
如果是`belongsTo`关系的，JSON结构与`hadMany`关系相差不大。
```js
{
  "data": {
    "type": "comment",
    "id": "1",
    "relationships": {
      "original-post": {
        "data": { "type": "post", "id": "5" },
      }
    }
  }
}
```
`id`为`1`的`comment`关联了`ID`为`5`的`post`。

#### 8，自定义转换规则

在某些情况下，Ember内置的属性类型（`string`、`number`、`boolean`、`date`）还是不够用的。比如，服务器返回的是非标准的数据格式时。

Ember Data可以注册新的JSON转换器去格式化数据，可用直接使用命令创建：`ember g transform coordinate-point`
```js
//  app/transforms/coordinate-point.js

import DS from 'ember-data';

export default DS.Transform.extend({
  deserialize: function(v) {
    return [v.get('x'), v.get('y')];
  },

  serialize: function(v) {
    return Ember.create({ x: v[0], y: v[1]});
  }
});
```
定义一个复合属性类型，这个类型由两个属性构成，形成一个坐标。
```js
//  app/models/curor.js
import DS from 'ember-data';
export default DS.Model.extend({
  position: DS.attr(‘coordinate-point’)
});
```
自定义的属性类型使用方式与普通类型一致，直接作为`attr`方法的参数。最后当我们接受到服务返回的数据形如下面的代码所示：
```js
{
  cursor: {
    position: [4, 9]
  }
}
```
加载模型实例时仍然作为一个普通对象加载。仍然可以使用`.`操作获取属性值。
```js
var cursor = this.store.findRecord(‘cursor’, 1);
cursor.get(‘position.x’);  //  => 4
cursor.get(‘position.y’);  //  => 9
```
#### 9，JSONSerializer

并不是所有的API都遵循JSONAPISerializer约定通过数据命名空间和拷贝关系记录。比如系统遗留问题，原先的API返回的只是简单的JSON格式并不是JSONAPISerializer约定的格式，此时你可以自定义序列化器去适配旧接口。并且可以同时兼容使用RESTAdapter去序列号这些简单的JSON数据。
```js
//  app/serializer/application.js

export default DS.JSONSerializer.extend({
  // ...
});
```
#### 10，EMBEDDEDRECORDMIXIN

尽管Ember Data鼓励你拷贝模型关联关系，但有时候在处理遗留API时，你会发现你需要处理的JSON中嵌入了其他模型的关联关系。不过`EmbeddedRecordsMixin`可以帮你解决这个问题。

比如`post`中包含了一个`author`记录。
```js
{
    "id": "1",
    "title": "Rails is omakase",
    "tag": "rails",
    "authors": [
        {
            "id": "2",
            "name": "Steve"
        }
    ]
}
```
你可以定义里的模型关联关系如下：
```js
//  app/serializers/post.js
export default DS.JSONSerialier.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
author: {
  serialize: ‘records’,
  deserialize: ‘records’
}
  }
});
```
如果你发生对象本身需要序列化与反序列化嵌入的关系，你可以使用属性`embedded`设置。
```js
//  app/serializers/post.js
export default DS.JSONSerialier.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
author: { embedded: ‘always’ }
  }
});
```
序列化与反序列化设置有3个关键字：

1. `records`  用于标记全部的记录都是序列化与反序列化的
2. `ids`  用于标记仅仅序列化与反序列化记录的id
3. `false`  用于标记记录不需要序列化与反序列化

例如，你可能会发现你想读一个嵌入式记录提取时一个JSON有效载荷只包括关系的身份在序列化记录。这可能是使用`serialize: ids`。你也可以选择通过设置序列化的关系 `serialize: false`。
```js
export default DS.JSONSerializer.extend(DS.EmbeddedRecordsMixin, {
  attrs: {
    author: {
      serialize: false,
      deserialize: 'records'
    },
    comments: {
      deserialize: 'records',
      serialize: 'ids'
    }
  }
});
```
#### 11，EMBEDDEDRECORDSMIXIN 默认设置

如果你没有重写`attrs`去指定模型的关联关系，那么`EmbeddedRecordsMixin`会有如下的默认行为：
```
belongsTo：{serialize: ‘id’, deserialize: ‘id’ }
hasMany: { serialize: false, deserialize: ‘ids’ }
```

#### 12，创作序列化器

如果项目需要自定义序列化器，Ember推荐扩展JSONAIPSerializer或者JSONSerializer来实现你的需求。但是，如果你想完全创建一个全新的与JSONAIPSerializer、JSONSerializer都不一样的序列化器你可以扩展`DS.Serializer`类，但是你必须要实现下面三个方法：

1. normalizeResponse
2. serialize
3. normalize

知道规范化JSON数据对Ember Data来说是非常重要的，如果模型属性名不符合Ember Data规范这些属性值将不会自动更新。如果返回的数据没有在模型中指定那么这些数据将会被忽略。比如下面的模型定义，`this.store.push()`方法接受的格式为第二段代码所示。
```js
//   app/models/post.js
import DS from 'ember-data';
export default DS.Model.extend({
  title: DS.attr(‘string’),
  tag: DS.attr(‘string’),
  comments: hasMany(‘comment’, { async: true }),
  relatedPosts: hasMany(‘post’)
});
```
```js
{
  data: {
    id: "1",
    type: 'post',
    attributes: {
      title: "Rails is omakase",
      tag: "rails",
    },
    relationships: {
      comments: {
        data: [{ id: "1", type: 'comment' },
               { id: "2", type: 'comment' }],
      },
      relatedPosts: {
        data: {
          related: "/api/v1/posts/1/related-posts/"
        }
      }
    }
}
```
每个序列化记录必须按照这个格式要正确地转换成Ember Data记录。
<br>
本篇的内容难度很大，属于高级主题的内容！如果暂时理解不来不要紧，你可以先使用[firebase](http://www.firebase.com)构建项目，等你熟悉了整个Ember流程以及数据是如何交互之后再回过头看这篇和上一篇[Ember.js 入门指南之四十四自定义适配器](http://blog.ddlisting.com/2016/04/17/zi-ding-yi-gua-pei-qi/)，这样就不至于难以理解了！！
<br>
到本篇为止，有关Ember的基础知识全部介绍完毕！！！从2015-08-26开始到现在刚好2个月，原计划是用3个月时间完成的，提前了一个月，归其原因是后面的内容难度大，理解偏差大！文章质量也不好，感觉时间比较仓促，说以节省了很多时间！（_本篇是重新整理发表的，原始版博文发布的时候Ember还是2.0版本，现在已经是2.5了！！_）
<br>
介绍来打算介绍APPLICATION CONCERNS和TESTING这两章！也有可能把旧版的Ember todomvc案例改成Ember2.0版本的，正好可以拿来练练手速！！！ 
<br>
很庆幸的是目标：把旧版的Ember todomvc案例改成Ember2.0版本的，也完成了！！！并且扩展了很多功能，有关代码情况[todos v2](https://github.com/ubuntuvim/todos_v2)，欢迎读者fork学习！如果觉得有用就给我一个`star`吧！！谢谢！！！

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
