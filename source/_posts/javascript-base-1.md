---
title: Javascript的闭包 
date: 2016-1-3
desc: Javascript  闭包
---
闭包是Javascript的一个重点的概念，在开发过程中用得非常多，但是在了解闭包之前我们需要了解全局与局部的概念，下面来看这个例子。
<!-- more -->
``` Javascript
	var global = 1;
    function myfun()
    {
    	var my = 1;
		function infun()
		{
			var in = 2;
			function innerfun()
			{
				var inner = 2;
			}
		}

		function infun2()
	   ｛
	   			   
		｝
    }

    function myfun2()
    {

    }
```
这是一个三层嵌套的函数。我对局部的了解是指函数的内部，局部对像就是指函数内部的变量及函数。全局对象就是没有定义在任何函数内的变量和函数。全局对象是window的子对象。

在上面的这个程序中，全局对象有:
*global变量
*myfun函数
*myfun2函数

myfun的内部对像有:
*my变量
*infun函数
*infun2函数

infun的内部对像有:
*in变量
*innerfun函数
以此类推。

了解了局部与全局后，闭包就容易了解,它是指内层函数可以使用外层函数局部对象，外层函数不能使用内层函数的局部对象，这种规则就叫函数的闭包。
从上面这个例子，我们就清楚在innerfun函数中，可以访问当前所有对像，因为它是最内层函数。而在infun函数中除了innerfun函数和它的内部变量inner不能访问外，其它的也都可以访问。
大家都可以访问全局对象，因为它在最外层。这就是为什么很多js库总是建一个对象放到window下做全局对象，你只要引用就可以访问到它。

补充:没有使用var进行定义的变量也是全局对象。例如:

``` Javascript
    function myfun()
    {
        var part ="part variable";
        global  ="global variable";
    }
```





