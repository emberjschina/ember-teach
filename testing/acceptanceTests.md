# 验收测试
使用`ember generate acceptance-test <name>`创建一个验收测试,将会创建一个如下的测试文件:
```javascript
//tests/acceptance/login-test.js

import { test } from 'qunit';
import moduleForAcceptance from 'people/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | login');

test('visting /login', function(assert) {
  visit('/login');

  andThen(function() {
  　assert.equal(currentURL(), '/login');
  });
});
```
`moduleForAcceptance`用来处理程序的初始化设置和teardown.最后几行`test`中包含了一个示例.

几乎所有的测试都有一个路由请求,用于和页面交互(通过helper)并检查DOM是否按照期望值进行改变.

举个例子:
```javascript
test('should add new post', function(assert) {
  visit('/posts/new');
  fillIn('input.title', 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text(), 'My new post'));
});
```
大体意思为:
进入`创建博客页面`,在`title标题`栏中填入`My new post`,点击`提交`,期望的结果是: 在对应列表下`ul.posts li.first`的文本为`My new post`.

## 测试助手
在测试web应用中的一个主要的问题是,由于代码都是基于事件驱动的,因此他们有可能是`异步`的,会使代码`无序`运行.

>比如有两个按钮,从不同的服务器载入数据,我们先后点击他们,但可能结果返回的顺序并不是我们点击的顺序.

当你在编写测试的时候,你需要特别注意一个问题,就是你无法确定在发出一个请求后,是否会立刻得到返回的响应.因此,你的断言需要以同步的状态来等待被测试体.
> 例如上面所举的例子,应该等待两个服务器均返回数据后,这时测试代码才会执行其逻辑来检测数据的正确性.

这就是为什么在做断言的时候,Ember测试助手都是被包裹在一个确保同步状态的代码中.这样做避免了对所有这样的代码都去做这样的包裹,并且因为减少了模板代码,从而提高了代码的可读性.

Ember包含多个`测试助手`来辅助进行`验收测试`.一共有2种类型:`同步测试助手`和`异步测试助手`

### 异步测试助手

异步测试助手可以意识到程序中的异步行为,使你可以更方便的编写确切的测试.

同时,这些测试助手会按顺序执行,当前一个调用结束之后,才会执行下一个.
* `click(selector)`
    点击一个元素,执行该元素所绑定的`click`事件,当事件完成后,返回`RSVP.promise`.
* `fillIn(selector, value)`
    选择一个`input`元素,填写`value`,返回`RSVP.promise`
* `keyEvent(selector, type, keyCode)`
    模拟键盘操作:`按键keypress`,`按键按下keydown`,`按键弹起keyup`
* `triggerEvent(selector,type,keyCode)`
    在指定元素上激活事件,比如`双击`等...
* `visit(url)`
    访问某个地址,返回`RSVP.promise`

### 同步测试助手
同步测试助手,当触发时会立即执行并返回结果.
* `currentPath()`
    返回当前路径
* `currentRouteName()`
    返回当前激活的`路由名称`
* `currentURL()`
    返回当前URL
* `find(selector, context)`
    寻找应用根元素中的指定下上文(可选)的元素.将其域定义到根元素可以有效的避免与测试框架的冲突.当上下文没有指定的时候,会按照默认方式完成.

### 等待助手
 `andThen`测试助手将会等待所有异步测试助手完成之后再执行.举个例子:
 ```javascript
 //tests/acceptance/new-post-appears-first-test.js

tese('should add new post', function(assert) {
  visit('/posts/new');
  fillIn('input.title', 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text(), 'My new post'));
  });
```

首先,我们访问`/posts/new`地址,在`title`css类的`input`输入框内内填入`My new post`,然后点击`提交`按钮.

当上面那些请求全部完成之后,才会执行`andThen`测试助手.我们最终使用`assert.equal`断言来判定对应位置的值是否为`My new post`.

### 自定义测试助手
使用`ember generate test-helper <helper-name>`来创建自定义测试助手.
举个例子:
```javascript
//ember generate test-helper shouldHaveElementWithCount
//tests/helpers/should-have-element-with-count.js

export default Ember.Test.registerAsyncHelper(
  'shouldHaveElementWithCount', function(app){
  });
```

`Ember.Test.registerAsyncHelper`和`Ember.Test.registerHelper'是当`startApp`被调用时,会将自定义测试助手注册.两者的区别在于,前者`AsyncHelper`会在之前任何的异步测试助手运行完成之后运行,并且后续的异步测试助手在运行前都会等待他完成.

测试助手方法一般都会以当前程序的第一个参数被调用,其他参数,比如`assert`,在测试助手被调用的时候,需要被提供. 测试助手需要在调用`startApp`前进行注册,但`ember-cli`会帮你处理.

下面是一个非异步的测试助手:
```javascript
//tests/helpers/should-have-element-with-count.js

export default Ember.Test.registerHelper('shouldHaveElementWithCount',
  function(app, assert, selector, n, context){
    const el = findWithAssert(selector, context);
    const count = el.length;
    assert.equal(n, count, 'found ${count} times');
  }
);

//shouldHaveElementWithCount(assert, 'ul li', 3);
```

下面是一个异步的测试助手:
```javascript
export default Ember.Test.registerAsynHelper('dblclick',
  function(app, assert, selector, context){
    let $el = findWithAssert(selector, context);
    Ember.run(() => $el.dblclick());
  }
);

//dblclick(assert, '#persion-1')
```

异步测试助手也可以让你将多个测试助手合并为一个.举个例子:
```javascript
//tests/helpers/add-contact.js
export default Ember.Test.registerAsyncHelper('addContact',
  function(app,name) {
    fillIn('#name', name);
    click('button.create');
  }
);

//addContact('Bob');
//addContact('Dan');
```

最后, 别忘了将你的测试助手添加进`tests/.jshintrc`和`tests/helpers/start-app.js`中.在`tests/.jshintrc`中,你需要将其添加进`predef`块中,不然就会得到`jshint`测试失败的消息.

```bash
{
  "predef": [
    "document",
    "window",
    "locaiton",
    ...
    "shouldHaveElementWithCount",
    "dblclick",
    "addContact"
  ],
  ...
}
```

在`tests/helpers/start-app.js`中,你需要引入测试助手文件,他们将会被注册
```javascript
import Ember from 'ember';
import Application from '../../app';
import Router from '../../router';
import config from '../../config/environmnet';
import './should-have-element-with-count';
import './dblclick';
import './add-contact';
```
