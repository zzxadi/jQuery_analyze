## jQuery分析(1)

打开jQuery未压缩版，首先是一个[立即执行函数](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)
```js
(function ( window, undefined ) {

	// jquery code

})( window );
```

1. 使用立即执行函数主要是为了创造私有的命名空间，这个在js中使用很普遍，因为js中很容易发生给变量或者函数命名相同的情况，而js中又不存在重载(overload)和重复声明这种情况。另外也可以使单元测试变得简单。

2. 看函数传入的参数和接受的参数，只传入一个`window`却接受`window`和`undefined`两个参数，首先传入参数`window`(也可以是this)，有缩短作用域和方便压缩的作用，而没有传入`undefined`参数，是为了避免`undefined`在某些不标准的浏览器中可以被重写，因为未定义的变量默认值就是`undefined`，但是在ECMAScript 5中已经规定`undefined`不能被重写。

接下来是在函数中定义一些变量(比如`rootjQuery`，`core_version`等)，这里我们着重要关注的是
```js
jQuery = function( selector, context ) {
	// The jQuery object is actually just the init constructor 'enhanced'
	return new jQuery.fn.init( selector, context, rootjQuery );
}
```
这说明什么，这说明jQuery对象是通过`jQuery.fn.init`方法来构造的,其中传入了两个形参(`selector`和`context`)。

那jQuery为什么要这样构造呢？先看看我们最平常的方式(模仿jQuery)
```js
var jQuery = function( selector, context ) {
   //构造函数 
}

jQuery.prototype.name = function() {

}

jQuery.prototype.age = 24;

var jQueryObj = new jQuery();
jQueryObj.name();
```
很明显我们并没有先new一个jQuery对象，然后再使用，而是类似这样使用的(`jQuery().html()`等价于`$().html()`)，要弄成这样的，`jQuery()`必须是一个类的实例，这样才能拥有属性和方法，所以改进后可能是这样的
```js
var jQuery = function( selector, context ) {
   //构造函数 
   return new jQuery();
}

jQuery.prototype.name = function() {

}

jQuery.prototype.age = 24;

var jQueryObj = jQuery();
jQueryObj.name();
```
但这很明显会出错，是一个死循环。

那么利用它的原型，这种方式呢？
```js
var jQuery = function(selector, context) {
    return jQuery.fn.init();
}

jQuery.fn = jQuery.prototype = {
    init:function(){
        return this;
    },
    name:function(){},
    age:24
}

var jQueryObj = jQuery();
jQueryObj.name();
```
这有一个问题，不知有没有看出来，就是这里所有对象的this都是jQuery实例，属性和方法是共享的，修改其中一个，其它的都会发生修改。所以为每个对象分配一个空间，不能共享属性和方法。于是就有了
```js
var jQuery = function(selector, context) {
    return  new jQuery.fn.init();
}

jQuery.fn = jQuery.prototype = {
    init:function(){
        return this;
    },
    name:function(){},
    age:24
}

var jQueryObj = jQuery();
jQueryObj.name();
```
很明显这个会抛错，因为对象根本没有name方法和age属性，于是我们又修改了一下
```js
var jQuery = function(selector, context) {
    return  new jQuery.fn.init();
}

jQuery.fn = jQuery.prototype = {
    init:function(){
        return this;
    },
    name:function(){},
    age:24
}

jQuery.fn.init.prototype = jQuery.prototype;

var jQueryObj = jQuery();
jQueryObj.name();
```
这样就完美了，既不会抛错，又不会对象之间相互影响，可以看到jQuery就是这样实现的。

再看在jQuery源码中最后几行
```js
if ( typeof window === "object" && typeof window.document === "object" ) {
	window.jQuery = window.$ = jQuery;
}
```
这说明如果`window`和`window.document`是对象，则在window上定义`jQuery`和`$`属性，这两个属性和这个函数中的`jQuery`是相等的。所以我们在使用`jQuery`时，`$('#id')`与`jQuery('#id')`是等价的。而且我们使用$()可以接受两个参数，一个就是平常接受的选择器，另一个就是这个选择器的作用环境。

