---
title: Javascript的面向对象编程
date: 2016-2-21
desc: Javascript  面向对象 
---
Javascript里的所有东西都是对象，可是它又不并像Java,C#。不是严格意义上的OOP语言。但是它可以实现OOP的效果。但在之前你需要了解哪些关键的知识点?

## 关键知识点

### this
我们看下面这个使用this的例子

``` Javascript
    var obj = {
        name:"yanbin",
        showName :function(){
            alert(this.name);
        }
    };

    var other ={
        name:"changsha",
        showName:obj.showName
    };
```
obj.showName()提示的是yanbin,obj.showName()提示的是changsha，this是指执行时当前的对象。
<!-- more -->
``` Javascript
    function test(){
        return this;
    }
```

执行时函数不属于任何对象时,this表示window。(使用var定义的对象也属于window)

### new
在Javascript中,new一个函数它做了以下3件事:
* 创建新对象。
* 拷贝prototype到新对象。
* 设置构造函数
* 执行函数返回给新对象。
请看下面这个示例
``` Javascript
    function func(){
        this.name = 'yanbin';
        this.showcity =function(){
        	alert('changsha')
    	}
    }
    var  model =new func();
```
通过代码还原new的步骤
``` Javascript
var newobj = {};
newobj.__proto__ = func.prototype;
func.prototype.constructor = func;
p.apply(newobj)
var model = newobj;
```
这段代码我们需要搞清楚二个属性:
* prototype:在定义一个新函数，都会给函数创建一个prototype属性，也就是原型对象，把它看成普通对象也行了，但它可以影响这个函数实例的__protype__.
* \__protype\__:是函数实例的原型对象指针，每次new一个函数的时候都会给实例生成一个指针newobj.\__proto\__=func.protype。当函数的原型对象改变时，那么实例也会发生改变，因为\__proto\__只是指针，所以它也不能修改，只能对应的原型方法或属性访问它。
弄清楚了这些我们就可以开始OPP实践了。

## 封装示例
``` Javascript
    function person(name,sex,s)
    {
        this.name =name;
        this.sex = sex;

        function sayname()
        {
            alert("my name is "+ name +", i'm a "+sex);
        }

        this.say =function()
        {
            sayname();
        }

        var secret = s;
        this.getsecret = function(){
            return secret;
        }
        this.setsecret = function(ss){
            secret = ss;
        }
    }
    var model =new Person('yanbin','man','test');
```
name,sex是公开属性,sayname是一个私有方法，say是一个公共方法。secret是一个私有属性,对它的赋值和取值则是通过getsecret和setsecret这二个方法。

## 继承示例
``` Javascript
    function Man(n)
    {
        this.say = function()
        {
            alert("my name is:"+ this.realname);
        }
       
        this.realname = n;
    }

    Man.prototype = {
        realname : this.realname,
        say : this.say
    }

    function Employee(sex)
    {
        var sex = sex;
        this.getsex = function(){
           return sex;
        }
    }
    
    Employee.prototype = new Man('yanbin');
    var model = new Employee("female");
```
Employee继承man的say方法和realname属性，Emplayee有一个私有属性sex和一个getsex方法。