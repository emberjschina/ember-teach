> 元数据是数据与一个特定的模式或类型，而不是一个纪录。

一个很常见的例子是分页。通常会像下面的代码设置分页：
```js
let result = this.store.query(‘post’, {
  limit: 10,
  offset: 0
});
```
设置了每页显示数据为10条，但是你不知道总条数，又怎么知道一共有多少页呢？这时候元数据就派上用场了。
```js
{
  "post": {
    "id": 1,
    "title": "Progressive Enhancement is Dead",
    "comments": ["1", "2"],
    "links": {
      "user": "/people/tomdale"
    },
    // ...
  },

  "meta": {
    "total": 100
  }
}
```
这些数据是从后台返回的[JSON](http://www.json.org)格式数据，如果你想获取元数据可以使用`this.get('meta')`获取。甚至还可以从`query()`方法中获取。

`let` 和 `=>` 都是[javascript ES6](http://es6.ruanyifeng.com/)的语法，如果你想了解有关javascript ES6请Google。

对于元数据在项目中的使用会在后面的例子中展现。在介绍完[Ember](http://emberjs.com)基础知识后我回做一个比较完整的小项目，我会在项目中尽可能的使用所讲过的知识点，敬请期待……
_小项目代码：[todos](https://github.com/ubuntuvim/todos_v2)_

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
