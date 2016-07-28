`action`助手所现实的功能与`javascript`里的事件是相似的，都是通过用户点击元素触发定义在元素上的事件。Ember的action助手还允许你传递参数到对应的`controller`、`component`类，在`controller`或者`component`上处理事件的逻辑。
准备工作，我们使用[Ember CLI](http://ember-cli.com/user-guide)命令创建一个名称为`myaction`的`controller`和同名的`route`，如果你不知道怎么使用Ember CLI请看前面的文章[Ember.js 入门指南之七第一章对象模型小结](http://blog.ddlisting.com/2016/03/18/ember-js-ru-men-zhi-nan-zhi-qi-di-zhang-dui-xiang-mo-xing-xiao-jie/)，这篇文件讲解了怎么使用Ember CLI构建一个简单的Ember项目。

### 1，action使用实例
#### 1，在route层增加测试数据
```javascript
//  apap/routes/myaction.js


import Ember from 'ember';

export default Ember.Route.extend({
	//  返回测试数据到页面
	model: function() {
		return { id:1, title: 'ACTIONS', body: "Your app will often need a way to let users interact with controls that change application state. For example, imagine that you have a template that shows a blog title, and supports expanding the post to show the body.If you add the {{action}} helper to an HTML element, when a user clicks the element, the named event will be sent to the template's corresponding component or controller." };
	}
});
```
重写`model`回调，直接返回一个对象数据。

#### 2，编写action模板
```html
<!--  //  app/templates/myaction.hbs  -->

<!-- showDetailInfo这个事件的名字必须要跟controller里的方法名字一致 -->
<h2 {{action ' showDetailInfo '}} style="cursor: pointer;">{{model.title}}</h2>
{{#if isShowingBody}}
<p>
{{model.body}}
</p>
{{/if}}
```
默认下只显示文章的标题，当用户点击标题的时候触发事件`toggleBody`显示文章的详细信息。

#### 3，编写action的controller实现模板所需要的逻辑
```javascript
// app/controllers/myaction.js

import Ember from 'ember';

export default Ember.Controller.extend({

	//  控制页面文章详细内容是否显示
	isShowingBody: false,
	actions: {
		showDetailInfo: function() {
			// toggleProperty方法直接把isShowingBody设置为相反值
			// toggleProperty方法详情：http://devdocs.io/ember/classes/ember.observable#method_toggleProperty
			this.toggleProperty('isShowingBody');
		}
	}
});
```
对于`controller`的处理逻辑你还可以直接编写触发的判断。
```javascript
actions: {
		showDetailInfo: function() {
			// toggleProperty方法直接把isShowingBody设置为相反值
			// toggleProperty方法详情：http://devdocs.io/ember/classes/ember.observable#method_toggleProperty
			// this.toggleProperty('isShowingBody');
			
			// 变量作用域问题
			var isShowingBody = this.get('isShowingBody');
			if (isShowingBody) {
				this.set('isShowingBody', false);	
			} else {
				this.set('isShowingBody', true);
			}
		}
	}
```
如果你不使用`toggleProperty`方法改变`isShowingBody`的值，你也可用直接编写代码修改它的值。
最后执行URL：[http://localhost:4200/myaction](http://localhost:4200/myaction)，默认情况下页面上是不显示文章的详细信息的，当你点击标题则会触发事件，显示详细信息，下面2个图片分别展示的是默认情况和点击标题之后。当我们再次点击标题，详细内容又变为隐藏。

![图片1](/content/images/2016/03/35.png)

![图片2](/content/images/2016/03/36-1.png)

通过上述的小例子可以看到`action`助手使用起来也是非常简单的。主要注意下模板上的`action`所指定的事件名称要与`controller`里的方法名字一致。

### 2，action参数

就像调用`javascript`的方法一样，你也可以为`action`助手增加必要的参数。只要在`action`名字后面接上你的参数即可。
```html
<p>
<!-- model直接作为参数传递到controller -->
<button {{action 'hitMe' model}}>点击我吧</button>
</p>
```
对应的在`controller`增加处理的方法`selected`。在此方法内打印获取到的参数值。
```javascript
// app/controllers/myaction.js

import Ember from 'ember';

export default Ember.Controller.extend({

	//  控制页面文章详细内容是否显示
	isShowingBody: false,
	actions: {
		showDetailInfo: function() {
			//  ……同上面的例子
		},
		hitMe: function(model) {   //  参数的名字可以任意
			console.log('The title is ' + model.title);
			console.log('The body is ' + model.body);
		}
	}
});
```

Ember规定我们编写的动作处理的方法都是放在`actions`这个哈希内。哈希的键就是方法名。在`controller`方法上的参数名不要求与模板上传递的名字一致，你可以任意定义。比如方法`hitMe`的参数`model`你也可以使用`m`作为`hitMe`方法的参数。

当用户点击按钮“点击我吧”就会触发方法`hitMe`，然后执行`controller`的同名方法，最后你可以在浏览器的`console`下看到如下的打印信息。

![run result](/content/images/2016/03/37.png)

看到这些打印结果很好的说明了获取的参数是正确的。

### 3，指定action触发的事件类型
默认情况下`action`触发的是`click`事件，你可以指定其他事件，比如键盘按下事件`keypress`。事件的名字与`javascript`提供的名字是相似的，唯一不同的是Ember所识别是事件名字如果是由不同单词组成的需要用中划线分隔，比如`keypress`事件在Ember模板中你需要写成`key-press`。
注意：你指定事件的时候要把事件的名字作为`on`的属性。比如`on='key-press'`。
```html
<a href="#/myaction" {{action 'triggerMe' on="mouse-over"}}>鼠标移到我身上触发</a>
```
```javascript
triggerMe: function() {
	console.log('触发mouseover事件。。。。');
}
```

### 4，指定`action`触发事件的辅助按键

甚至你还可以指定按下键盘某个键后点击才触发`action`所指定的事件，比如按下键盘的`Alt`再点击才会触发事件。使用`allowedkeys`属性指定按下的是那个键。
```html
<br><br>
<button {{action 'pressALTKeyTiggerMe' allowedkeys='alt'}}>按下Alt点击触发我</button>
```

### 5，禁止标签默认行为

在`action`助手内使用属性`preventDefault=false`可以禁止标签的默认行为，比如下面的a标签，如果`action`助手内没有定义这个属性那么你点击链接时只会执行执行的`action`动作，`a`标签默认的行为不会被触发。
```html
<a href="http://www.baidu.com" {{action "showDetailInfo" preventDefault=false}}>
点我跳转
</a>
```

### 6，可以把触发的事件作为参数传递到`controller`

`handlebars`的`action`助手真的是非常强大，你甚至可以把触发的事件作为`action`的参数直接传递到`controller`。不过你需要把`action`助手放在`javascript`的事件里。比如下面的代码当失去焦点时触发，并且通过`action`指定的`dandDidChange`把触发的事件`blur`传递到`controller`。
```html
<label>失去焦点时候触发</label>
<input type="text" value={{textValue}} onblur={{action 'bandDidChange'}} />
```
```javascript
// app/controllers/myaction.js

import Ember from 'ember';

export default Ember.Controller.extend({

	actions: {
		bandDidChange: function(event) {
			console.log('event = ' + event);
		}
	}
	
});
```

![result](/content/images/2016/03/38.png)

从控制台输出结果看出来`event`的值是一个对象并且是一个`focus`事件。
但是如果你在`action`助手内增加一个属性`value='target.value'`(别写错只能是`target.value`)之后，传递到`controller`的则是输入框本身的内容。不再是事件对象本身。
```html
<input type="text" value={{textValue}} onblur={{action 'bandDidChange' value="target.value"}} />
```

![result](/content/images/2016/03/39.png)

这个比较有意思，实现的功能与前面的参数传递类似的。

### 7，`action`助手用在非点击元素上
	`action`助手可以用在任何的`DOM`元素上，不仅仅是用在能点击的元素上（比如`a`、`button`），但是用在其他非点击的元素上默认情况下是不可用的，也就是说点击也是无效的。比如用在`div`标签上，但是你点击`div`元素是没有反应的。如果你需要让`div`元素也能触发单击事件你需要给元素添加一个CSS类'cursor:pointer;`。

总的来说Ember的`action`助手与普通的javascript的事件是差不多的。用法基本上与javascript的事件相似。

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
