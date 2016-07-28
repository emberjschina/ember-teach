本篇主要介绍格式转换、自定义`helper`、自定义`helper`参数、状态`helper`、HTML标签转义这几个方面的东西。

按照文章惯例先准备好测试所需要的数据、文件。仍然是使用[Ember CLI](http://ember-cli.com/user-guide)命令，这次我们创建的是`helper`、`controller`、`route`（创建`route`会自动创建`template`）。
```
ember generate helper my-helper
ember generate controller tools-helper
ember generate route tools-helper
```

### 1，自定义`helper`

自定义助手非常简答直接使用[Ember CLI](http://ember-cli.com/user-guide)命令生成就可以了。当然你也可以手动创建，自定义的助手都是放在`app/helpers`目录下。Ember会根据模板上使用的助手自动到这个目录查找。定义了`helper`之后你就可以直接在模板上使用。
```html
<!--  //app/templates/tools-helper.hbs  -->

my-helper: {{my-helper}}
```
程序没有报错，但是什么也没有显示。是的什么也没有显示。没有显示就对了。因为我们对于刚刚创建的`app/helpers/my-helper.js`没有做任何的修改。你可以看这个文件的代码。直接返回了`params`，目前来说这个参数是空的。修改这个文件，直接返回一个字符串。
```javascript
//  app/helpers/my-helper.js

import Ember from 'ember';

export function myHelper(params/*, hash*/) {
  return "hello world!";
}

export default Ember.Helper.helper(myHelper);
```
此时可以在页面上看到直接打印了“hello world!”这个字符串。这就是一个最简单的自定义`helper`，不过这么简单`helper`显然是没啥用的。Ember的作者肯定不会这么傻的，接着下面介绍`helper`的参数。
注意：使用模板的名字跟文件名是一致的。不同单词使用`-`分隔，虽然这个命名规则不是强制性的但是Ember建议你这么做，Ember会自动根据`helper`的名字找到对应的自定义的`helper`，然后执行`helper`里名字为`myHelper`（名字是文件名的驼峰式命名）的方法，在这个方法里你可以实现你需要的逻辑。这些工作Ember自动帮你做了，不需要你编写解析的代码。

#### 1，`helper`无名参数

上面的代码定义了一个最简单的helper，不过没啥用，Ember允许在自定义的helper上添加自定义的参数。
```html
my-helper-param: {{my-helper 'chen' 'ubuntuvim'}}
```
在这个自定义的`helper`中增加了两个参数，既然有了参数那么又有什么用呢？当然是有用的，你可以在自定义的`helper`中获取参数，获取模板的参数有两种方式。
<br>**写法一**<br>
```javascript
//  app/helpers/my-helper.js

import Ember from 'ember';

export function myHelper(params/*, hash*/) {
	// 获取模板上的参数
  	var p1 = params[0];
  	var p2 = params[1];
  	console.log('p1 = ' + p1 + ", p2 = " + p2);

	return p1 + " " + p2;
}

export default Ember.Helper.helper(myHelper);
```
<br>**写法二**<br>
```javascript
//  app/helpers/my-helper.js

import Ember from 'ember';

export function myHelper([arg1, arg2]) {
	console.log('p1 = ' + arg1 + ", p2 = " + arg2);
	return arg1 + " " + arg2;
}

export default Ember.Helper.helper(myHelper);
```
参数很多的情况使用第一种方式用循环获取参数比较方便，参数少情况第二种方式更加简便，直接使用！

**注意：参数的顺序与模板传入的顺序一致。**

页面刷新之后可以在页面或者浏览器控制台看到在`helper`上设置的参数值了吧！！如果你的程序没有错误在浏览器上你也会得到下图的结果：

![result](/content/images/2016/03/52.png)

第一行因为在模板上没有传入参数所以是`undefined`，第二行传入了参数，并且直接从`helper`返回显示。

#### 2，`helper`命名参数

上一点演示了在模板中传递无名的参数，这一小节讲为你介绍有名字的参数。
```html
my-helper-named-param: {{my-helper firstName='chen' lastName='ubuntuvim'}}
```
相比于第一种使用方式给每个参数增加了参数名。那么`helper`处理类有要怎么去获取这些参数呢？
```javascript
//  app/helpers/my-helper.js

import Ember from 'ember';

// 对于命名参数使用namedArgs获取
export function myHelper(params, namedArgs) { 
	console.log('namedArgs = ' + namedArgs);
	console.log('params = ' + params);
	console.log('=========================');
	return namedArgs.firstName + ", " + namedArgs.lastName; 
}

export default Ember.Helper.helper(myHelper);
```
获取命名参数使用`namedArgs`，其实你也可以按照前面的方法使用`params`获取参数值。你在第一行打印语句上打上断点，是浏览器进入debug模式，但不执行，你会发现`params`一开始是有值`namedArgs`没有值，但是执行到最后正好相反，`params`的值被置空了，`namedArgs`却有了模板设置的值，你可以猜想下，Ember可能是把`params`的值赋值到`namedArgs`上了，不同之处是`namedArgs`是以对象属性的方式取值并且不用关心参数的顺序问题，`params`是以数组的方式取值需要关心参数的顺序。

![result](/content/images/2016/03/49.png)


### 2，时间格式化

做开发的都应该遇到过数字或者时间格式问题，特别是时间格式问题应该是最普遍遇到的。不同的项目时间格式往往不同，有`yyyy-DD-mm`类型的有`yyyyMMdd`类型以及其他类型。

同样的Ember模板也给我们提供了类似的解决办法，那就是自定义格式化方法。通过自定义`helper`实现数据的格式化。

1. 创建格式化`helper`：`ember generate helper format-date`
2. 在`controller`初始化一个时间数据。

```javascript
//  app/controllers/tools-helper.js

import Ember from 'ember';

export default Ember.Controller.extend({
	currentDate: new Date()
});
```
默认情况下显示数据`currentDate`。
```html
<!--  //app/templates/tools-helper.hbs  -->
{{ currentDate}}`
```
此时显示的默认的数据格式。
运行[http://localhost:4200/tools-helper](http://localhost:4200/tools-helper)，可以在页面看到：`Mon Sep 21 2015 23:46:03 GMT+0800 (CST) `这种格式的时间。显然不太合法我们的习惯，看着都觉得不舒服。那下面使用自定义的`helper`格式化日期格式。
```javascript
//  app/helpers/format-data.js

import Ember from 'ember';

/**
 * 注意：方法名字是文件名去掉中划线变大写，驼峰式写法
 * 或者你也可以直接作为helper的内部函数
 * @param  {[type]} params 从助手{{format-data}}传递过来的参数
 */
export function formatDate(params/*, hash*/) {
	console.log('params = ' + params);
    var d = Date.parse(params);
    var dd = new Date(parseInt(d)).toLocaleString().replace(/:\d{1,2}$/,' ');  //  2015/9/21 下午11:21
    var v = dd.replace("/", "-").replace("/", "-").substr(0, 9);
    return v;
}

export default Ember.Helper.helper(formatDate);
```
或者你也可以这样写。
```javascript
export default Ember.Helper.helper(function formatDate(params/*, hash*/) {
    var d = Date.parse(params);
    var dd = new Date(parseInt(d)).toLocaleString().replace(/:\d{1,2}$/,' ');  //  2015/9/21 下午11:21
    var v = dd.replace("/", "-").replace("/", "-").substr(0, 9);
    return v;
});
```
为了简便，直接就替换字符，修改时间分隔字 `/`为`-`。	然后修改显示的模板，使用自定义的`helper`。
```html
<!--  //app/templates/tools-helper.hbs  -->
{{format-date currentDate}}
```
此时页面上显示的时间是我们熟悉的时间格式：

![result](/content/images/2016/03/50.png)


上面介绍的是简答的用法，Ember还允许你传入时间的格式（`format`），以及本地化类型（`locale`）。

1. 用命令新建一个`helper`：`ember generate helper format-date-time`
2. 在`controller`类里新增两个用于测试的属性`cDate`和
`currentTime`。

```javascript
//  app/controllers/tools-helper.js

import Ember from 'ember';
export default Ember.Controller.extend({
	currentDate: new Date(),
	cDate: '2015-09-22',
	currentTime: '00:22:32'
});
```

```html
<!--  //app/templates/tools-helper.hbs  -->

<br><br><br>
format-date-time: {{format-date-time currentDate cDate currentTime format="yyyy-MM-dd h:mm:ss"}}


<br><br><br>
format-date-time-local: {{format-date-time currentDate cDate currentTime format="yyyy-MM-dd h:mm:ss" locale="en"}}
```
在助手`format-date-time`上一共有4个属性。`cDate`和`currentTime`是从上下文获取值的变量，`format`和`locale`是Ember专门提供用于时间格式化的属性。

下面看看`format-date-time`这个助手怎么获取页面的数据。
```javascript
// app/helpers/format-date-time.js

import Ember from 'ember';

export function formatDateTime(params, hash) {
	//  参数的顺序跟模板{{format-date-time currentDate cDate currentTime}}上使用顺序一致，
	//  cDate比currentTime先，所以第一个参数是cDate
	console.log('params[0] = ' + params[0]);  //第一个参数是cDate,
	console.log('params[1] = ' + params[1]);  //  第二个是currentTime
	console.log('hash.format = ' + hash.format);
	console.log('hash.locale = ' + hash.locale);
	console.log('------------------------------------');

  	return params;
}

export default Ember.Helper.helper(formatDateTime);
```
我只是演示怎么获取页面`format-date-time`助手设置的值，得到页面设置的值你想干嘛就干嘛……
最后看看浏览器控制台的输出信息。

![result](/content/images/2016/03/51.png)

因为页面使用了两次这个助手，所以自然也就打印了两次。
#### 3，转义HTML标签

官方的解释是：为了保护你的应用免受跨点脚本攻击（XSS），Ember会自动把`helper`返回值中的HTML标签转义。

新建一个`helper`：`ember generate helper escape-helper`
```javascript
// app/helpers/escape-helper.js

import Ember from 'ember';

export function escapeHelper(params/*, hash*/) {
  // return Ember.String.htmlSafe(`<b>${params}</b>`);
  return `<b>${params}</b>`;
}

export default Ember.Helper.helper(escapeHelper);
```
```html
escape-helper: {{escape-helper "helloworld!"}}
```
此时页面上会直接显示“<b>helloworld!</b>”而不是“helloworld”被加粗了！如果你确定你返回的字符串是安全的你可用使用`htmlSafe`方法，这个方法不会把HTML代码转义，HTML代码仍然能起作用，那么页面显示的将是加粗的“helloworld！”。

到此模板这一章全部讲完了！！！但愿你能从中得到一点收获！！后面的文章将开始讲`route`，`route`在[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)这一篇已经讲过一点，但不是很详细。接下来的一章将会为你详细解释`route`。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！














