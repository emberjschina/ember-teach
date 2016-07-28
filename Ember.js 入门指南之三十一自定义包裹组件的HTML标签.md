按照惯例，先做好准备工作，使用[Ember CLI](http://ember-cli.com/user-guide)命令生成演示所需的文件：
```
ember g route customizing-component-element
ember g component customizing-component-element
ember g route home
ember g route about
```
默认情况下，组件会被包裹在`div`标签内。比如，组件渲染之后得到下面的代码：
```html
<div id="ember180" class="ember-view">
  <h1>My Component</h1>
</div>
```
`h1`标签就是组件的内容。以`ember`开头的`id`和`class`都是[Ember](http://emberjs.com)自动生成的。如果你需要修改渲染之后生成的HTML不是被包裹在`div`标签，或者修改`id`和`class`等属性值为自定义的值，你可以在组件类中设置。

## 1，自定义包裹组件的HTML标签

默认情况下，组件会被包裹在`div`标签内，如果你需要修改这个默认值你可以在组件类中指定这个包裹的HTML标签。
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav'
});
```
下面自定义一个组件。
```html
<!--  app/templates/components/customizing-component-element.hbs  -->

<ul>
	<li>{{#link-to 'home'}}Home{{/link-to}}</li>
	<li>{{#link-to 'about'}}About{{/link-to}}</li>
</ul>
```
下面是调用组件的模板代码。注意组件被包裹在那个HTML标签内，正确情况下应该是被包裹在`nav`标签中。
```html
<!--  app/templates/customizing-component-element.hbs  -->

{{customizing-component-element}}
```
页面加载之后查看页面的源代码。如下：

![结果HTML代码](/content/images/2016/04/126.png)

可以看到组件`customizing-component-element`的内容确实是被包裹在`nav`标签之中，如果在组件类中没有使用属性`tagName`指定包裹的HTML标签，默认是`div`，你可以把组件类中`tagName`属性删除之后再查看页面的HTML源码代码。

## 2，自定义包裹组件的HTML元素的类名

默认情况下，[Ember](http://emberjs.com)会自动为包裹组件的HTML元素增加一个以`ember`开头的类名，如果你需要增加自定义的CSS类，可以在组件类中使用`className`数组属性指定，可以一次性指定多个CSS类。比如下面的代码例子：
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name']  //指定包裹元素的CSS类
});
```
页面重新加载之后查看源代码，可以看到`nav`标签中多了两个CSS类，一个是`primary`，一个是`my-class-name`。
```html
<nav id="ember411" class="ember-view primary my-class-name">
……
</nav>
```
如果你想根据某个数据的值决定是否增加CSS类也是可以做到的，比如下面的代码，当`urgent`为`true`的时增加一个CSS类`urgent`，否则不增加这个类。要达到这个目的可以通过属性`classNameBindings`设置。
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name'],  //指定包裹元素的CSS类
	classNameBindings: ['urgent'],
	urgent: true
});
```
页面重新加载之后查看源代码，可以看到`nav`标签中多了一个CSS类`urgent`，如果属性`urgent`的值为`false`，CSS类`urgent`将不会渲染到`nav`标签上。
```html
<nav id="ember411" class="ember-view primary my-class-name urgent">
……
</nav>
```
**注意**：`classNameBindings`指定的属性值必须要跟用于判断数据的属性名一致，比如这个例子中`classNameBindings`指定的属性值是`urgent`，用户判断是否增加类的属性也是`urgent`。如果这个属性只是驼峰式命名的那么渲染之后CSS类名将是以中划线`-`分隔，比如`classNameBindings`指定一个名为`secondClassName`，渲染后的CSS类为`second-class-name`。比如下面的演示代码：
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name'],  //指定包裹元素的CSS类
	classNameBindings: ['urgent', 'secondClassName'],
	urgent: true,
	secondClassName: true
});
```
页面重新加载之后查看源代码，可以看到`nav`标签中多了一个CSS类`second-class-name`。
```html
<nav id="ember411" class="ember-view primary my-class-name urgent second-class-name">
……
</nav>
```
如果你不想渲染之后的CSS类名被修改为中划线分隔形式，你可以值`classNameBindings`属性中指定渲染之后的CSS类名。比如下面的代码：
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name'],  //指定包裹元素的CSS类
	classNameBindings: ['urgent', 'secondClassName:scn'],  //指定secondClassName渲染之后的CSS类名为scn
	urgent: true,
	secondClassName: true
});
```
页面重新加载之后查看源代码，可以看到`nav`标签中原来CSS类为`second-class-name`的变成了`scn`。
```html
<nav id="ember411" class="ember-view primary my-class-name urgent scn">
……
</nav>
```
有没有感觉[Ember](http://emberjs.com)既灵活又强大！！[Ember](http://emberjs.com)的设计理念是“约定优于配置”！所以很多的属性默认的设置都是我们平常开发中最常用的格式。
	
除了上述可以指定CSS类名之外，还可以在`classNameBindings`增加简单的逻辑，特别是在处理一些动态效果的时候上述特性是非常有用的。
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name'],  //指定包裹元素的CSS类
	classNameBindings: ['urgent', 'secondClassName:scn', 'isEnabled:enabled:disabled'],
	urgent: true,
	secondClassName: true,
	isEnabled: true  //如果这个属性为true，类enabled将被渲染到nav标签上，如果属性值为false类disabled将被渲染到nav标签上，类似于三目运算
});
```
正如代码的注释所说的，`isEnabled:enabled:disabled`可以理解为一个三目运算，会根据`isEnabled`的值渲染不同的CSS类到`nav`上。

下面的HTML代码是`isEnabled`为`true`的情况，对于`isEnabled`为`false`的情况请读者自己试试：
```html
<nav id="ember411" class="ember-view primary my-class-name urgent scn enabled">
……
</nav>
```
**注意**：如果用于判断的属性值不是一个`Boolean`值而是一个字符串那么得到的结果与上面的结果是不一样的，[Ember](http://emberjs.com)会直接把这个字符串的值作为CSS类名渲染到包裹的标签上。比如下面的代码：
```js
//  app/components/customizing-component-element.js

import Ember from 'ember';

export default Ember.Component.extend({
	// 使用tabName属性指定渲染之后HTML标签
	// 注意属性的值必须是标准的HTML标签名
	tagName: 'nav',
	classNames: ['primary', 'my-class-name'],  //指定包裹元素的CSS类
	classNameBindings: ['urgent', 'secondClassName:scn', 'isEnabled:enabled:disabled', 'stringValue'],
	urgent: true,
	secondClassName: true,
	isEnabled: true,  //如果这个属性为true，类enabled将被渲染到nav标签上，如果属性值为false类disabled将被渲染到nav标签上，类似于三目运算
	stringValue: 'renderedClassName'
});
```
此时页面的HTML源码就有点不一样了。`renderedClassName`作为CSS类名被渲染到了`nav`标签上。
```html
<nav id="ember411" class="ember-view primary my-class-name urgent scn enabled renderedClassName">
……
</nav>
```
对于这点需要特别注意。[Ember](http://emberjs.com)对于`Boolean`值和其他值的判断结果是不一样的。

## 3，自定义包裹组件的HTML元素的属性
	
在前面两点介绍了包裹组件的HTML元素的标签名、CSS类名，在HTML标签上出来CSS类另外一个最常用的就是属性，那么[Ember](http://emberjs.com)同样提供了自定义包裹HTML元素的属性的方法。使用`attributeBindings`属性指定，这个属性的属性方式与`classNameBindings`基本一致。
为了与前面的例子区别开新建一个组件`link-items`，使用命令`ember g component link-items`创建。
```html
<!--  app/templates/components/link-items.hbs  -->

这是个组件
```
在模板中调用组件。
```html
<!--  app/templates/customizing-component-element.hbs  -->

{{customizing-component-element}}
<br><br>

{{link-items}}
```
下面设置组件类，指定包裹的HTML标签为`a`标签，并增加一个属性`href`。
```js
//  app/components/link-items.js

import Ember from 'ember';

export default Ember.Component.extend({
	tagName: 'a',
	attributeBindings: ['href'],
	href: 'http://www.google.com.hk'
});
```
页面重新加载之后得到如下结果：

![渲染后的HTML代码](/content/images/2016/04/127-1.png)

比较简单，对于渲染之后的结果我就不过多解释了，请参考`classNameBindings`属性理解。

到此，有关于组件渲染之后包裹组件的HTML标签的相关设置介绍完毕。内容不多，`classNameBindings`和`attributeBindings`这两个属性的使用方式基本相同。如有疑问欢迎给我留言或者直接查看官方教程。
<br>
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！








