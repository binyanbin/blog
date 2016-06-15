---
title: Javascript模块化开发
date: 2016-3-21
desc: Javascript  模块化 
---
Javascript不算模块化编程语言，但是web开发需要团队开发和团队协做。现在javascript模块化开发已非常正熟，支持模块化开发的框架非常多。

## 原始写法
``` Javascript
    function func1(){
        ...
    }

    function func2(){
        ...
    };
```
只要把不同的函数简单地放在一个文件中，就算是一个模块，通常一个文件一个模块。
<!-- more -->

## jquery时代
``` Javascript
　　var module1 = (function ($, windows) {
　　　　//...
　　})(jQuery, windows);
```
通过立即执行匿名函数来达到封装的作用,如模块内部调用全局变量，则将其输入模块中。

## commonjs时代
commonjs规范是目前JavaScript模块化的事实标准。支持commonjs规范的框架及程序比较多，如node,seajs.规范中最重要的二点:
* require它是一个函数,引用其它模块使用require。
* exports是一个对象，导出模块api使用exports，可供其它模块调用。
当然除了这两点还有一些其它细节不一一介绍。下面看一个示例:

math.js
``` Javascript
    exports.add=function(x,y){
        return x+y;
    };
```
increment.js
``` Javascript
    var add = require('math').add;
    exports.increment = function (val){
        return add(val,1);
    }
```

program.js
``` Javascript
    var inc = require('increment').increment;
    var a = 1;
    inc(a); 
```

