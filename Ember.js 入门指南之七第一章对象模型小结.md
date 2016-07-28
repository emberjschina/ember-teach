本篇之前的6篇文章都是第一章的内容，这一章节主要介绍了`Ember`的对象模型。其中最重要的是计算属性和枚举这2章，非常之重要，一定要好好掌握！

下一章节是第二章模板，`Ember`应用使用的模板库是`handlebar`（[点我查看更多有关此模板的介绍](http://handlebarsjs.com/)），这个模板库功能强大，有丰富的标签，包括判断标签`if`，`if else`,以及遍历标签`each`等等。

另外，从下一章开始，我们不再自己手动搭建`Ember`项目，也不用手动引入`Ember`库文件，而是使用官方提供的一个非常棒的构建工具——`Ember CLI`，要使用这个构建工具首先安装并配置。下面两个地址是介绍安装与配置的（推荐第一个）：

1. [http://www.ember-cli.com/user-guide/](http://www.ember-cli.com/user-guide/)
2. [https://guides.emberjs.com/v2.4.0/getting-started/](https://guides.emberjs.com/v2.4.0/getting-started/)

`Ember CLI`是一个非常重要的构建工具，它可以为开发者创建文件并初始化部分固定的代码。它还可以运行、打包、测试`Ember`应用。

下面使用这个工具创建一个新的`Ember`项目`chapter2_tempalte`。

1. 新建项目命令: 
`ember new chapter2_tempalte`
2. 进入项目目录并启动服务器： 
`cd chapter2_template`<br>
`ember server`
3. 运行项目，浏览器打开这个链接：[http://localhost:4200/](http://localhost:4200/)，如果你能看到如下信息说明安装成功了。

![run proj](/content/images/2016/03/14-1.png)

如果项目创建成功你可以继续往下看，如果项目创建不成功请重试，因为后面的代码都基于这个项目来演示的！！！对于创建项目后得到的每个文件和目录请你看官网文档，上面会有非常详细的说明。为了方便懒人在此就简单介绍其中几个很重要的文件和目录：
<table border="1" style="border: 1px solid #ccc !important;">
  <tr bgcolor="#ccc" style="border: 1px solid #ccc !important;">
<td>目录</td>	<td>说明</td>
</tr>
<tr>
<td>app	</td>	<td>项目的主要代码都是放在这个目录下</td>
</tr>
<tr>
<td>app/controllers</td>	<td>存放C（MVC）层（controller）的代码文件</td>
</tr>
<tr>
<td>app/helpers</td>	<td>	存放自定义的helper代码文件</td>
</tr>
<tr>
<td>app/models	</td>	<td>存放M（MVC）层（model）代码文件</td>
</tr>
<tr>
<td>app/routes</td>	<td>	存放项目路由设置代码文件</td>
</tr>
<tr>
<td>app/templates	</td>	<td>存放项目模板代码文件</td>
</tr>
<tr>
<td>bower_components</td>	<td>存放使用bower命令安装的第三方插件库</td>
</tr>
<tr>
<td>bower.json</td>	<td>保存使用bower命令安装的第三方库的配置</td>
</tr>
<tr>
<td>package.json</td>	<td>保存使用npm命令安装的第三方库的配置</td>
</tr>
<tr>
<td>node_modules</td>	<td>存放使用npm命令安装的第三方插件库</td>
</tr>
<tr>
<td>ember-cli-build.js	</td>	<td>设置构建规范，引入第三方库</td>
</tr>
<tr>
<td>dist	</td>	<td>存放编译打包后的项目文件，可以直接复制到服务器中运行</td>
</tr>
</table>
上述这些文件或者目录是后面开发过程经常会用到，相对其他目录和文件来说这些目录和文件是很重要的，只要你是使用`ember new appName`命令生成的项目都会包括上述这些目录或者文件。其中最重要的就是`app`目录下的文件、目录了，从`app`里面的目录结果你就可以很清楚的看到这是个`MVC`框架的项目。`Ember`之所以能找到`controller`对应的`template`也是根据目录和文件的名称找到的，`Ember`是有自己的一套命名规则的，如果你想了解更多有关信息请移步[folder-layout](http://ember-cli.com/user-guide/#folder-layout)。

搭好环境之后开始我们的`Ember`之旅吧！！！
<br>
博文完整代码放在[Github](https://github.com/ubuntuvim/my_emberjs_code)（博文经过多次修改，博文上的代码与github代码可能又出入，不过影响不大！），如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！

