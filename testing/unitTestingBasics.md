# 单元测试基础
单元测试一般被用来测试一些小的代码块,并确保他们能发挥应有的效果.不像验收测试,单元测试被限定在一个域内并且不需要Emeber程序运行.

## 测试计算属性
看了例子:
```javascript
//app/models/somt-thing.js

export default Ember.Object.extend({
  foo: 'bar',

  computedFoo: Ember.compuuted('foo',function() {
    const foo = this.get('foo');
    return 'computed ${foo}`;
  })
});
```

在测试中,我们建立一个实例,然后更新`foo`的值,然后给出一个符合预期的`断言`:
```javascript
//tests/unit/models/some-thing-test.js

import {moduleFor, test} from 'ember-qunit';

moduleFor('model:some-thing', 'Unit | some thing', {
  unit: true
});

test('should correctly concat foo', function(assert) {
  const someThing = this.subject();
  somtThing.set('foo', 'baz');
  assert.equal(someThing.get('computedFoo'), 'computed baz');
});
```

`moduleFor`是由`Ember-Qunit`提供的单元测试助手.这些测试助手提供了很多便利,比如`subject`功能,他可以寻找并实例化测试所用的对象.同时`subject`方法还可接受参数用以初始化对象内的字段,比如`this.subject({foo: 'bar'});`

## 测试对象方法
下面让我们来看一下如何测试对象方法的逻辑.
```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  foo: 'bar',
  testMethod() {
    this.set('foo','baz');
  }
});
```

要对其进行测试,我们先创建一个实例,然后调用`testMethod`方法,然后给出断言.
```javascript
//tests/unit/models/some-thing-test.js

test('should update foo on testMethod', function(assert) {
  const someThing = this.subject();
  someThing.testMethod();
  assert.equal(someThing.get('foo'),'baz');
});
```

如果一个对象方法返回的是一个值,你可以很容易的给予断言进行判定.假设我们的对象拥有一个`calc`方法,它可以根据对象内的某些状态来决定返回的值.
```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  count: 0,
  calc() {
    this.incrementProperty('count');
    let count = this.get('count');
    return 'count: ${count}`;
  }
});
```

测试中需要调用`calc`方法,并且断言其返回值.
```javascript
//tests/unit/models/some-thing-test.js

test('should return incremented count on calc', function(assert) {
  const someThing = this.subject();
  assert.equal(someThing.calc(), 'count: 1');
  assert.equal(someThing.calc(), 'count: 2');
});
```

## 测试观察者
假设我们有一个对象拥有一些属性,并且又一个方法在监视着这些属性.
```javascript
//app/models/some-thing.js

export default Ember.Object.extend({
  foo: 'bar',
  other: 'no',
  doSomething: Ember.observer('foo', function() {
    this.set('other', 'yes');
  })
});
```

为了测试`doSomething`方法,我们创建一个`someThing`对象,更新`foo`属性字段,然后进行断言.
```javascript
//tests/unit/models/some-thing-test.js

test('should set other prop to yes when foo changes', function(assert) {
  const someThing = this.subject();
  someThing.set('foo', 'baz');
  assert.equal(someThing.get('other'), 'yes');
 });
```
