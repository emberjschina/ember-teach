Ember提供的表单元素都是经过封装的，封装成了`view`组件。经过解析渲染之后就会生成普通的HTML标签。更多详细信息你可以查看他们的实现源码：[Ember.TextField](https://github.com/emberjs/ember.js/blob/v2.0.1/packages/ember-views/lib/views/text_field.js#L36)、[Ember.Chechbox](https://github.com/emberjs/ember.js/blob/v2.0.1/packages/ember-views/lib/views/checkbox.js#L10)、[Ember.TextArea](https://github.com/emberjs/ember.js/blob/v2.0.1/packages/ember-views/lib/views/text_area.js#L8)。

按照惯例，先创建一个`route`：<br>
`ember generate route form-helper`。

### 1，`input`助手

```html
{{! //app/templates/form-helper.hbs }}
{{input name="username" placeholder="your name"}}
```

其中可以使用在`input`助手上的属性有很多，包括`readonly`、`value`、`disabled`、`name`等等，更多有关的属性介绍请移步[官网介绍](guides.emberjs.com/v2.0.0/templates/input-helpers/)。
<br>**注意：对于使用在`input`助手上的属性是不是使用双引号括住是有区别的。比如`value='helloworld'`和`value=helloworld`渲染之后的结果是不一样的，第一种写法是直接把"helloworld"这个字符串赋值设置到`value`上，第二种写法是从上下文获取变量helloworld的值再设置到`value`上，通常是在`controller`或者`route`设置的值。**
看下面2行代码的演示结果：
```html
{{input name="username" placeholder="your name" value="model.helloworld"}}
<br><br>
{{input name="username" placeholder="your name" value=model.helloworld}}
```
修改对应的`route`类，重写`model`回调，返回一个字符串；或者你可以在模板对应的`controller`类设置。比如下面的第二段代码（使用命令`ember generate controller form-helper`得到模板对应的`controller`类。
）。
```javascript
// app/routes/form-helper.js

import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {
		return { helloworld: 'The value from route...' }
	}
});
```

在`controller`类初始化测试数据。
```javascript
// app/controllers/form-helper.js

import Ember from 'ember';

export default Ember.Controller.extend({
	helloworld: 'The value from route...'
});
```
对应的，如果你使用的是`controller`初始化测试数据，那么你的模板获取数据的方式就要稍微修改下。需要去掉前缀`model.`。`controller`不需要在回调中初始化测试数据，可用直接定义成`controller`的属性。
```html
{{input name="username" placeholder="your name" value=helloworld}}
```

![运行结果](/content/images/2016/03/40.png)

### 2，在`input`助手上指定触发事件

你可以想想下，我们平常写过的javascript代码，不是可用直接在`input`输入框上使用javascript的函数，同理的，`input`助手上可以使用javascript函数，不过使用方式有点差别，请看下面示例。比如按`enter`键触发指定的事件、失去焦点触发事件等等。
首先编写`input`输入框，获取`input`输入框的值有点不按常理=^=。在`controller`类获取`input`输入框的值是通过不用双引号的`value`属性获取的。
```html
按enter键触发
{{input value=getValueKey enter="getInputValue" name=getByName placeholder="请输入测试的内容"}}
```
```javascript
// app/controllers/form-helper.js

import Ember from 'ember';

export default Ember.Controller.extend({
	actions: {
		getInputValue: function() {
			var v = this.get('getValueKey');
			console.log('v = ' + v);

			var v2 = this.get('getByName');
			console.log('v2 = ' + v2);
		}
	}
});
```
输入测试内容后按`enter`键。

![run result](/content/images/2016/03/41.png)

最后的输出结果有那么一点点意外。`v`的值是正确的，`v2`却是`undefined`。可见在`controller`层获取页面的值是通过`value`这个属性而不是`name`这个属性。跟我们平常HTML的`input`有点不一样了！！这个需要注意下。

### 3，`checkbook`助手

`checkbox`这个表单元素也是经过Ember封装了，作为一个组件使用。使用过程需要注意的问题与前面的`input`是一样的，属性是不是使用双引号所起的作用是不一样的。
```html
checkbox{{input type="checkbox" checked=isChecked }}
```
你可以在`controller`增加一个属性`isChecked`并设置为`true`，`checkbox`将默认为选中。
```javascript
// app/controllers/form-helper.js

import Ember from 'ember';

export default Ember.Controller.extend({
	actions: {
		// ……
	},
	isChecked: true
});
```

![result](/content/images/2016/03/42.png)

### 4，`textarea`助手

```html
{{textarea value=key cols="80" rows="3" enter="getValueByV"}}
```
同样的也是通过`value`属性获取输入的值。

本篇简单的介绍了常用的表单元素，使用的方式比较简单，唯一需要注意的是获取的输入框输入值的方式与平常使用的HTML表单元素有点差别。其他的基本上与普通的HTML表单元素没什么差别。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
