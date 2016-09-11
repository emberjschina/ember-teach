# 单元测试基础

单元测试一般被用来测试一些小的代码块，并确保它正在做的是什么。与验收测试不同的是，单元测试被限定在小范围内并且不需要Emeber程序运行。  

与Ember基本对象一样的，创建单元测试也只需要继承`Ember.Object`即可。然后在代码块内编写具体的测试内容，比如控制器、组件。每个测试就是一个`Ember.Object`实例对象，你可以设置对象的状态，运行断言。通过下面的例子，让我们一起看看测试如何使用。

## 测试计算属性

创建一个简单的实例，实例内包含一个计算属性`computedFoo`，此计算属性依赖普通属性`foot`。

```javascript
//app/models/somt-thing.js

export default Ember.Object.extend({
  foo: 'bar',

  computedFoo: Ember.compuuted('foo'，function() {
    const foo = this.get('foo');
    return `computed ${foo}`;
  })
});
```

在测试中，我们建立一个实例，然后更新属性`foo`的值（这个操作会触发计算属性`computedFoo`，使其自动更新），然后给出一个符合预期的`断言`:

```javascript
//tests/unit/models/some-thing-test.js

import {moduleFor， test} from 'ember-qunit';

moduleFor('model:some-thing'， 'Unit | some thing'， {
  unit: true
});

test('should correctly concat foo'， function(assert) {
  const someThing = this.subject();
  somtThing.set('foo'， 'baz');  //设置属性foo的值
  assert.equal(someThing.get('computedFoo'), 'computed baz');  //断言，判断计算属性值是否相等于computed baz
});
```

例子中使用了`moduleFor`，它是由`Ember-Qunit`提供的单元测试助手。这些测试助手为我们提供了很多便利，比如`subject`功能，它可以寻找并实例化测试所用的对象。同时你还可以在`subject`方法中自定义初始化的内容，这些初始化的内容可以是传递给主体功能的实例变量。比如在单元测试内初始化属性“foo”你可以这样做：`this.subject({foo: 'bar'});`，那么单元测试在运行时属性`foo`的值就是`bar`。

## 测试对象方法

下面让我们来看一下如何测试对象方法的逻辑。在本例中对象内部有一个设置属性（更新属性`foo`值）值的方法`testMethod`。

```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  foo: 'bar'，
  testMethod() {
    this.set('foo', 'baz');
  }
});
```

要对其进行测试，我们先创建如下实例，然后调用`testMethod`方法，然后用断言判断方法的调用结果是否是正确的。

```javascript
//tests/unit/models/some-thing-test.js

test('should update foo on testMethod'， function(assert) {
  const someThing = this.subject();
  someThing.testMethod();
  assert.equal(someThing.get('foo'), 'baz');
});
```

如果一个对象方法返回的是一个值，你可以很容易的给予断言进行判定是否正确。假设我们的对象拥有一个`calc`方法，方法的返回值是基于对象内部的状态值。代码如下：

```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  count: 0,
  calc() {
    this.incrementProperty('count');
    let count = this.get('count');
    return `count: ${count}`;
  }
});
```

在测试中需要调用`calc`方法，并且断言其返回值是否正确。

```javascript
//tests/unit/models/some-thing-test.js

test('should return incremented count on calc'， function(assert) {
  const someThing = this.subject();
  assert.equal(someThing.calc(), 'count: 1');
  assert.equal(someThing.calc(), 'count: 2');
});
```

## 测试观察者

假设我们有一个对象，这个对象拥有一些属性，并且有一个方法在监测着这些属性。

```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  foo: 'bar'
  other: 'no',,
  doSomething: Ember.observer('foo', function() {
    this.set('other', 'yes');
  })
});
```

为了测试`doSomething`方法，我们创建一个`SomeThing`对象，更新`foo`属性值，然后进行断言是否达到预期结果。

```javascript
//tests/unit/models/some-thing-test.js

test('should set other prop to yes when foo changes'， function(assert) {
  const someThing = this.subject();
  someThing.set('foo', 'baz');
  assert.equal(someThing.get('other'), 'yes');
 });
```
