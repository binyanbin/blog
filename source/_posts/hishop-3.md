---
title: Himall集群版解决方案及性能测试结果
date: 2016-6-7
desc: 分布式架构 微服务
---
## 架构设计
集中架构设计出于系统的可扩展性、可维护性以及成本等多方面的因素考虑，逐渐被放弃，转而采用分布式的架构设计。常用分布式架构采用按模块纵向分解的方式,然后纵向分解中的某些模块仍有可能是一个复杂且庞大的业务，需要继续进行拆分。每个模块又由许多个微服务完成业务功能，微服务可独部署弹性伸缩。下图为Himall分布式集群版的整体架构设计:

![图](http://binyanbin.github.io/img/hishop-11.png)

## 部署图

![图](http://binyanbin.github.io/img/hishop-12.png)

## 标准部署方案的业务支撑能力

| 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） | 日均最大PV（11小时峰值测试）|
|-----|-----|-----|-----|
| 3000 | 4秒内 | 2000 | 约7920万 |

## 标准集群部署配置

| 硬件（设施） | 配置 | 数量 |
|-------|-------|-------|
| SLB负载均衡服务器 | 依实际流量付费 | 5 |
| 前端应用服务器 | Intel Xeon E5-2430 2.2GHz 8G RAM 64位Win 2008 R2 | 3 ~ 6 |
| 后端应用服务器 | Intel Xeon E5-2430 2.2GHz 8G RAM 64位Win 2008 R2 | 6 ~ 10 |
| CDN内容加速 | 依图片及视频等资源的容量和使用量付费 |    |	
| Redis集群 | 热点内容缓存 | 按使用收费 |
| DRDS | 按使用收费	|       |

## 标准配置性能测试数据
| 服务器 | 说明 |
|------|------|
| 测试服务器 | 一台ECS 系统:centOS 测试软件:Jmeter |
| 测试对象Himall | 标准集群阿里云部署 |

####  浏览首页
响应时间:

![图](http://binyanbin.github.io/img/hishop-13.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-14.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） |
|-----|-----|-----|-----|
| 首页 | 3000 | 0.42秒 | 3550 |

#### 浏览商品详情页
响应时间:

![图](http://binyanbin.github.io/img/hishop-15.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-16.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） |
|-----|-----|-----|-----|
| 商品浏览 | 3000 | 3.16秒 | 660 |

#### 商品搜索
响应时间:

![图](http://binyanbin.github.io/img/hishop-17.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-18.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） | 关键字数 |
|-----|-----|-----|-----|
| 商品搜索 | 3000 | 1.125秒 | 1534.1 | 100 |

#### 用户登陆
响应时间:

![图](http://binyanbin.github.io/img/hishop-19.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-20.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） |
|-----|-----|-----|-----|
| 用户登陆 | 2000 | 4.283秒 | 349 |

#### 下单
响应时间:

![图](http://binyanbin.github.io/img/hishop-21.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-22.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） |
|-----|-----|-----|-----|
| 下单 | 1000 | 4.657秒 | 189.1 |

#### 下单流程
响应时间:

![图](http://binyanbin.github.io/img/hishop-23.png)

事务处理:

![图](http://binyanbin.github.io/img/hishop-24.png)

| 测试内容 | 并发用户数（VU） | 平均响应时间（RT） | 每秒事务数（TPS） |
|-----|-----|-----|-----|
| 购买商品流程 | 1000 | 15.657秒 | 176.8 |
