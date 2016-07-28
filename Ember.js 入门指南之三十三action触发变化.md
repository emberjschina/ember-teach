组件就像一个相对独立的盒子。在前面的文章中介绍过组件是怎么通过属性传递参数，并且这个属性值你可以在模板或者js代码中获取。

但是到目前为止还没介绍过子组件从父组件中获取数组，在[Ember](http://emberjs.com)应用中组件之间的通信是通过`actions`实现的。

跟着下面的步骤来，创建一个组件之间通信的示例。

#### 1，创建组件

创建组件的方法不用我多说，直接使用[Ember CLI](http://ember-cli.com/user-guide)命令创建即可。
```
ember g component button-with-confirmation
ember g component user-profile
ember g route button-with-confirmation-route
```
为了测试方便多增加了一个路由。

下面是组件`user-profile`的定义，调用组件`button-with-confirmation`，那么此时`user-profile`作为父组件，`button-with-confirmation`作为子组件：
```html
<!--  app/temlates/components/user-profile.hbs  -->

{{button-with-confirmation text="Click OK to delete your account"}}
```
#### 2，在组件类中添加action

要想`action`能执行需要作如下两步：

* 在父组件中定义好需要处理的动作（action）。在这个例子中父组件的动作是要删除用户账号并发送一个提示信息到另一个组件。
* 在子组件中，判断发生什么事件并发出通知信息。在这个例子中当用户点击“确认”按钮之后触发一个外部的动作（删除账户或者发送提示信息）。

下面是实现代码：

**实现父组件动作（action）**

在父组件中，定义好当用户点击“确认”之后触发的动作。在这个例子中的动作（`action`）是先找出用户账号再删除。

在[Ember](http://emberjs.com)应用中，每个组件都有一个名为`actions`的属性。这个属性值是函数，是可以被用户或者子组件执行的函数。
```js
//  app/components/user-profile.js

import Ember from 'ember';

export default Ember.Component.extend({
	actions: {
		userDidDeleteAccount: function() {
			console.log(“userDidDeleteAccount…”);
		}
	}
});
```
现在已经实现了父组件逻辑，但是并没有告诉Ember这个动作什么时候触发，下一步将实现这个功能。

**实现子组件动作（action）**


这一步我们将实现当用户点击“确定”之后触发事件的逻辑。
```js
//  app/components/button-with-confirmation.js

import Ember from 'ember';

export default Ember.Component.extend({
	tagName: 'button',
	click: function() {
		if (confirm(this.get('text'))) {
			// 在这里获取父组件的事件（数据）并触发
		}
	}
});
```

#### 3，传递action到组件中

现在我们在`user-profile`组件中使用`onConfirm()`方法触发组件`button-with-confirmation`类中的`userDidDeleteAccount`事件。
```html
<!--  app/temlates/components/user-profile.hbs  -->

{{#button-with-confirmation text="Click OK to delete your account" onConfirm=(action 'userDidDeleteAccount')}}
执行userDidDeleteAccount方法
{{/button-with-confirmation}}
```
这段代码的意思是告诉父组件，`userDidDeleteAccount`方法会通过`onConfirm`方法执行。

现在你可以在子组件中使用`onConfirm`方法执行父组件的动作。
```js
//  app/components/button-with-confirmation.js

import Ember from 'ember';

export default Ember.Component.extend({
	tagName: 'button',
	click: function() {
		if (confirm(this.get('text'))) {
			// 在父组件中触发动作
			this.get('onConfirm')();
		}
	}
});
```
`this.gete(“onConfirm”)`从父组件返回一个值`onConfirm`，然后与`()`组合成了一个个方法`onConfirm()`。

在模板`button-with-confirmation-route.hbs`中调用组件。
```html
<!--  app/temlates/button-with-confirmation-route.hbs  -->

{{user-profile}}
```

![结果](/content/images/2016/04/128.png)

点击这个`button`，会触发事件。弹出对话框。再点击“确认”后执行方法`userDidDeleteAccount`，可以看到浏览器控制台输出了**userDidDeleteAccount…**，未点击“确认”前或者点击“取消”不会输出这个信息，说明不执行这个方法`userDidDeleteAccount`。

![结果截图](/content/images/2016/04/129.png)

![结果截图](/content/images/2016/04/130.png)

像普通属性，`actions`可以组件的一个属性，唯一的区别是，属性设置为一个函数，它知道如何触发的行为。

在组件的`actions`属性中定义的方法，允许你决定怎么去处理一个事件，有助于模块化，提高组件重用率。


到此，组件这一章节的内容全部介绍完毕了，不知道你看懂了多少？如果有疑问请给我留言一起交流学习，获取是直接去官网学习官方教程。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！






