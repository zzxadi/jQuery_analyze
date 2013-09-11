## jQuery分析(2)

下面重点看看
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
`selector`分为四种大情况,
1. 当`selector`为`null`,`""`,`false`,`undefined`时，直接返回`this`。
2. 当`typeof selector === string`时，这种情况最复杂，一般我们写的选择器也是使用这个来处理的。
3. 当`selector`为DOMElement时，就将`context`置为当前选择的元素,`length`置为1,比如使用`$(document.getElementById('#id'))`。
4. 当`selector`是一个函数时,就把`selector`加入`rootjQuery.ready()`中，这里的`rootQuery`(看源码的第886行)就是`$(document)`,所以使用`$(function(){})`和`$(doucument).ready(function(){})`效果一样。

就第二种情况单独拿出来分析，`$('<div>hello,world</div>')`,`$('#id .class')`,`$('div .class')`都会使`typeof selector === string`。首先判断`selector`是不是HTML字符串(形如< >)，如果不是就分别处理，这个可以直接看注释。





