# 验收测试

使用`ember generate acceptance-test <name>`创建一个验收测试，比如:

```
ember g acceptance-test login
```

执行完毕命令之后得到如下文件内容：

```javascript
//tests/acceptance/login-test.js

import { test } from 'qunit';
import moduleForAcceptance from 'people/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | login');

test('visting /login'， function(assert) {
  visit('/login');

  andThen(function() {
  　assert.equal(currentURL()， '/login');
  });
});
```

`moduleForAcceptance`用来启动、终止程序。最后几行`test`中包含了一个示例。

几乎所有的测试都有一个路由请求，用于和页面交互(通过helper)并检查DOM是否按照期望值进行改变。

举个例子:

```javascript
test('should add new post'， function(assert) {
  visit('/posts/new');
  fillIn('input.title'， 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text()， 'My new post'));
});
```

大体意思为:
进入路由`/posts/new`，在输入框`input.title`填入`My new post`，点击`button.submit`，期望的结果是: 在对应列表下`ul.posts li.first`的文本为`My new post`.

## 测试助手

在测试web应用中的一个主要的问题是，由于代码都是基于事件驱动的，因此他们有可能是`异步`的，会使代码`无序`运行。

比如有两个按钮，从不同的服务器载入数据，我们先后点击他们，但可能结果返回的顺序并不是我们点击的顺序。

当你在编写测试的时候，你需要特别注意一个问题，就是你无法确定在发出一个请求后，是否会立刻得到返回的响应。因此，你的断言需要以同步的状态来等待被测试体。例如上面所举的例子，应该等待两个服务器均返回数据后，这时测试代码才会执行其逻辑来检测数据的正确性。

这就是为什么在做断言的时候，Ember测试助手都是被包裹在一个确保同步状态的代码中。这样做避免了对所有这样的代码都去做这样的包裹，并且因为减少了模板代码，从而提高了代码的可读性.

Ember包含多个测试助来辅助进行验收测试。一共有2种类型:异步助手`asynchronous`和同步助手`synchronous`

### 异步测试助手

异步测试助手可以意识到程序中的异步行为，使你可以更方便的编写确切的测试。

同时，这些测试助手会按注册的顺序执行，并且是链式运行。每个测试助手的调用都是在前一个调用结束之后，才会执行下一个。因此，你可以不用当心测试助手的执行顺序。

* `click(selector)`
	* 点击一个元素，触发该元素所绑定的`click`事件，返回一个异步执行成功的`promise`。
* `fillIn(selector， value)`
    * 用执行成功的promise值填充到选中的input元素上。使用`<select>`作为`<input>`时，记得`<select>`元素的`value`是标签`<option>`所指定的值，并不是`<option>`标签显示的内容。
* `keyEvent(selector， type， keyCode)`
    * 模拟键盘操作。比如选中的元素检测按键`keypress`，按键按下`keydown`，按键弹起`keyup`事件的`keyCode`。
* `triggerEvent(selector，type，keyCode)`
    * 在指定元素上触发给定事件，比如`blur`、`ddlclick`等事件...
* `visit(url)`
    * 访问路由并返回promise执行成功的结果。

### 同步测试助手

同步测试助手，当触发时会立即执行并返回结果.

* `currentPath()`
	* 返回当前路径
* `currentRouteName()`
    * 返回当前激活的路由名称
* `currentURL()`
    * 返回当前URL
* `find(selector， context)`
    * 寻找应用根元素中的指定下上文(可选)的元素。将其域定义到根元素可以有效的避免与测试框架的冲突。当上下文没有指定的时候，会按照默认方式完成。

### 等待助手

 `andThen`测试助手将会等待所有异步测试助手完成之后再执行.举个例子:
 
 ```javascript
//  tests/acceptance/new-post-appears-first-test.js

tese('should add new post'， function(assert) {
  visit('/posts/new');
  fillIn('input.title'， 'My new post');
  click('button.submit');
  andThen(() => assert.equal(find('ul.posts li:first').text()， 'My new post'));
  });
```

首先，我们访问`/posts/new`地址，在有`title`css类的`input`输入框内内填入内容“My new post”，然后点击有CSS类`submit`的按钮。

等待签名的异步测试助手执行完（具体说，`andThen`会在路由`/posts/new`访问完毕，`input`表单填充完毕，按钮被点击之后）毕后会执行`andThen`助手。注意`andThen`助手只有一个参数，这个参数是一个函数，函数的代码是其实测试助手执行完毕之后才执行的代码。

在`andThen`助手中，我们最后会调用`assert.equal`断言来判定对应位置的值是否为`My new post`。

### 自定义测试助手

使用命令`ember generate test-helper <helper-name>`来创建自定义测试助手。

下面的代码是执行命令`ember g test-helper shouldHaveElementWithCount`得到的测试例子:

```javascript
//tests/helpers/should-have-element-with-count.js

export default Ember.Test.registerAsyncHelper(
    'shouldHaveElementWithCount'， function(app){}
);
```

`Ember.Test.registerAsyncHelper`和`Ember.Test.registerHelper'是当`startApp`被调用时，会将自定义测试助手注册。两者的区别在于，前者`Ember.Test.registerHelper`会在之前任何的异步测试助手运行完成之后运行，并且后续的异步测试助手在运行前都会等待他完成.

测试助手方法一般都会以当前程序的第一个参数被调用，其他参数，比如`assert`，在测试助手被调用的时候提供。测试助手需要在调用`startApp`前进行注册，但`ember-cli`会帮你处理，你不需要担心这个问题。

下面是一个非异步的测试助手:

```javascript
//tests/helpers/should-have-element-with-count.js

export default Ember.Test.registerHelper('shouldHaveElementWithCount'，
  function(app， assert， selector， n， context){
    const el = findWithAssert(selector， context);
    const count = el.length;
    assert.equal(n， count， 'found ${count} times');
  }
);

//shouldHaveElementWithCount(assert， 'ul li'， 3);
```

下面是一个异步的测试助手:
```javascript
export default Ember.Test.registerAsynHelper('dblclick'，
  function(app， assert， selector， context){
    let $el = findWithAssert(selector， context);
    Ember.run(() => $el.dblclick());
  }
);

//dblclick(assert， '#persion-1')
```

异步测试助手也可以让你将多个测试助手合并为一个.举个例子:

```javascript
//tests/helpers/add-contact.js
export default Ember.Test.registerAsyncHelper('addContact',
  function(app，name) {
    fillIn('#name', name);
    click('button.create');
  }
);

//addContact('Bob');
//addContact('Dan');
```

最后， 别忘了将你的测试助手添加进`tests/.jshintrc`和`tests/helpers/start-app.js`中、在`tests/.jshintrc`中，你需要将其添加进`predef`块中，不然就会得到jshint测试失败的消息.

```json
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
}*
```

你需要在`tests/helpers/start-app.js`引入测试助手，这些助手将会被注册到应用中。

```javascript
import Ember from 'ember';
import Application from '../../app';
import Router from '../../router';
import config from '../../config/environmnet';
import './should-have-element-with-count';
import './dblclick';
import './add-contact';
```
