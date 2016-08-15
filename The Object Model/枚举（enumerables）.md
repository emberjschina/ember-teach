在`Ember`中，枚举是包含多个子对象的对象，并且提供了丰富的API（[Ember.Enumerable API](http://emberjs.com/api/classes/Ember.Enumerable.html)）去获取所包含的子对象。`Ember`的枚举都是基于原生的`javascript`数组实现的，`Ember`扩展了其中的很多接口。
`Ember`提供一个标准化接口处理枚举，并且允许开发者完全改变底层数据存储，而无需修改应用程序的数据访问代码。
`Ember`的`Enumerable API`尽可能的遵照`ECMAScript`规范。为了减少与其他库不兼容，`Ember`允许你使用本地浏览器实现数组。

下面是一些重要而常用的`API`列表；请注意左右两列的不同。
<table border="1">
  <tr bgcolor="#ccc">
      <td>标准方法</td>
      <td>可被观察方法</td>
      <td>说明</td>
  </tr>
  <tr>
    <td>pop</td>
    <td>popObject</td>
    <td>该函数从从数组中删除最后项，并返回该删除项</td>
  </tr>
  <tr>
    <td>push</td>
    <td>pushObject</td>
    <td>新增元素</td>
  </tr>
 <tr>
    <td>reverse</td>
    <td>reverseObject</td>
    <td>颠倒数组元素</td>
  </tr>
<tr>
    <td>shift</td>
    <td>shiftObject</td>
    <td>把数组的第一个元素从其中删除，并返回第一个元素的值</td>
  </tr>
<tr>
    <td>unshift</td>
    <td>unshiftObject</td>
    <td>可向数组的开头添加一个或更多元素，并返回新的长度</td>
  </tr>
</table>

详细文档请看：[http://emberjs.com/api/classes/Ember.Enumerable.html](http://emberjs.com/api/classes/Ember.Enumerable.html)

在列表上右侧的方法是`Ember`重写标准的`JavaScript`方法而得的，他们最大的不同之处是使用普通的方法（左边部分）操作的数组不会在你的应用程序中自动更新（不会触发观察者），而使用`Ember`重写过的方法则可以触发观察者，只要你的数据有变化`Ember`就可以观察到，并且更新到模板上。

## 常用API

#### 1，数组迭代器

遍历数组元素使用`forEach`方法。

```javascript
var arr = ['chen', 'ubuntuvm', '1527254027@qq.com', 'i2cao.xyz', 'ubuntuvim.xyz'];
arr.forEach(function(item, index) {
  console.log(index+1 + ", " +item);
});
```

#### 2，获取数组首尾元素

```javascript
//  获取头尾的元素，直接调用Ember封装好的firstObject和lastObject方法即可
console.log('The firstItem is ' + arr.get('firstObject'));  // output> chen
console.log('The lastItem is ' + arr.get('lastObject'));  //output> ubuntuvim.xyz
```

#### 3，map方法

```javascript
//  map方法，转换数组，并且可以在回调函数里添加自己的逻辑
//  map方法会新建一个数组，并且返回被转换数组的元素
var arrMap = arr.map(function(item) {
  return 'map: ' + item;  //  增加自己的所需要的逻辑处理
});
arrMap.forEach(function(item, index) {
  console.log(item);
});
console.log('-----------------------------------------------');
```

#### 4，mapBy方法

```javascript
// mapBy 方法：返回对象属性的集合，
// 当你的数组元素是一个对象的时候，你可以根据对象的属性名获取对应值
var obj1 = Ember.Object.create({
  username: '123',
  age: 25
});
 
var obj2 = Ember.Object.create({
  username: 'name',
  age: 35
});
var obj3 = Ember.Object.create({
  username: 'user',
  age: 40
});
 
var obj4 = Ember.Object.create({
  age: 40
});
 
var arrObj = [obj1, obj2, obj3, obj4];  //对象数组
var tmp = arrObj.mapBy('username');  // 
 
tmp.forEach(function(item, index) {
  console.log(index+1+", "+item);
});
 
console.log('-----------------------------------------------');
```


#### 5，filter方法

```javascript
//  filter 过滤器方法，过滤普通数组元素
//  filter方法可以跟你指定的条件过滤掉不匹配的数据，比如下面的代码：过滤了元素大于4的元素
var nums = [1, 2, 3, 4, 5];
//  参数self值数组本身
var numsTmp = nums.filter(function(item, index, self) {
    return item < 4;
});
 
numsTmp.forEach(function(item, index) {
  console.log('item = ' + item);  //  1， 2， 3
});
console.log('-----------------------------------------------');
```

`filter`方法会返回所有判断为`true`的元素，会把判断结果为`false`或者`undefined`的元素过滤掉。

#### 6，filterBy方法

```javascript
//  如果你想根据对象的某个属性过滤数组你需要用filterBy方法，比如下面的代码根据isDone这个对象属性过滤
var o1 = Ember.Object.create({
  name: 'u1',
  isDone: true
});
 
var o2 = Ember.Object.create({
  name: 'u2',
  isDone: false
});
 
var o3 = Ember.Object.create({
  name: 'u3',
  isDone: true
});
 
var o4 = Ember.Object.create({
  name: 'u4',
  isDone: true
});
 
var todos = [o1, o2, o3, o4];
var isDoneArr = todos.filterBy('isDone', true);  //会把o2过滤掉
isDoneArr.forEach(function(item, index) {
  console.log('name = ' + item.get('name') + ', isDone = ' + item.get('isDone'));
  // console.log(item);
});
 
console.log('-----------------------------------------------');
```

`filter`和`filterBy`不同的地方是前者可以自定义过滤逻辑，后者可以直接使用。

#### 7，every、some方法

```javascript
// every、some 方法
// every 用于判断数组的所有元素是否符合条件，如果所有元素都符合指定的判断条件则返回true，否则返回false
// some 用于判断数组的所有元素只要有一个元素符合条件就返回true，否则返回false
Person = Ember.Object.extend({
  name: null,
  isHappy: true
});
var people = [
  Person.create({ name: 'chen', isHappy: true }),
  Person.create({ name: 'ubuntuvim', isHappy: false }),
  Person.create({ name: 'i2cao.xyz', isHappy: true }),
  Person.create({ name: '123', isHappy: false }),
  Person.create({ name: 'ibeginner.sinaapp.com', isHappy: false })
];
var every = people.every(function(person, index, self) {
  if (person.get('isHappy'))
    return true;
});
console.log('every = ' + every);
 
var some = people.some(function(person, index, self) {
  if (person.get('isHappy'))
    return true;
});
console.log('some = ' + some);
```

#### 8，isEvery、isAny方法

```javascript
//  与every、some类似的方法还有isEvery、isAny 
console.log('isEvery = ' + people.isEvery('isHappy', true));  //  全部都为true，返回结果才是true
console.log('isAny = ' + people.isAny('isHappy', true));  //只要有一个为true，返回结果就是true
```

上述方法的使用与普通`JavaScript`提供的方法基本一致。学习难度不大……自己敲两边就懂了！

这些方法非常重要，请一定要学会如何使用！！！
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用在github项目上给我个`star`吧。您的肯定对我来说是最大的动力！！