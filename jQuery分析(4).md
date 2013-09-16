## jQuery分析(4)

前面有很多地方用到了工具函数，比如`jQuery.type()`,`jQuery.isFunction()`等，下面我们挑几个讲讲。

首先最关键的就是无处不在的判断类型，在jQuery中是怎样实现的呢？
```js
var class2type = {}, 
    core_toString = class2type.toString;

isFunction: function( obj ) {
	return jQuery.type(obj) === "function";
},

isArray: Array.isArray,

isWindow: function( obj ) {
	return obj != null && obj === obj.window;
},

isNumeric: function( obj ) {
	return !isNaN( parseFloat(obj) ) && isFinite( obj );
},

type:b function( obj ) {
	if ( obj == null ) {
		return String( obj );
	}
	// Support: Safari <= 5.1 (functionish RegExp)
	return typeof obj === "object" || typeof obj === "function" ?
		class2type[ core_toString.call(obj) ] || "object" :
		typeof oj;
}

// Populate the class2type map
jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
	class2type[ "[object " + name + "]" ] = name.toLowerCase();
});
```
其实就是用了简单的类属性，你可以参考玉伯的这篇文章[Sea.js 源码解析（三）](https://github.com/lifesinger/lifesinger.github.com/issues/175),另外关于`isNumeric()`可以参考stackoverflow上的一篇解答[Validate numbers in JavaScript - IsNumeric()](http://stackoverflow.com/questions/18082/validate-numbers-in-javascript-isnumeric)。





