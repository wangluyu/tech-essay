# OpenRTB规范 V2.5

> 原文参考[OpenRTB API Specification Version 2.5 FINAL](https://iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf)。
>
> 作者：王璐瑜
>
> 转载请署名

## 介绍 （Introduction）

### 概览（Mission / Overview）

OpenRTB是数字媒体自动化交易的一套开放标准，用于规范自动化交易，为买方和卖方之间的对接提供开放的行业标准。这些标准涉及多个方面，包括但不限于实时出价协议，信息分类，离线配置同步等等。

### 资源（Resources）

- [OpenRTB GitHub Repository](https://github.com/openrtb/OpenRTB/)

- [Development Community Mailing List](https://groups.google.com/group/openrtb-dev)

- [User Community Mailing List](https://groups.google.com/group/openrtb-user)

### 术语（Terminology）

| 术语      | 解释                                                  |
| :-------- | ----------------------------------------------------- |
| RTB       | Real-Time-Biding 一次实时竞价交易                     |
| Exchange  | 在多个广告主之间进行拍卖的服务                        |
| Bidder    | 为了获得展示而参与实时竞价的实体                      |
| Seat      | 有广告预算的一方(广告主，媒体)                        |
| Publisher | 出售广告位的一方                                      |
| Site      | 网站或者app                                           |
| Deal      | Publisher和Site提前达成的协议，用于按特定条款购买展示 |

## OpenRTB基础（OpenRTB Basics）

下图描述了RTB的流程。

- 首先publisher通过站点或app向exchange发起一个bid request。
- exchange将请求转发给多个广告主，收到bid response后，exchange根据计费方式（一价/二价等）选出竞价成功的广告主，并给竞价成功的广告主发送win notice，给没有竞价成功的广告主发送loss notice。
- 广告主收到win notice后，如果bid response没有包含物料，则需要在收到win notice时给exchange返回物料。

![Reference Model - Request Sequence](https://pic1.zhimg.com/v2-26596b6429892a1439a08733b4a0b324_r.jpg)

### 通信（Transport）

在发起bid requests时应当使用HTTP POST，因为POST与GET相比，能携带更多的信息，而且对二进制的支持也较好。而Win notice则两者（指POST和GET）都可以使用。

对于有效的但response为空的请求应当返回HTTP状态码204。对于无效的请求（例如请求参数错误）应当返回HTTP状态码400。其他请求则应当返回HTTP状态码200。

> 建议：使用HTTP长连接可以减少请求双方的连接管理和CPU使用以提高效率。

### 安全（Security）

HTTPS不是必须的，但使用HTTPS对于双方之间的交易更加安全。因此推荐exchange和广告主可以同时支持HTTP和HTTPS。

### 数据格式（Data Format）

略

### 数据编码（Data Encoding）

略

### 在请求头传递OpenRTB版本（OpenRTB Version HTTP Header）

略

### 自定义和拓展（Customization and Extensions）

略

## bid request规范（Bid Request Specification）

当exchange或者下游向广告主发送bid request时，一个RTB交易就产生了。bid request由顶级请求对象组成，包含至少一个impression对象以及其他可选的可能包含impression上下文的对象。

### 对象模型 （Object Model）

下图是bid request的对象模型。在模型中，顶级对象（在json中为没有命名的最外层对象）被描述为BidRequest。在技术上来说，对于BidRequest的直接子对象，只有Imp是必备的，因为它是描述impression的基础，并且Imp至少要包含Banner（可以允许多种格式）、Video、Audio和Native中的一种，用于定义impression的类型。

![Bid Request object model](https://pic1.zhimg.com/v2-733438f59a1fad666aff5fbcefa6688c_r.jpg)

其他BidRequest的直接子对象提供各种信息来辅助广告主判断是否出价以及出价价格。这些信息包括用户信息(User)、设备信息(Device)、地理位置信息(Geo)、监管限制信息(Regs)、展示广告的载体信息(DistributionChannel)。

对于展示广告的载体，Site和app之间会存在差别。抽象类DistributionChannel只是一个概念而不是具体的名称，它用于代指与BidRequest相关联的Site或者App，Site和App不能同时存在。Site和App都可以通过publisher、content、producer被进一步详细地描述。

