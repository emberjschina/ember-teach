[Ember JS](http://emberjs.com/)提供一套自己的类系统，普通的`JavaScript`标准类不能自动更新属性值，Ember JS的类会自动触发观察者，自动更新属性值、自动刷新模板上的属性值。如果一个类是Ember JS提供的可以看到前缀命名空间是`Ember.Object`。
`Ember`类定义使用`extend()`方法，创建类实例使用`create()`方法，可以在方法传入参数，但是参数要以`hash`列表方式传入。

Ember JS重写了标准`JavaScript`的数组类`Array`，并且为了与标准`JavaScript`类区别命名为`Ember.Enumerable`（[API介绍](http://emberjs.com/api/classes/Ember.Enumerable.html)）

Ember JS还扩展了`String`属性的特性，提供了一系列特有的处理方法，[API介绍](http://emberjs.com/api/classes/Ember.String.html)。

*关于类的命名规则在此不做介绍，自己网上找一份`Java`的命名规则的教材看看即可。*

开始之前先做好准备工作，首先创建一个HTML文件，并引入Ember JS所必须的文件（后面会介绍一种更加简单的方法去搭建`EmberJS`的项目方法，当然如果你有时间也可以提前去了解，这种方式是使用[`Ember CLI`](http://www.ember-cli.com/user-guide/)搭建`EmberJS`的项目）。

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Ember.js • Guides</title> 
  <script src="http://cdn.bootcss.com/jquery/2.0.0/jquery.js"></script>

<!--  <script src="http://cdn.bootcss.com/ember.js/2.1.0-beta.2/ember.js"></script>-->
   <script src="http://cdn.bootcss.com/ember.js/2.1.0-beta.2/ember.debug.js"></script>
    
    <script src="http://cdn.bootcss.com/ember.js/2.1.0-beta.2/ember.prod.js"></script>
<script type="text/javascript">
//  在这里编写Ember代码
</script>
  </head>
   
   
  <body>
  
  </body>
</html>
```

上面代码是一个简单的`HTML`文件，所需的`Ember`库直接使用`CDN`。

### 类定义

下面定义一个`Person`类，定义方式如下：

```javascript
Person = Ember.Object.extend({
  say(thing) {
    alert(name);
  }
});
```

上面代码定义了一个`Person`类，并且在类里面还定义了一个方法`say`，方法传入一个参数`thing`。方法体仅仅是打印了传入的参数。

### 类继承

在子类重写父类的方法，并在方法里调用`_super()`方法来调用父类中对应的方法触发父类方法的行为。

```javascript
Person = Ember.Object.extend({
    
  say(thing) {
    var name = this.get('name');
    alert(name + " says: " + thing);
  }
});

Soldier = Person.extend({
    
  say(thing) {
    // this will call the method in the parent class (Person#say), appending
    // the string ", sir!" to the variable `thing` passed in
    this._super(thing + ", sir!");
  }
});

var yehuda = Soldier.create({
  name: "Yehuda Katz"
});

yehuda.say("Yes"); // alerts "Yehuda Katz says: Yes, sir!"
```

运行代码，刷新浏览器，可以看到如下结果：

![运行结果截图](http://7xnrhh.com1.z0.glb.clouddn.com/1.png)

结果正确了，但是我们还不知道类是怎么初始化的，它初始化的顺序又是怎么样的呢？其实每个类都有一个默认的初始化方法，555……别急，接着往下看。

### 类实例化

要获取一个类的实例只需要调用类的`create()`方法即可。

```javascript
Person = Ember.Object.extend({
    show() {
        console.log("My name is " + this.get('name'));
    }
});
var person = Person.create({
    name: 'ubuntuvim'
});
person.show();  // My name is ubuntuvim
    
var person2 = Person.create({
    pwd: 'ubuntuvim'
});
//  由于创建person2的时候没有设置name的值，默认是undefined
person2.show();  // My name is undefined
```

![结果图](http://7xnrhh.com1.z0.glb.clouddn.com/2.png)

**注意**：处于性能的考虑在使用`create()`方法创建实例的时候，不允许新定义、重写计算属性，也不推荐新定义、重写普通方法，`Ember`推荐在使用`create()`方法时只是传递简单的参数，比如上述代码的`{name: 'ubuntuvim'}`。如果你需要新地定义、重写方法请新建一个子类来实现。

**在`create()`方法内定义计算属性**，运行后会直接报如下图的报错信息。

```javascript
Person = Ember.Object.create({
    show() {
        console.log("My name is " + this.get('name'));
    },
	fullName: Ember.computed(function() {
		console.log("computed properties.");
	})
});
```

![报错信息](/content/images/2016/03/18.png)


### 类初始化
前面提过，我们在类继承的时候到底类是怎么初始化，这节就介绍类的初始化，`Ember`定义了一个`init()`方法，此方法在类被实例化的时候自动调用。

```javascript
Parent = Ember.Object.extend({
    init() {
        console.log("parent init...");
    },
    show() {
        console.log("My name is " + this.get('name'));
    },
    others() {
        console.log("the method in parent class..");
    }
});
//parent = Parent.create({
//    name: 'parent'
//});  
    
Child = Parent.extend({
    init() {
        console.log("child init...");
    },
    show() {
        this._super();
    }
});

child = Child.create({
    name: 'child'
});    
child.show();
child.others();
```

**注意**：`init()`方法只有在类的`create()`方法被调用的时候才会被自动调用，上面的例子中，如果只是`child.others()`这个方法父类并不会调用`init()`方法，只有执行`Parent.create()`这个调用的时候才会执行`init()`方法。
上面代码如果把`Parent.create()`这几句代码注释掉得到的结果如下：

![运行结果](http://7xnrhh.com1.z0.glb.clouddn.com/3.png)

可见父类的`init()`方法没有被调用，然后修改代码，注释掉`child.others()`这句，再把`Parent.create()`这几句的注释去掉。得到如下结果

![去掉child.others()结果](http://7xnrhh.com1.z0.glb.clouddn.com/4.png)

可以看到父类的`init()`方法被调用了！由此可见`init()`方法是在调用`create()`方法的时候才调用的。
在项目中有可能你需要继承`Ember`提供的组件，比如继承`Ember.Component`类，此时你就要注意了，在你继承`Ember`的组件的时候你必须显式的调用父类方法`this._super()`否则你继承得到的类无法获取`Component`提供的行为或者得到无法预知的结果。

### 类属性的访问

`Ember`建议访问类的属性使用`get、set`方法。如果你直接使用`obj.prop`这种方式访问也是可以得到类的属性值，但是如果你不是使用访问器操作的就会导致很多问题：计算属性不能被重新计算、无法察觉对象属性的变化、模板也不能自动更新。

```javascript
Person = Ember.Object.extend({
    name: 'ubuntuvim'
});
// Ember 推荐的访问方式
var person = Person.create();
console.log("My name is " + person.get('name'));
person.set('name', "Tobias Funke");
console.log("My name is " + person.get('name'));   

console.log("---------------------------");
//  不推荐的方式
var person2 = Person.create();    
console.log("My name is " + person2.name);
person2.name = "Tobias Funke";
console.log("My name is " + person2.name);
```

Ember为我们封装了`get、set`实现细节，开发者直接使用即可。
<br>
最后感谢[唯獨莪靑睐](http://weibo.com/3265765111)的指正。
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用在github项目上给我个`star`吧。您的肯定对我来说是最大的动力！！
