
**独乐乐不如众乐乐。**

你踩的坑很可能后来者也回踩！拯救菜鸟人人有着！！


本篇为`Ember.js`开发者汇总常见问题，比如浏览器兼容性问题、组件使用等。大部分内容都摘了于[`stackoverflow`](http://stackoverflow.com/search?q=ember.js)和[`emberjs discuss`](http://discuss.emberjs.com/)。

**1. 浏览器兼容性支持**

[emberjs browser support](http://stackoverflow.com/questions/9873744/ember-js-browser-support)

**2. 部署服务器后刷新出现404问题**

如果你的项目使用命令`ember build --prod`打包后部署到服务器上，进入首页没问题，并且在首页刷新也没问题。但是进入某个子路由后再刷新页面就会出现`404`错误。但是这个错误在开发环境却没有任何问题。这是由于服务器设置的问题。以nginx为例。只需要修改：`nginx/conf/vhosts/default.conf`即可_配置文件路由要根据自己nginx路径为准_。在此配置文件中加入如下内容：

```
location / {
        rewrite ^ /index.html break;
    }
    
    location /assets/ {
        # do nothing and let nginx handle this as usual
    }
```

参考网址：[http://discuss.emberjs.com/t/how-to-serve-all-routes-on-a-production-server-exactly/6372/2](http://discuss.emberjs.com/t/how-to-serve-all-routes-on-a-production-server-exactly/6372/2)

还有另外一种更加简单的解决办法：直接在`router.js`设置`location`属性为`hash`即可。更多解释请看官网教程[EMBER-ROUTING MODULE](http://emberjs.com/api/modules/ember-routing.html)

**3. 如何修改使用命令`ember build --prod`打包后的静态文件的名字，每个名字中会自动生成一个hash值一个类似于md5加密后的密文。**

**4. 项目安装出错，报错信息为无法访问github**

```bash
Error creating new application. Removing generated directory `./super-rentals`
Failed to execute "git ls-remote --tags --heads https://github.com/ember-cli/ember-cli-shims.git", exit code of #128
fatal: unable to access 'https://github.com/ember-cli/ember-cli-shims.git/': Failed to connect to github.com port 443: Connection refus
ed

Error: Failed to execute "git ls-remote --tags --heads https://github.com/ember-cli/ember-cli-shims.git", exit code of #128
fatal: unable to access 'https://github.com/ember-cli/ember-cli-shims.git/': Failed to connect to github.com port 443: Connection refus
ed

    at createError (D:\Program Files\nodejs\node_global\node_modules\ember-cli\node_modules\bower\lib\util\createError.js:4:15)
    at ChildProcess.<anonymous> (D:\Program Files\nodejs\node_global\node_modules\ember-cli\node_modules\bower\lib\util\cmd.js:102:21)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at maybeClose (internal[擦汗]ild_process.js:827:16)
    at Process.ChildProcess._handle.onexit (internal[擦汗]ild_process.js:211:5)
```
解决办法：直接从github下载缺失的依赖包，比如上述错误是缺少`ember-cli-shims`，可以尝试直接从`https://github.com/ember-cli/ember-cli-shims.git`下载，然后复制到`bower_components`目录之下。



参考网址：[https://github.com/ember-cli/ember-cli/issues/1418](https://github.com/ember-cli/ember-cli/issues/1418)


**欢迎补充**
