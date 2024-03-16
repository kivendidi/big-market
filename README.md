# ✨营销抽奖系统 - By 刘仕杰✨

项目介绍：营销抽奖系统是各个互联网公司用于拉新、促活、留存、转化的重要手段，此项目针对高并发场景，搭建为各平台促销提效的营销抽奖平台，管理员可通过后台管理活动与奖品，用户可以通过抽奖的方式获取奖品，在寻常抽奖的基础上扩展了黑名单用户兜底，用户权重分析，次数解锁奖品，兜底奖品随机积分等功能。整体采用DDD架构设计，支持分布式水平扩展，单实例部署下QPS实测1000左右。

核心技术：SpringBoot、MyBatis、MySQL、Redis、Redisson、Guava、RabbitMQ。

项目尚未完结，只完成了一个简单的抽奖功能，仍在持续迭代，小伙伴可以先行扩展

前端很简单，故就没有上传了，使用了 https://100px.net/ 这个大转盘插件，vue/react/原生js都可以实现的

---

>**作者**：LuckySJ-刘仕杰 - 在线演示地址 [**www.luckylottery.site**](www.luckylottery.site)

## 💫运行配置

- 运行环境：JDK 1.8+
- Springboot：2.7.12
- mysql：8.2
- redis：5.0



## ⭐后端开发进度

- ✅抽奖策略和奖品库表设计
- ✅抽取概率区间算法实现
- ✅概率预装配，权重概率预装配
- ✅抽奖前，责任链规则校验(黑名单处理，权重处理，抽取处理)
- ✅抽奖后，配合数据库构建决策树，完成次数锁判断，库存扣减判断，兜底处理等分支流程
- ✅实现活动与用户的库表设计，设计分库分表的库表设计
- ✅实现分库分表的路由(利用一个小巧的分库分组开源组件，个人认为shardingsphere使用起来比较麻烦不灵活)
- 🔥实现活动抽奖主要流程，liushijie-20240314-events分支开发中
- 实现活动发布流程(后台服务分离开来，发布活动时通过RPC调用进行库存，概率区间等数据的预热)
- 实现用户登录，用户通过扫码从微信公众号获取验证码来进行身份验证
- 用户登录后，可点击参与活动，参与活动后为用户生成活动账户
- 实现获奖数据入库
- 实现发货功能(完成获奖人信息填写，进行奖品发货，这里有时间的话会尝试对接大语言模型平台)



## 🪐前端开发进度

前端提供了vue2 vue3两个版本，/docs/web/vue和/docs/web/vue3

vue2版本

```
npm install
npm run dev
```

vue3版本

```
yarn install
yarn run dev
```

- ✅对接后端抽奖接口实现转盘抽奖功能
- 🔥实现扫码登录界面
- 实现主界面，通过主界面公示当前所有的活动
- 用户可通过点击活动，参与活动，进入活动界面
- 在活动界面展示活动信息(抽奖，活动说明，概率声明等)
- 实现后台-活动发布
- 实现后台-数据统计

**效果图预览**

![](https://img-blog.csdnimg.cn/direct/9816127cb88f4e5aafd996c8ee32efbf.png)



## 🫧项目架构

项目整体根据DDD架构架构搭建，感兴趣的小伙伴可以看下我另外一个仓库([Spring DDD架构基础骨架](https://github.com/1321928757/spring-ddd-archetype))

### 架构分层

![](https://img-blog.csdnimg.cn/direct/e367e120dc7543d385f518fd5ff67267.png)

DDD四层架构主要是app应用层，domain领域层，trigger触发器层，infrastructure基础建设层，本项目的架构在这基础上新增了api接口标准层，type通用类型层

**✨spring-ddd-archetype-app**：**应用层**

主要负责管理整个系统的配置，项目启动。对于一些第三方Bean的声明，AOP切面都可在这层完成

**✨spring-ddd-archetype-api**：**接口标准层**

该层的目的是为了让 HTTP 接口、RPC 接口，都能在一个标准下开发，为trigger层提供标准接口，这一层是为了编码规范，也可以不要这一层在trigger层直接编写对应实现类

✨**spring-ddd-archetype-trigger**：**触发器层**

触发器层主要是定义对触发动作的监听，比如Http请求接口，RPC服务，MQ消息监听，定时任务等触发动作。

✨**spring-ddd-archetype-domain**：**领域层 **

DDD架构最核心的部分，根据不同业务划分不同的领域包，为infrastructure层提供仓储接口，实现依赖倒置

✨**spring-ddd-archetype-infrastructure**：**基础建设层**

基础建设层主要负责与底层数据库的交互，消息的生产等，如mysql和缓存。实现了领域层提供的仓储接口，为领域层提供仓储服务，依赖倒置

✨**spring-ddd-archetype-types**：**通用类型层**

基础类型层是为了让提供一些像工具类的通用类，提高代码复用



## 🐾业务流程

### 奖品概率装配流程

这里用到的抽奖算法大概是为每个奖品分配其范围，然后装配对应范围数量的元素到Map中，value即为奖品id，我们通过生成总范围内的一个随机数，再通过该随机数作为key从map中取，就实现了奖品的抽取，这种方式空间换时间，优点是速度快，缺点是数据不能太苛刻，假如总范围为1000000，那么装配到map中就很容易OOM爆内存，不过在正常的情况下还是没问题的

![](https://github.com/1321928757/static-resources/blob/main/yuque_diagram%20(5).jpg?raw=true)

### 抽奖业务流程图

![](https://github.com/1321928757/static-resources/blob/main/yuque_diagram%20(2).jpg?raw=true)

### 后续整体的流程(具体代码目前只完成了策略抽奖部分，路漫漫啊)

我们把用户每次抽奖看转为一次订单下单操作

![](https://github.com/1321928757/static-resources/blob/main/yuque_diagram%20(6).jpg?raw=true)

### 库表设计

### 具体代码分析

## 🕊作者动态

最近需要准备竞赛，先鸽一段时间，会抽空把整体项目一些相关的信息补充上来，包括抽奖业务的整体流程(还没和活动用户结合)，以及项目整体的架构分层