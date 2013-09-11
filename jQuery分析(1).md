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
所以jQuery.fn和jQuery的原型都是一个对象，在这个对象中也定义了一些属性和方法，我们可以直接使用，比如$().length或者$().map()；

我们仔细看看jQuery.fn.init函数里到底做了什么
```js
function( selector, context, rootjQuery ) {
	var match, elem;

	// HANDLE: $(""), $(null), $(undefined), $(false)
	if ( !selector ) {
		return this;
	}

	// Handle HTML strings
	if ( typeof selector === "string" ) {
		if ( selector.charAt(0) === "<" && selector.charAt( selector.length - 1 ) === ">" && selector.length >= 3 ) {
			// Assume that strings that start and end with <> are HTML and skip the regex check
			match = [ null, selector, null ];

		} else {
			match = rquickExpr.exec( selector );
		}

		// Match html or make sure no context is specified for #id
		if ( match && (match[1] || !context) ) {

			// HANDLE: $(html) -> $(array)
			if ( match[1] ) {
				context = context instanceof jQuery ? context[0] : context;

				// scripts is true for back-compat
				jQuery.merge( this, jQuery.parseHTML(
					match[1],
					context && context.nodeType ? context.ownerDocument || context : document,
					true
				) );

				// HANDLE: $(html, props)
				if ( rsingleTag.test( match[1] ) && jQuery.isPlainObject( context ) ) {
					for ( match in context ) {
						// Properties of context are called as methods if possible
						if ( jQuery.isFunction( this[ match ] ) ) {
							this[ match ]( context[ match ] );

						// ...and otherwise set as attributes
						} else {
							this.attr( match, context[ match ] );
						}
					}
				}

				return this;

			// HANDLE: $(#id)
			} else {
				elem = document.getElementById( match[2] );

				// Check parentNode to catch when Blackberry 4.6 returns
				// nodes that are no longer in the document #6963
				if ( elem && elem.parentNode ) {
					// Inject the element directly into the jQuery object
					this.length = 1;
					this[0] = elem;
				}

				this.context = document;
				this.selector = selector;
				return this;
			}

		// HANDLE: $(expr, $(...))
		} else if ( !context || context.jquery ) {
			return ( context || rootjQuery ).find( selector );

		// HANDLE: $(expr, context)
		// (which is just equivalent to: $(context).find(expr)
		} else {
			return this.constructor( context ).find( selector );
		}

	// HANDLE: $(DOMElement)
	} else if ( selector.nodeType ) {
		this.context = this[0] = selector;
		this.length = 1;
		return this;

	// HANDLE: $(function)
	// Shortcut for document ready
	} else if ( jQuery.isFunction( selector ) ) {
		return rootjQuery.ready( selector );
	}

	if ( selector.selector !== undefined ) {
		this.selector = selector.selector;
		this.context = selector.context;
	}

	return jQuery.makeArray( selector, this );
}
```
我们发现主要是分为四种大情况处理了传入的selector,最后使用jQuery.makeArray()返回一个数组。

