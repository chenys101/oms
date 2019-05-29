# OMS

## TOC

- [What is order](#what-is-order)
   - [Functionalities](#functionalities)
   - [Characteriscs](#characteriscs)
      - [variety](#variety多样性)
      - [正向、逆向和搬仓混在一起，有race condition](#正向逆向和搬仓混在一起有race-condition)
      - [模型复杂](#模型复杂)
      - [依赖多](#依赖多)
- [OFC](#ofc)
- [Core building blocks](#core-building-blocks)
- [Design](#design)

## What is order

订单作为电商系统的纽带，贯穿了电商系统的关键流程，其他模块都是围绕订单构建的。 

订单可以认为是一次交易的生命周期，订单是交易的载体，反应的是履约的内容，即：一份订单就是一份合同。

```
           User --+         +-- TMS
                  |-- OMS --|
           API ---+    |    +-- WMS
                       |
    +-------------------------------+
    |		 |       |          |
dependencies    FSM   storage      打标
                         |                      
                    +----+--------+            
                    |             |           
              lookup & search  modeling    
	                          |
				facts
				  |
	                   BI & data mining
```

### Functionalities

- FSM
- Information Collector and Passthru(model)
   - Products
      - 商品信息主要影响库存、金额和WMS生产
   - Stakeholders
   - Addresses
   - Promotions
      - 涉及风控、黑白名单等
      - 可能涉及到人工审核
   - Payment
      - 记录交易金额，也要记录过程金额
      - 例如，商品分摊的优惠金额、积分、支付金额、应付等
         - 积分互换
      - 退换货、风控、财务等会用到
   - Time
      - 全生命周期的状态节点触发时间
      - 与状态一起表示它发生的时间语义
- Interact with stock
   - 避免超卖和少卖
   - 超卖对用户体验差
   - 少卖对商家体验差
- Execution(translation or orchestrator)
   - transport to WMS
   - transport to TMS
   - 履约内容
      - 时效
      - 发票
      - 优惠
      - COD收款
      - 预约
      - 运费
- Push
   - to whom
      - 商家
      - 用户
      - 仓储人员
      - 配送人员
   - how 
      - 手机push
      - 站内信
      - callback API
      - SMS
      - wechat

### Characteriscs

#### Variety(多样性)

例如
- 地址信息，O2O订单，会多出自提点地址
- 支付信息，如果COD
- 缺货处理
   - 部分缺货
- 预售，团购，拼单，拆单，合单，集单，憋单
- 来源
   - 导入
   - API
   - 页面手工建单
   - 内部测试UAT
   - 自动生成订单
   - 无界(新)零售
      - 意味着到处都可能是入口

#### 正向、逆向和搬仓混在一起，有race condition

取消可能包括如下场景，同时包括整单取消和部分取消：
- 买家取消
- 卖家取消
- WMS取消
- TMS取消
- 风控取消
- 客服取消
- 超时未支付取消
- 换货报缺
- 仲裁取消

#### 模型复杂

- 父子单(拆单)
- 主赠关系
- 团单与子单

#### 依赖多

- 内部依赖
   - 强依赖
      - 商品
      - 主数据
      - 客户中心
      - 库存
      - 产品中心
      - 时效
         - 控制下发WMS的时机
	 - 控制下发TMS的时机
      - 预分拣
      - 台账
      - 风控
      - VMI
      - GIS
      - 优惠券
      - 账户中心
   - 弱依赖
      - 清关
      - 大件预约
      - 发票系统
      - 计费系统
      - 售后系统
      - 订单异常中心
      - 保价系统
      - TMS
      - WMS
         - 仓间调拨
- 外部依赖
   - 第三方承运商
   - 第三方WMS
   - 第三方支付

## OFC

```
用户提交订单 --- OFC --- 仓储生产
```

## Core building blocks

- 各种语义的分布式锁
   - 幂等性
- 定时任务平台
- 可靠的MQ发送机制
- FSM
   - 订单状态是交易进展的反馈，是订单流程的一个个连接点
   - 不同类型订单的状态机不同
- 统一异常中心
- 流程引擎
- 异步接单框架
- 查询
   - 读写分离
   - 按订单号查询
      - 动静分离
   - 搜索引擎

## Design

### 订单形态

- 合约单(用户下达的指令)
- 计划单(生产计划单)
   - 订单拆分
   - 订单转移
- 生产单(生产运营单)
