### 简单的计算属性

简单地来说，计算属性就是将函数声明为属性，就类似于调用了一个函数，`Ember`会自动调用这个函数。计算属性最大的特点就是能自动检测变化，及时更新数据。

```javascript
Person = Ember.Object.extend({
    firstName: null,
    lastName: null,
    
    //  fullName 就是一个计算属性
    fullName: Ember.computed('firstName', 'lastName', function() {
        return this.get('firstName') + ", " + this.get('lastName');
    })
});

//  实例化同时传入参数
var piter = Person.create({
    firstName: 'chen',
    lastName: 'ubuntuvim'
});
console.log(piter.get('fullName'));  // output >>   chen, ubuntuvim
```

计算属性其实就是一个函数，如果你接触过就`jQuery、Extjs`相信你回非常熟悉，在这两个框架中函数就是这么定义的。只不过在`Ember`中，把这种函数当做属性来处理，并且可以通过get获取函数的返回值。

### 计算属性链

在`Ember`程序中，计算属性还能调用另外一个计算属性，形成计算属性链，也可以用于扩展某个方法。在上一实例的基础上增加一个`description()`方法。

```javascript
Person = Ember.Object.extend({
    firstName: null,
    lastName: null,
    age: null,
    county: null,
    
    //  fullName 就是一个计算属性
    fullName: Ember.computed('firstName', 'lastName', function() {
        return this.get('firstName') + ", " + this.get('lastName');
    }),
    description: Ember.computed('fullName', 'age', 'county', function() {
        return this.get('fullName') + " age " + this.get('age') + " county " + this.get('county');
    })
});

//  实例化同时传入参数
var piter = Person.create({
    firstName: 'chen',
    lastName: 'ubuntuvim',
    age: 25,
    county: 'china'
});
console.log(piter.get('description'));  // output >>   chen, ubuntuvim
```

当用户使用`set`方法改变`firstName`的值，然后再调用`get('description')`得到的值也是更新后的值。

### 重写计算属性的get、set方法

注意要把重写的属性作为参数传入`computed`方法，要区别计算属性的定义方法，定义的时候`computed`方法的最后一个参数是一个`function`，而重写的时候最后一个参数是一个`hash`。

```javascript
//    重写计算属性的get、set方法
Person = Ember.Object.extend({
    firstName: null,
    lastName: null,
    
    //  重写计算属性fullName的get、set方法
    fullName: Ember.computed('firstName', 'lastName', {
        get(key) {
            return this.get('firstName') + "," + this.get('lastName');
        },
        set(key, value) {
            //  这个官方文档使用的代码，但是我运行的时候出现 Uncaught SyntaxError: Unexpected token [  这个错误，不知道是否是缺少某个文件，后续会补上；
//            console.log("value = " + value);
//            var [ firstName, lastName ] = value.split(/\s+/);  
            var firstName = value.split(/\s+/)[0];
            var lastName = value.split(/\s+/)[1];
            this.set('firstName', firstName);
            this.set('lastName', lastName);
            
        }
    }),
//    对于普通的属性无法重写get、set方法
//    firstName: Ember.computed('firstName', {
//        get(key) {
//            return this.get('firstName') + "@@";
//        },
//        set(key, value) {
//            this.set('firstName', value);
//        }
//    })
});
    
var jack = Person.create();    
jack.set('fullName', "james kobe");
console.log(jack.get('firstName'));
console.log(jack.get('lastName'));
```

![运行结果](/content/images/2016/03/7.png)

### 计算属性值的统计

我们经常会遇到这种情况：某个计算属性值是依赖某个数组或者其他对象的，比如在`Ember`的`todos`这个例子中有这样的一段代码。

```javascript
export default Ember.Controller.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],
  remaining: Ember.computed('todos.@each.isDone', function() {
    var todos = this.get('todos');
    return todos.filterBy('isDone', false).get('length');
  })
});
```

计算属性`remaining`的值于依赖数组`todos`。在这里还有个知识点：在上述代码`computed()`方法里有一个`todos.@each.isDone`这样的键，里面包含了一个特别的键`@each`（后面还会看到更特别的键`[]`）。需要注意的是这种键不能嵌套并且是只能获取一个层次的属性。比如`todos.@each.foo.name`(获取多层次属性，这里是先得到foo再获取`name`)或者`todos.@each.owner.@each.name`(嵌套)这两种方式都是不允许的。

在如下4种情况`Ember`会自动更新绑定的计算属性值：<br>
1.在`todos`数组中任意一个对象的`isDone`属性值发生变化的时候；
2.往`todos`数组新增元素的时候；
3.从`todos`数组删除元素的时候；
4.在控制器中`todos`数组被改变为其他的数组的时候；

比如下面代码演示的结果；
```javascript
Task = Ember.Object.extend({
  isDone: false  //  默认为false
}); 

WorkerLists = Ember.Object.extend({
  //  定义一个Task对象数组
  lists: [
    Task.create({ isDone: false }),
    Task.create({ isDone: true }),
    Task.create(),
    Task.create({ isDone: true }),
    Task.create({ isDone: true }),
    Task.create({ isDone: true }),
    Task.create({ isDone: false }),
    Task.create({ isDone: true })
  ],

  remaining: Ember.computed('lists.@each.isDone', function() {
    var lists = this.get('lists');
    //  先查询属性isDone值为false的对象，再返回其数量
    return lists.filterBy('isDone', false).get('length');
  })
});

// 如下代码使用到的API请查看：http://emberjs.com/api/classes/Ember.MutableArray.html
var wl = WorkerLists.create();
//  所有isDone属性值未做任何修改
console.log('1,>> Not complete lenght is ' + wl.get('remaining'));  //  output 3
var lists = wl.get('lists');  //  得到对象内的数组

// -----  演示第一种情况： 1. 在todos数组中任意一个对象的isDone属性值发生变化的时候；
//  修改数组一个元素的isDone的 值
var item1 = lists.objectAt(3);  //  得到第4个元素 objectAt()方法是Ember为我们提供的
// console.log('item1 = ' + item1);
item1.set('isDone', false);
console.log('2,>> Not complete lenght is ' + wl.get('remaining'));  //  output 4

//  --------- 2.  往todos数组新增元素的时候；
lists.pushObject(Task.create({ isDone: false }));  //新增一个isDone为false的对象
console.log('3,>> Not complete lenght is ' + wl.get('remaining'));  //  output 5

//  --------- 3.  从todos数组删除元素的时候；
lists.removeObject(item1);  // 删除了一个元素
console.log('4,>> Not complete lenght is ' + wl.get('remaining'));  //  output 4

//  --------- 4.  在控制器中todos数组被改变为其他的数组的时候；
//  创建一个Controller
TodosController = Ember.Controller.extend({
  // 在控制器内定义另外一个Task对象数组
  todosInController: [
    Task.create({ isDone: false }),
    Task.create({ isDone: true })
  ],
  //  使用键”@each.isDone“遍历得到的filterBy()方法过滤后的对象的isDone属性
  remaining: function() {
    //  remaining()方法返回的是控制器内的数组
    return this.get('todosInController').filterBy('isDone', false).get('length');
  }.property('@each.isDone')  //  指定遍历的属性
});
todosController = TodosController.create();
var count = todosController.get('remaining');
console.log('5,>> Not complete lenght is ' + count);  //  output 1
```

![代码演示的结果](/content/images/2016/03/8.png)

上述的情况中，我们对数组对象的是关注点是在对象的属性上，但是实际中往往很多情况我们并不关系对象内的属性是否变化了，而是把数组元素作为一个整体对象处理（比如数组元素个数的变化）。相比上述的代码下面的代码检测的是数组对象元素的变化，而不是对象的`isDone`属性的变化。在这种情况你可以看看下面例子，在例子中使用键`[]`代替键`@each`。从键的变化也可以看出他们的不同之处。

```javascript
Task = Ember.Object.extend({
  isDone: false,  //  默认为false
  name: 'taskName',
  //  为了显示结果方便，重写toString()方法
  toString: function() {
    return '[name = '+this.get('name')+', isDone = '+this.get('isDone')+']';
  }
}); 

WorkerLists = Ember.Object.extend({
  //  定义一个Task对象数组
  lists: [
    Task.create({ isDone: false, name: 'ibeginner.sinaapp.com' }),
    Task.create({ isDone: true, name: 'i2cao.xyz' }),
    Task.create(),
    Task.create({ isDone: true, name: 'ubuntuvim' }),
    Task.create({ isDone: true , name: '1527254027@qq.com'}),
    Task.create({ isDone: true })
  ],

  index: null,
  indexOfSelectedTodo: Ember.computed('index', 'lists.[]', function() {
    return this.get('lists').objectAt(this.get('index'));
  })
});


var wl = WorkerLists.create();
//  所有isDone属性值未做任何修改
var index = 1;
wl.set('index', index);
console.log('Get '+wl.get('indexOfSelectedTodo').toString()+' by index ' + index);
```

![代码演示的结果](/content/images/2016/03/9.png)


`Ember.computed`这个组件中有很多使用键`[]`实现的方法。当你想创建一个计算属性是数组的时候特别适用。你可以使用`Ember.computed.map`来构建你的计算属性。

```javascript
const Hamster = Ember.Object.extend({
  chores: null,
  excitingChores: Ember.computed('chores.[]', function() { //告诉Ember chores是一个数组
    return this.get('chores').map(function(chore, index) {
      //return `${index} --> ${chore.toUpperCase()}`;  //  可以使用${}表达式，并且在表达式内可以直接调用js方法
      return `${chore}`;  //返回元素值
    });
  })
});

//  为数组赋值
const hamster = Hamster.create({
  //  名字chores要与类Hamster定义指定数组的名字一致
  chores: ['First Value', 'write more unit tests']
});

console.log(hamster.get('excitingChores'));
hamster.get('chores').pushObject("Add item test");  //add an item to chores array
console.log(hamster.get('excitingChores'));
```

`Ember`还提供了另外一种方式去定义数组类型的计算属性。

```javascript
const Hamster = Ember.Object.extend({
  chores: null,
  excitingChores: Ember.computed('chores.[]', function() {
    return this.get('chores').map(function(chore, index) {
      //return `${index} --> ${chore.toUpperCase()}`;  //  可以使用${}表达式，并且在表达式内可以直接调用js方法
      return `${chore}`;  //返回元素值
    });
  })
});

//  为数组赋值
const hamster = Hamster.create({
  //  名字chores要与类Hamster定义指定数组的名字一致
  chores: ['First Value', 'write more unit tests']
});

console.log(hamster.get('excitingChores'));
hamster.get('chores').pushObject("Add item test");  //add an item to chores array
console.log(hamster.get('excitingChores'));
```

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用在github项目上给我个`star`吧。您的肯定对我来说是最大的动力！！

