---
title: 电商面试题
date: 2016-2-18
desc: 电商 抢购 超卖
---
15年年底去一家电商服务公司面试，其中有一道题目是如何解决电商站点商品秒杀的相关问题。[面试题目可以参见](https://github.com/hishopdc/dc2015)。固定的数据库结构下实现三个接口，查看、下单、付款。
<!-- more -->
## 主要问题:高并发查询及更新数据库
解决的方案其实比较其实并不复杂，也很常规。无非就是用缓存队列实现批量更新数据库。把下单的数据放入队列中，然后批量更新到数据库中去。说起来比较简单，实践中还是踩了一些坑。

### 主要实现逻辑

订单处理类
``` CSharp
    /// <summary>
    /// 订单缓存队列处理类
    /// </summary>
    public class OrderCacheQuene
    {
        /// <summary>
        /// 订单队列缓存
        /// </summary>
        private static List<Order> _orders;
        /// <summary>
        /// 数据库操作接口
        /// </summary>
        private static IPromotion _service;
        /// <summary>
        /// 锁
        /// </summary>
        private static object _locker = new object();
        /// <summary>
        /// 队列处理线程
        /// </summary>
        private static Thread _thread;

        /// <summary>
        /// 订单队列启动
        /// </summary>
        public static void Start()
        {
            _thread = new Thread(new ThreadStart(OrderDispose));
            _thread.Start();
        }

        /// <summary>
        /// 缓存结束
        /// </summary>
        public static void End()
        {
            lock (_locker)
            {
                _thread.Abort();
                _service.OrderTrans(_orders);
                _orders.Clear();
            }
        }


        /// <summary>
        /// 下单
        /// </summary>
        public static string OrderBuy(Order o)
        {
            lock (_locker)
            {
                _orders.Add(o);
            }
            return o.OrderId;
        }

        /// <summary>
        /// 支付
        /// </summary>
        public static bool OrderPay(RequestPay o, DateTime? paytime)
        {
            lock (_locker)
            {
            	Order order = null;
                int index = _orders.FindIndex(t => t.OrderId == o.order_id && t.UserId == o.uid);
                if (index >= 0)///已存在缓存中
                {
                    _orders[index].PayTime = paytime;
                    return true;
                }            	
                order = _service.GetOrder(o.order_id);
                if (order == null)
                    return false;
                else///已存在数据库中
                {
                    order.PayTime = paytime;
                    lock (_locker)
                    {
                        _orders.Add(order);
                    }
                    return true;
                }
            }
        }


        /// <summary>
        /// 判断队列是否已满
        /// </summary>
        public static bool isFull()
        {
            return _orders.Count >= Constant.QueueMaxCount ? true : false;
        }

        /// <summary>
        /// 订单队列处理事件
        /// </summary>
        private static void OrderDispose()
        {
            while (true)
            {
                //DateTime starttime = DateTime.Now;
                lock (_locker)
                {
                   List<Order> orders =  _orders.Take(Constant.MaxOrderDispose).ToList();
                    _service.OrderTrans(orders.ToList());//调用数据库接口处理订单
                    foreach (Order o in orders)
                        _orders.Remove(o);
                }
                Thread.Sleep(Constant.QueueDisposeTimeSpan);//每次处理完成休眠
                //DateTime endtime = DateTime.Now;
                //System.Diagnostics.Debug.WriteLine("Yanbin TimeSpan:"+endtime.Subtract(starttime).Milliseconds);
            }
        }
    }
```

缓存我直接使用的静态对像，效果是一样的。批量提交的方法是 _service.OrderTrans，在这里就不列出来，但是一些数据库操作。

订单队列启动与结束

``` CSharp
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            //订单队列初始化
            OrderCacheQuene.Start();
        }

        protected void Application_End()
        {
            //结束处理
            OrderCacheQuene.End();
        }
    }
```

### 坑1:超卖
程序实现以后，我写了一个测试程序去跑它，跑到200线程时超卖了，思来想去之后发现是判断的问题，开始下单时的判断条件没有放到订单队列程序里面。因为并发大，所以有一些客户端请求下单的时候,程序判断是满足下单条件,但是还有一些客户端的请求正在响应，把订单加入缓存了，造成判断不一致，最终引起超卖。所以下单的条件判断需要加到锁里面，虽然对性能上有一些损耗，但是可以保证不会超卖。

``` CSharp
        /// <summary>
        /// 下单
        /// </summary>
        public static string OrderBuy(Order o)
        {

            lock (_locker)
            {
	            if (队列已满，已卖空，已下单等条件)
	            	return null;            	
                _orders.Add(o);
            }
            return o.OrderId;
        }
```

### 坑2:效率坑
做web接口，在.net的系统中当然用webapi,更种好处用过都知道，没想到在高并发的条件下效率不如handler。以下这些数据是在我这台旧的笔记本上测试出来的。

|线程数|线程调用接口数|处理方式|说明|产生订单数|接口方式|完成时间(秒)|完成时间(分钟)|处理请求总数|平均每秒处理请求|
|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|2000|100|只查询|2000人同时查100种商品|0|handler|650.5|10.84|199706|307.004|
|2000|100|只查询|2000人同时查100种商品|0|api|1180.8|19.68|199747|169.162|
|1000|50|只查询|1000人同时查50种商品|0|handler|86.3|1.44|49932|578.586|
|1000|50|只查询|1000人同时查50种商品|0|api|103.9|1.73|49948|480.731|
|2000|10|抢购|2000人抢100个订单|100|handler|81.9|1.37|40430|493.651|
|2000|10|抢购|2000人抢100个订单|100|api|79.7|1.33|28738|360.577|
|2000|50|并发购买|50个人同时买2000种不同商品|100000|Handler|3467|57.78299471|86.378|
|2000|50|并发购买|50个人同时买2000种不同商品|100000|api|3638|60.63|299375|82.291|

从上图可以看出，在高并发情况下webapi的性能确实不行。

### 开发总结

* 锁在高并发的情况下的使用，如何满足业务要求。
* 缓存的使用，缓存与数据库之间的数据如何保持一致。
* 对于大型电商网站，缓存队列需要设置上限，不然会引起内存问题，不过在这里不是这道题目的重点。

[源码下载](https://github.com/binyanbin/interview/)

