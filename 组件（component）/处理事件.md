你可以在组件中响应事件，比如用户的双击、鼠标滑过、键盘的按下等等事件。只需要在组件类中增加[Ember](http://emberjs.com)提供的处理事件，然后[Ember](http://emberjs.com)会自动判断用户的操作执行相应的事件，只要在组件类中添加的事件不冲突你甚至一次性增加多个事件，事件执行次序根据触发的次序执行。

1，简单事件处理
准备工作，使用[Ember CLI](http://ember-cli.com/user-guide)创建演示所需文件：
```
ember g component handle-events
ember g route component-route
```
生成的组件模板不做任何修改。
```html
<!--  app/components/handle-events.hbs -->

{{yield}}
```
注意看组件类的实现：
```js
//  app/components/handle-events.js

import Ember from 'ember';

export default Ember.Component.extend({
	click: function() {
		alert('click...');

		return true;  // 返回true允许事件冒泡到父组件
	},
	mouseLeave: function() {
		alert("mouseDown....");

		return true;
	}
});
```
在组件类中增加了两个事件`click`和`mouseLeaver`，一个是单击事件一个是鼠标移开事件，更多[Ember](http://emberjs.com)支持的事件请看[handling-events](https://guides.emberjs.com/v2.4.0/components/handling-events/#toc_event-names)。

调用组件的模板如下：
```html
<!--  app/templates/component-route.hbs  -->

{{#handle-events}}
<span style="cursor: pointer;">从我身上飘过或者点我都会触发事件~</span>
{{/handle-events}}
```
当用户只是把鼠标从文字“从我身上飘过或者点我都会触发事件~”上划过市只执行`mouseLeave`事件，当用户点击文字时先执行`click`事件再执行`mouseLeave`事件，因为用户点击文字时鼠标还没移开。

但是如果你增加的事件是有冲突的可能会得到无法预知的结果，比如在组件类中增加了双击和单击事件，此时只会执行单击事件，双击事件就无法触发。

## 2，发送行为

某些情况下，你的组件需要支持拖放事件。比如组件可能要发送一个`id`到`drop`事件中。
```html
{{drop-target action=”didDrop”}}
```
你可以定义组件的事件处理器去管理`drop`事件。如果有需要可以通过返回`false`防止事件冒泡。
```js
//  app/components/drop-target.js

import Ember from 'ember';

export default Ember.Component.extend({
	attribuBindings: ['draggable'],
	draggable: 'true',

	dragOver: function() {
		return false;
	},
	didDrop: function(event) {
		let id = event.dataTransfer.getData('text/data');
		this.sendAction('action', id);
	}
});
```
本章内容不多，重点是第一点的内容，第二点的内容就简单介绍，更多详细信息请移步[官网文档](guides.emberjs.com/v2.0.0/components/handling-events/)。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
