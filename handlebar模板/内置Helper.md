# 内置Helper方法
上一节我们学习了如何编写一个Helper方法.一个Helper方法通常是在任何模板内可以使用的一些简单方法.Ember自带了一些可以使你开发模板变简单的Helper方法,这些方法允许你更灵活的向其他Helper方法或组件中传输数据.
## 使用Helper方法动态的获取属性
`{{get}}`方法可以帮助你动态地发送一个变量的值到另一个Helper方法或组件中,如果你想输出某些`计算属性`中的一个值的时候,这会非常有用.
```hbs
{{get address part}}
```
如果计算属性`part`返回值为`zip`,那这条语句会显示`this.get('address.zip')`的结果;如果返回值为`city`,那便会显示`this.get('address.city')`的结果.
## 内置Helper方法的嵌套
上一节我们提到过Helper方法允许嵌套使用,内置的方法同样可以.举个栗子,`{{concat}}`可以将参数中的数字动态地发送到组件或Helper方法中,然后将其合并为一个独立的参数.
```hbs
{{get "foo" (concat "item" index)}}
```
当`index`为1时,这条语句会显示`this.get('foo.item1')`的结果;当`index`为其他值时,以此类推.
