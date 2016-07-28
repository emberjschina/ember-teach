正如其他的框架一样，`Ember`也有它特有的数据绑定方式，并且可以在任何一个对象上使用绑定。而然，数据绑定大多数情况都是使用在`Ember`框架本身，对于开发者最好还是使用计算属性更为简单方便。

### 双向绑定

```javascript
// 双向绑定
Wife = Ember.Object.extend({
  householdIncome: 800
});
var wife = Wife.create();

Hasband = Ember.Object.extend({
  //  使用 alias方法实现绑定
  householdIncome: Ember.computed.alias('wife.householdIncome')
});

hasband = Hasband.create({
  wife: wife
});

console.log('householdIncome = ' + hasband.get('householdIncome'));  //  output > 800
// 可以双向设置值

//  在wife方设置值
wife.set('householdIncome', 1000);
console.log('householdIncome = ' + hasband.get('householdIncome'));  // output > 1000
// 在hasband方设置值
hasband.set('householdIncome', 10);
console.log('wife householdIncome = ' + wife.get('householdIncome'));
```

![run result](/content/images/2016/03/13.png)

需要注意的是绑定并不会立刻更新对应的值，`Ember`会等待直到程序代码完成运行完成并且是在同步改变之前，所以你可以多次改变计算属性的值。由于绑定是很短暂的所以也不需要担心开销问题。

### 单向绑定

单向绑定只会在一个方向上传播变化。相对双向绑定来说，单向绑定做了性能优化，对于双向绑定来说如果你只是在一个方向上设置关联其实就是一个单向绑定。

```javascript
var user = Ember.Object.create({
  fullName: 'Kara Gates'
});

UserComponent = Ember.Component.extend({
  userName: Ember.computed.oneWay('user.fullName')
});

userComponent = UserComponent.create({
  user: user
});

console.log('fullName = ' + user.get('fullName'));
// 从user可以设置
user.set('fullName', "krang Gates");
console.log('component>> ' + userComponent.get('userName'));
// UserComponent 设置值，user并不能获取，因为是单向的绑定
userComponent.set('fullName', "ubuntuvim");
console.log('user >>> ' + user.get('fullName'));
```

![run result](/content/images/2016/03/14.png)

关于数据绑定的知识点不多，相对来说不是重点，毕竟对象之间的关联关系是越少、越简单越好。关联关系多了反而难以维护。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用在github项目上给我个`star`吧。您的肯定对我来说是最大的动力！！