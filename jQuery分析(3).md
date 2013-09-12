## jQuery分析(3)

现在来讲一下`jQuery.extend()`函数，这一个函数很好很强大,jQuery中很多方法都是通过此函数来添加的，而且你如果要编写jQuery插件，也离不开它。
```js
jQuery.extend = jQuery.fn.extend = function() {
	var options, name, src, copy, copyIsArray, clone,
		target = arguments[0] || {},
		i = 1,
		length = arguments.length,
		deep = false;

	// Handle a deep copy situation
	if ( typeof target === "boolean" ) {
		deep = target;
		target = arguments[1] || {};
		// skip the boolean and the target
		i = 2;
	}

	// Handle case when target is a string or something (possible in deep copy)
	if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
		target = {};
	}

	// extend jQuery itself if only one argument is passed
	if ( length === i ) {
		target = this;
		--i;
	}

	for ( ; i < length; i++ ) {
		// Only deal with non-null/undefined values
		if ( (options = arguments[ i ]) != null ) {
			// Extend the base object
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// Prevent never-ending loop
				if ( target === copy ) {
					continue;
				}

				// Recurse if we're merging plain objects or arrays
				if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
					if ( copyIsArray ) {
						copyIsArray = false;
						clone = src && jQuery.isArray(src) ? src : [];

					} else {
						clone = src && jQuery.isPlainObject(src) ? src : {};
					}

					// Never move original objects, clone them
					target[ name ] = jQuery.extend( deep, clone, copy );

				// Don't bring in undefined values
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
			}
		}
	}

	// Return the modified object
	return target;
}
```
你看，这个方法很简单吧，这里关键有一个`target`，什么是`target`呢？就是传入的第一个参数，如果这个参数是布尔值`true`，那么表明是深度拷贝，举个例子。
```js
var result1 = $.extend({}, 
{name: "zzxadi", location: {country: "china", state: "renming"}}, 
{name: "zhazhexio", location: {country: "china", city: "beijing"}}),
result2 = $.extend(true, {}, 
{name: "zzxadi", location: {country: "china", state: "renming"}}, 
{name: "zhazhexio", location: {country: "china", city: "beijing"}});

console.dir(result1);//result1 = {name: "zhazhexio", location: {country: "china", city: "beijing"}}

console.dir(result2);//result2 = {name: "zhazhexio", location: {country: "china", state: "renming", city: "beijing"}}
```
具体的用法可以参考这篇文章[jQuery.extend 函数详解](http://www.cnblogs.com/RascallySnake/archive/2010/05/07/1729563.html)，这里的源码和注释也写得很详细，我就不多作介绍。