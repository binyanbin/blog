---
title: jmeter himall电商抢购业务简单测试
date: 2016-5-13
desc: jmeter 电商抢购测试
---
Jmeter是一款比较好的测试工具，我以[Hishop](http://www.hishop.com.cn/)旗下[Himall](http://www.hishop.com.cn/products/himall/)(多用户商城产品)为例，使用jmeter对电商网站的抢购业务建立一个简单测试方案。
## [himall](http://www.hishop.com.cn/products/himall/)抢购业务的测试
测试要求:模拟1000用户同时登陆himall商城，同时购买固定活动商品库存，全部下单完成之后，去验证结果。
整个测试摸拟过程为:
* 登陆 
* 频刷活动商品页面 
* 下单
<!-- more -->
## JMeter设置
整个测试的结构如下图

![图](/img/jmeter-6.png)

* 线程组:模拟用户的数量
* 聚合报告:测试的性能数据
* 查看结果树:具体每个请求的详细信息。
* 一个事务控件器:定义从登陆到下单的所有http请求
* HttpCookie管理器:登陆的用户信息在Cookie中，你需要它了保持cookie用户信息
* 用户定义变量:记录这次测试指定活动的产品ID及库存ID

1.设置模拟用户数1000
![图](/img/jmeter-7.png)

2.设置商品及库存ID
![图](/img/jmeter-8.png)
skuid为库存ID,productid为商品ID。此测试只操作单个库存

3.设置账号数据
![图](/img/jmeter-9.png)
准备1000个用户名和账号，以.csv保存，用户名与密码以','隔开。

4.设置登陆请求
![图](/img/jmeter-10.png)
设置登陆请求,ip和端口,路径。请求方法:post。重要的是BodyData的内容,其中包括用户名，密码及验证码。
__CSVRead是一个读csv文件数据的方法。
第一参数为目录:C:\Users\admin\Desktop\jmeterdata\account.csv这是我本地的账户信息文件路径。
第二参数指你要读第几列数据，列数由0开始。
checkCode 验证码验证
keep 是否保持登陆
* 注意:需要在网站代码中注释掉验证环节才可以测试，否则你无法登陆成功。

4.设置活动页面刷新请求
![图](/img/jmeter-11.png)
设置刷新活动页面次数.10次20次你随便设吧

![图](/img/jmeter-12.png)
刷活动设置比较简单，就是一个路径，${productid}代表是你自定义的产品ID,couts是购买数量。

5.设置提交订单请求
![图](/img/jmeter-13.png)
定单提交页面也是一个路径，这个请求参数相对较多一些，最重要的是设置好库存ID
integral:使用积分,没有为0
couponIds:红包ID,没有为空
skuIds:购买库存ID,${skuid}则是开始配置的库存ID
counts:购买数量
collpIds:组合购ID，没有为空
recieveAddressId:收货地区ID
invoiceType:发票类型 0 不要发票 1 增值税发票 2普通发票
&invoiceTitle:发票抬头,没有为空
invoiceContext:发票内容，没有为空
isCashOnDelivery:是否货到付款

设置完成，你现在可以泡杯茶坐等结果。当然你也可以修改相应相关参数，如摸拟用户数，指定活动页面的刷新次数等，来看看结果会有哪些不同。

