---
title: C#由内存模型说性能2 数组与集合
date: 2016-5-2
desc: gc C# 性能优化 集合
---
由于.net不会实时回收内存，那么.net对数组与集合是内存的是怎么处理的

## 数组与集合
大家都知道数组必须指定大小，而且大小一但指定就不能更改了，也就是说数组不能动态的增加容量，那么对于一些需要动态增加容量的需求是实现不了的。实现动态扩容的需求都是通过集合,如List,Dictionary,ArrayList.
<!-- more -->

## 集合如何实现动态扩容
拿ArrayList和List为例,实际的代码大概如下:

``` CSharp
/// <summary>
/// 增加元素
/// </summary>
public virtual int Add(object value)
{
	//超出集合大小
    if (this._size == this._items.Length)
    {
        this.EnsureCapacity(this._size + 1);
    }
    this._items[this._size] = value;
    this._version++;
    return this._size++;
}
/// <summary>
/// 设置容量大小 默认容量为4,超出容量则扩容2倍
/// </summary>
private void EnsureCapacity(int min)
{
    if (this._items.Length < min)
    {
    	//默认容量为4,超出容量则扩容2倍
        int num = (this._items.Length == 0) ? 4 : (this._items.Length * 2);
        if (num < min)
        {
            num = min;
        }
        this.Capacity = num;
    }
}
/// <summary>
/// 设置容量大小
/// </summary>
public virtual int Capacity
{
    get
    {
        return this._items.Length;
    }
    set
    {
        if (value != this._items.Length)
        {
            if (value < this._size)
            {
                throw new ArgumentOutOfRangeException("value", Environment.GetResourceString("ArgumentOutOfRange_SmallCapacity"));
            }
            if (value > 0)
            {
                object[] array = new object[value];//重新分配内存
                if (this._size > 0) 
                {
                    Array.Copy(this._items, 0, array, 0, this._size); 
                }
                this._items = array;
                return; 
            } 
            this._items = new object[4]; 
        } 
    } 
}
```

测试

``` CSharp
List<int> list = new List<int>();//list.Capacity=0
list.Add(new int());//list.Capacity=4
list.Add(new int());//list.Capacity=4
list.Add(new int());//list.Capacity=4
list.Add(new int());//list.Capacity=4
list.Add(new int());//list.Capacity=8
```

## 总结
* 无论是List,ArrayList或其它集合,不过是对Array的一层包装,也由此可以断定集合的性能肯定不如Array。
* 集合的扩容过程增加CPU的损耗和GC的压力，对于问题的严重性就取决于实际应用的场合，如果在高并发的应用下存在大量这操作那问题就变得严重多了。
* 使用:对集合初始化的时候指定容量,如果可以直接使用Array代替集合。