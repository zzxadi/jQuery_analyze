## jQuery分析(1)

打开jQuery未压缩版，首先是一个[立即执行函数](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)
```js
(function ( window, undefined ) {

	// jquery code

})( window );
```

1. 使用立即执行函数主要是为了创造私有的命名空间，这个在js中使用很普遍，因为js中很容易发生给变量或者函数命名相同的情况，而js中又不存在重载(overload)和重复声明这种情况。另外也可以使单元测试变得简单。

2. 看函数传入的参数和接受的参数，只传入一个window却接受window和undefined两个参数，首先传入参数window(也可以是this)，有缩短作用域和方便压缩的作用，而没有传入undefined参数，是为了避免undefined在某些不标准的浏览器中可以被重写，因为未定义的变量默认值就是undefined，但是在ECMAScript 5中已经规定undefined不能被重写。

接下来是在函数中定义一些变量(比如rootjQuery，core_version等)，这里我们着重要关注的是
```js
jQuery = function( selector, context ) {
	// The jQuery object is actually just the init constructor 'enhanced'
	return new jQuery.fn.init( selector, context, rootjQuery );
}
```
这说明什么，这说明jQuery对象是通过jQuery.fn.init方法来构造的,其中传入了两个形参(selector和context)。

再看在jQuery源码中最后几行
```js
if ( typeof window === "object" && typeof window.document === "object" ) {
	window.jQuery = window.$ = jQuery;
}
```
这说明如果window和window.document是对象，则在window上定义jQuery和$属性，这两个属性和这个函数中的jQuery是相等的。所以我们在使用jQuery时，$('#id')与jQuery('#id')是等价的。而且我们使用$()可以接受两个参数，一个就是平常接受的选择器，另一个就是这个选择器的作用环境。

我们看jQuery.fn.init又是如何定义的，看第96行
```js
jQuery.fn = jQuery.prototype = {
	jquery: core_version,
	constructor: jQuery,
    init: function( selector, context, rootjQuery ){
      
    	//some code  
    }
}
```

