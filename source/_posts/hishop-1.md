---
title: 浅谈Himall商城限时购设计
date: 2016-4-21
desc: 抢购 并发下单 
---
互联网正在高速发展，使用互联网服务的用户越多，高并发的场景也变得越来越多。[Himall](http://www.hishop.com.cn/products/himall/)限时购功能则是一个典型的短时间高并发场景。虽然我们解决问题的具体技术方案可能千差万别，但是遇到的挑战却是相似的，因此解决问题的思路也异曲同工。
什么是限时购?限时购跟大部分电商抢购业务相同,即限时且限量抢购。不管小米还是华为，或是其它电商公司，对抢购业务运营总是最为火爆，每发一款新品，都限量发售，每次搞的大家心里痒痒的。抢购太火爆有时引起站点打不开，崩溃了;还有就是卖出的数量比设置可购买的数量要多。那么问题来了：我们如何在设计中如何解决。通常我们需要从设计中考虑以下问题:
* 针对高并发，我们如何解耦后端压力，特别是数据库的压力。
* 如何保障库存可靠。
<!-- more -->

我们可以试想一下抢购时哪些页面会请求最多。抢购之前人们通常会通常刷页面等待，一般在抢购开始前一点时间会频繁刷新抢购倒数的页面或购买详情页面。抢购开始以后前一段时间下单的人会很多。付款并发量相对较小，通常订单在下单后几小时内都能付款，缓解了并发压力。针对以上问题及场景，我们做了以下处理，增加限时购缓存订单系统，去支持限时购高并发处理，并保持限时购业务的可靠性。
![图1](/img/hishop-1.png)

[Hiamll](http://www.hishop.com.cn/products/himall/)在2.3版本做了如下改进:
1.引入Redis做缓存。
2.在用户抢购开始前频繁刷页面时,系统只从缓存中取商品数据，解耦了数据库查询的压力。
3.用户下单时系统只把订单数据存入订单缓存队列后然后告诉用户你的订单正在处理。然后由Redis Pub/Sub服务通知Web服务器，服务器把库存订单进行串行化处理，解耦数据库并发下单压力，保证库存可靠。
4.支付功能保持原来实现不变。

具体实现如下:
买家前端查询限时购商品数据时只走缓存。
![图2](/img/hishop-2.png)
卖家后台更新限时购或库存信息时需同步更新数据库及缓存。
![图3](/img/hishop-3.png)
系统为每个正在开卖的限时购商品库存创建锁，买家对某库存下单时锁住该库存的下单操作，每一个商品库存只允许一个会员下单，下单的订单数据直接加入订单缓存后告诉买家[您的订单正在处理,请稍等]。然后通过Redis Pub/Sub服务通知服务器处理订单，将订单按库存串行化处理，订单处理完成后，则更新限时购订单缓存的处理状态。
![图4](/img/hishop-4.png)
买家得知订单正在处理后，则不断查询缓存的订单处理状态。直到获取订单处理结果，下单成功则进行支付页面，失败则提示失败原因并引导买家重新下单。
![图5](/img/hishop-5.png)
最后就是在Web服务启动时，需要对限时购订单缓存系统初始化，把商品数据加入缓存中，并处理上次未处理完成的订单。
![图6](/img/hishop-6.png)

总结:无论你用什么方式处理性能问题，性能优化的核心思想是分治。这种思想在日常生活中无处不在，大家都知道一次做不了的事，就分多次做，这就是分治。