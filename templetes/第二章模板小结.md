真快，第二章模板（`template`）已经介绍完毕了！这个章节相对来说是比较简单，只有是有点HTML基础的学习起来并不会很难，几乎也不需要去记忆，自己动手实践实践就能理解。其中比较重要的是`{{link-to}}`和`{{action}}`这两篇。特别是`{{link-to}}`，这个标签几乎都是与路由结合使用的，要注意与路由配置一一对应。

在下一章将为读者介绍第三章路由，如果你是看官网文档的你会发现路由是在模板之前介绍的，我稍微做了下调整，因为根据我自己学习的ember的经验我觉得先介绍模板更好学习。很多东西结合显示效果讲会容易很多。

在介绍路由这一章之前，重新创建了一个项目用于演示，依然是使用[Ember CLI](http://ember-cli.com/user-guide)创建项目。下面是创建命名并且运行项目，测试项目是否创建成功。

```
ember new chapter3_routes
cd chapter3_routes
ember server
```
在浏览器运行：[http://localhost:4200/](http://localhost:4200/)，在界面上能看到**Welcome to Ember**说明项目搭建成功了！！

如果你还不知道怎么使用[Ember CLI](http://ember-cli.com/user-guide)创建项目，请自行根据提供的地址安装配置[Ember CLI](http://ember-cli.com/user-guide)命令环境，在[第一章的小节](http://blog.ddlisting.com/2016/03/18/ember-js-ru-men-zhi-nan-zhi-qi-di-zhang-dui-xiang-mo-xing-xiao-jie/)已经详细介绍过，这里不再赘述。

下面开始路由的学习之旅吧~~~

<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！


