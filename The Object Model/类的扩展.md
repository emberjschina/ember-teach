### 扩展一般属性

> `reopen`不知道怎么翻译好，如果按照`reopen`翻译过来应该是“重新打开”，但是总觉得不顺，所以就译成`扩展`了，如果有不妥请指正。

当你想扩展一个类你可以直接使用`reopen()`方法为一个已经定义好的类添加属性、方法。如果是使用`extend()`方法你需要重新定义一个子类，然后在子类中添加新的属性、方法。
前一篇所过，调用`create()`方法时候不能传入计算属性并且不推荐在此方法中新定义、重写方法，但是使用`reopen()`方法可以弥补`create()`方法的补足。与`extend()`方法非常相似，下面的代码演示了它们的不同之处。

```javascript
Parent = Ember.Object.extend({
    name: 'ubuntuvim',
    fun1() {
        console.log("The name is " + this.name);
    },
    common() {
       console.log("common method..."); 
    }
});   

//  使用extend()方法添加新的属性、方法
Child1 = Parent.extend({
    //  给类Parent为新增一个属性
    pwd: '12345',
    //  给类Parent为新增一个方法
    fun2() {
        console.log("The pwd is " + this.pwd);
    },
    //    重写父类的common()方法
    common() {
        //console.log("override common method of parent...");
        this._super();
    }
});
    
var c1 = Child1.create();
console.log("name = " + c1.get('name') + ", pwd = " + c1.get('pwd'));   
c1.fun1();
c1.fun2();     
c1.common();
console.log("-----------------------");    
    
//  使用reopen()方法添加新的属性、方法
Parent.reopen({
    //  给类Parent为新增一个属性
    pwd: '12345',
    //  给类Parent为新增一个方法
    fun2() {
        console.log("The pwd is " + this.pwd);
    },
    //  重写类本身common()方法
    common() {
        console.log("override common method by reopen method...");
        //this._super();
    },
    //  新增一个计算属性
    fullName: Ember.computed(function() {
	console.log("compute method...");
    })
});
var p = Parent.create();    
console.log("name = " + p.get('name') + ", pwd = " + p.get('pwd'));   
p.fun1();
p.fun2();    
p.common();
console.log("---------------------------");    
p.get('fullName');  //  获取计算属性值，这里是直接输出：compute method...

//  使用extend()方法添加新的属性、方法
Child2 = Parent.extend({
    //  给类Parent为新增一个属性
    pwd: '12345',
    //  给类Parent为新增一个方法
    fun2() {
        console.log("The pwd is " + this.pwd);
    },
    //    重写父类的common()方法
    common() {
        //console.log("override common method of parent...");
        this._super();
    }
});    
var c2 = Child2.create();
console.log("name = " + c2.get('name') + ", pwd = " + c2.get('pwd'));   
c2.fun1();
c2.fun2(); 
c2.common();
```

![执行结果](/content/images/2016/03/19.png)

从执行结果可以看到如下的差异:<br>
**同点**： 都可以用于扩展某个类。<br>
**异点**：<br>
1. `extend`需要重新定义一个类并且要继承被扩展的类；
2. `reopen`是在被扩展的类本身上新增属性、方法，可以扩展计算属性（相比`create()`方法）； 

到底用那个方法有实际情况而定，`reopen`方法会改变了原类的行为（可以想象为修改了对象的原型对象的方法和属性），就如演示实例一样在`reopen`方法之后调用的`Child2`类的`common`方法的行为已经改改变了，在编码过程忘记之前已经调用过`reopen`方法就有可能出现自己都不知道怎么回事的问题！
如果是`extend`方法会导致类越来越多，继承树也会越来越深，对性能、调试也是一大挑战，但是`extend`不会改变被继承类的行为。

### 扩展静态属性

使用`reopenClass()`方法可以扩展`static`类型的属性、方法。

```javascript
Parent = Ember.Object.extend();   
    
//  使用reopenClass()方法添加新的static属性、方法
Parent.reopenClass({
    isPerson: true,
    username: 'blog.ddlisting.com' 
	//,name: 'test'  //这里有点奇怪，不知道为何不能使用名称为name定义属性，会提示这个是自读属性，使用username却没问题！！估计name是这个方法的保留关键字
});

Parent.reopen({
    isPerson: false,
    name: 'ubuntuvim'
});
console.log(Parent.isPerson);
console.log(Parent.name);  //  输出空
console.log(Parent.create().get('isPerson'));
console.log(Parent.create().get('name'));    //  输出 ubuntuvim
```

对于在`reopenClass`方法中使用属性`name`的问题下面的地址有解释

1. [http://discuss.emberjs.com/t/reopenclass-method-can-not-pass-attribute-named-name-of-it/10189](http://discuss.emberjs.com/t/reopenclass-method-can-not-pass-attribute-named-name-of-it/10189)
2. [http://stackoverflow.com/questions/36078464/reopenclass-method-can-not-pass-attribute-named-name-of-it](http://stackoverflow.com/questions/36078464/reopenclass-method-can-not-pass-attribute-named-name-of-it)

博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用在github项目上给我个`star`吧。您的肯定对我来说是最大的动力！！