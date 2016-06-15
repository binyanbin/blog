---
title: .net由内存模型说性能1
date: 2016-4-30
desc: gc C# 性能优化
---
.net不必手工管理内存，但要编写高性能的代码，就仍需理解后面发生的事情。

## 内存模型:堆与栈
C#内存主要有两类：Stack和Heap
Stack叫做栈区，由编译器自动分配释放，存放函数的参数值，局部变量的值等。
Heap则称之为堆区，由开发人员申请内存，在垃圾回收器的控制下工作。
![效果](/img/CSharp-1.jpg)

<!-- more -->
## 值类型与引用类型
值类型的数据和内存在同一个位置，而引用类型是一个指向内存的指针。也就是对Stack和Heap的实际使用。
![效果](/img/CSharp-2.jpg)
![效果](/img/CSharp-3.jpg)
除了Object和String,其它都是值类型。值类型的性能要略优于引用类型，它只需要一次访问内存就可拿到数据。

## 装箱与拆箱
值类型与引用类型进行类型转换时会产生装箱和拆箱操作。
![效果](/img/CSharp-4.jpg)
非必要时尽量避免这类操作，会对性能一定的影响。

## 垃圾回收器GC
垃圾回收器为什么不会立即回收对象?
对象不再被引用时,如果立即删除,堆上的自由空间就会分散开来，给新对象分配内存就会很难处理,程序必须搜索整个堆才能找到一块足够大的内存块来存储整个新对象。
整个heap中对象的引用关系错综复杂（交叉引用、循环引用），形成复杂的graph。heap对应有一个Roots，它能使程序在heap之外可以找到的各种入口点。root包括:全局对象、静态变量、局部对象、函数调用参数、当前CPU寄存器中的对象指针。
下面可以通过一组图片来进一步了解GC的工作。黄色代表还在引用的对象，灰色代表没有引用的对象。
现在内存使用已达到阀值,GC需要清理内存，下图就是GC整个清理的过程。
![效果](/img/CSharp-5.gif)
![效果](/img/CSharp-6.gif)
![效果](/img/CSharp-7.gif)
![效果](/img/CSharp-8.gif)
GC回收内存是一种非常耗费性能的工作，减少不必要的内存使用有助于提高GC性能

## 传参的优化
如果我们要将一个非常大的值类型数据(如数据量大的struct类型)入栈，它会占用非常大的内存空间，而且会占有过多的处理器资源来进行拷贝复制。
``` CSharp
public struct ConstNum
{
	long a,b,c,d,e,f,g,h,i,j,k;
}

public  void Go()
{
	ConstNum x = new ConstNum();
	Do(x);
}

public  void Do(ConstNum x)
{
	//do something
}
```
可以将struct改为class。实例对象之后则是引用类型。或使用ref关键字将方法改为Do(ref ConstNum x)，也可以达到引用类型的作用。

## 静态
有一个Dude类，你需要实例化多个对像使用。
``` CSharp
Class Dude
{
	private _name ="test";
	public void SayHello()
	{
		//DoSomething
	}
}
```
![效果](/img/CSharp-9.gif)
你可以使用静态方法,以达到内存节省的目的。
``` CSharp
Class Dude
{
	private _name ="test";
	public static void SayHello()
	{
		//DoSomething
	}
}
```
![效果](/img/CSharp-10.gif)
实际项目中使用单例模式来达到省内存的目的。

## IDisposable与析构函数
托管的资源只能由CG回收。而非托管的资源可以通过实现IDisposable进行释放。
``` CSharp
public class CDisposable : IDisposable
{
    //析构函数，编译后变成 protected void Finalize()，GC会在回收对象前会调用调用该方法 
    ~CDisposable() 
    { 
        Dispose(false); 
    } 

    //通过实现该接口，显式地释放对象，只针对非托管对象。
    void Dispose() 
    { 
        Dispose(true); 
    } 

}
```

Dispose调用

``` CSharp
Using(MyClass myObj = new CDisposable())
{ 
	//DoSomething
}
```

或者直接调myObj.Dispose();


## 字符串的内存规则
string是引用类型，但string的值是不可变，但你改变string值的时候会对新值重新分配内存,老值仍驻留在内存中。

```CSharp
string a = "1234";
a += "5678";
Console.ReadLine();
```

看起来我们似乎已经把a的值从“1234”改为了“12345678”，实际上并没有改变。因为string的值是无法修改不了了。堆中其实存在着两个字符串对象。字符串“1234”仍然在内存中驻留。

string容易引起内存驻留，但我们仍有办法减少内存使用。

``` CSharp
string str1 = "ABCD1234";
string str2 = "ABCD1234";
string str3 = "ABCD" + "1234"; 
string str4 = "1234";
string str5 = "ABCD" + str4;
object.ReferenceEquals(str1, str2) == True；
object.ReferenceEquals(str1, str3) == True；
object.ReferenceEquals(str1, str5) == False
```
通过上面比较，字符串都驻留在内存。尽量使用字符串相加来代替字符串变量和字符创相加，这样可以使用重复利用字符串驻留，减少内存使用。

``` CSharp
string str3 = "ABCD" + "1234"; 
```

对string作频繁的操作使用StringBuilder或string.format(也是StringBuilder实现)处理。
StringBuilder内部维护一个字符数组，而不是一个string来避免string操作带来的新的string的创建。

``` CSharp
StringBuilder sb = new StringBuilder();
sb.Append(str1);
sb.Append(str2);
sb.Append(str3);
``` 

``` CSharp
string sb = string.format("{0}{1}{2}",str1,str2,str3);
``` 

相当于sb = str1 + str2 + str3;

4.高效地进行string的比较操作
对象之间的比较有比较Value和比较Reference之说。一般地对Reference进行比较的速度最快，因为只需要比较一下是不是同一地址就行了。
object.ReferenceEquals和string. Compare就是引用比较的方法。