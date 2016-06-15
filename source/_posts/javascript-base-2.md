---
title: Javascript的匿名函数 
date: 2016-2-20
desc: Javascript  匿名函数 
---
匿名函数在javascript中非常常见且实用，它最大的用途是创建闭包（这是JavaScript语言的特性之一），并且还可以构建命名空间，以减少全局变量的使用。javascript的框架这种用法随处可见。下面看二个例子，如何定义匿名函数。
<!-- more -->
``` Javascript
	var f = function(t){
        alert(t);
    };
	f("abc");
```
“=”右边的函数就是一个匿名函数，创造完毕函数后，又将该函数赋给了变量f。通过f再调用这个匿名函数。

``` Javascript
	(function(t){
		alert(t)
    })("abc");
```
这里创建了一个匿名函数(在第一个括号内)，第二个括号用于调用该匿名函数，并传入参数。
这就是匿名函数常用的使用方式，通常用得最多的是第二种。

前端人员一般为了避免声明了一些全局变量而污染，把代码放在一个“沙箱执行”，然后在暴露出命名空间（可以为API，函数，对象）,如Jquery:

``` Javascript
	(function( window, undefined ) {
	    window.jQuery = window.$ = jQuery;
	})( window );
```

再如我想建一个自己的框架叫yb:

``` Javascript
	(function(window,undefined){
		var yb = {
			add:function(){
				alert("add");
			},
			sub:function(){
				alert("sub");
			}
		};
		window.yb = yb;
	})(window);
```
你的方法add和sub只能通过全局对象yb访问，这个是函数闭包规则决定的。如不能理解闭包请参见[函数的闭包](http://binyanbin.github.io/2016/01/03/javascript-base-1/);





