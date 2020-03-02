# OpenRTB规范 V2.5

> 本文参考[OpenRTB API Specification Version 2.5 FINAL](https://iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf)，面向google翻译整理而成，本人英语水平欠佳，故文中会存在语句不通、信息错误等情况，欢迎留言勘误。
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

| 术语       | 解释                                                  |
| :--------- | ----------------------------------------------------- |
| RTB        | Real-Time-Biding 一次实时竞价交易                     |
| Exchange   | 在多个广告主之间进行拍卖的服务                        |
| Bidder     | 为了获得展示而参与实时竞价的实体                      |
| Seat       | 有广告预算的一方(广告主，媒体)                        |
| Publisher  | 出售广告位的一方                                      |
| Site       | 网站或者app                                           |
| Deal       | Publisher和Site提前达成的协议，用于按特定条款购买展示 |
| Adx        | Ad Exchange， 广告交易平台                            |
| impression | 指一次展示，大概是指在某广告位展示广告的行为          |

## OpenRTB基础（OpenRTB Basics）

下图描述了RTB的流程。

- 首先publisher通过站点或app向adx发起一个bid request。
- adx将请求转发给多个广告主，收到bid response后，adx根据计费方式（一价/二价等）选出竞价成功的广告主，并给竞价成功的广告主发送win notice，给没有竞价成功的广告主发送loss notice。
- 广告主收到win notice后，如果bid response没有包含物料，则需要在收到win notice时给adx返回物料。

![Reference Model - Request Sequence](https://pic1.zhimg.com/v2-26596b6429892a1439a08733b4a0b324_r.jpg)

### 通信（Transport）

在发起bid requests时应当使用HTTP POST，因为POST与GET相比，能携带更多的信息，而且对二进制的支持也较好。而Win notice则两者（指POST和GET）都可以使用。

对于有效的但response为空的请求应当返回HTTP状态码204。对于无效的请求（例如请求参数错误）应当返回HTTP状态码400。其他请求则应当返回HTTP状态码200。

> 建议：使用HTTP长连接可以减少请求双方的连接管理和CPU使用以提高效率。

### 安全（Security）

HTTPS不是必须的，但使用HTTPS对于双方之间的交易更加安全。因此推荐adx和广告主可以同时支持HTTP和HTTPS。

### 数据格式（Data Format）

略

### 数据编码（Data Encoding）

略

### 在请求头传递OpenRTB版本（OpenRTB Version HTTP Header）

略

### 自定义和拓展（Customization and Extensions）

略

## Bid Request规范（Bid Request Specification）

RTB交易发起于adx或者下游向广告主发送bid request时。bid request由顶级请求对象组成，包含至少一个impression对象以及其他可选的可能包含impression上下文的对象。

### 对象模型 （Object Model）

下图是bid request的对象模型。在模型中，顶级对象（在json中为没有命名的最外层对象）被描述为BidRequest。在技术上来说，对于BidRequest的直接子对象，只有Imp是必备的，因为它是描述impression的基础，并且Imp至少要包含Banner（可以允许多种格式）、Video、Audio和Native中的一种，用于定义impression的类型。

![Bid Request object model](https://pic1.zhimg.com/v2-733438f59a1fad666aff5fbcefa6688c_r.jpg)

其他BidRequest的直接子对象提供各种信息来辅助广告主判断是否出价以及出价价格。这些信息包括用户信息(User)、设备信息(Device)、地理位置信息(Geo)、监管限制信息(Regs)、展示广告的载体信息(DistributionChannel)。

对于展示广告的载体，Site和app之间会存在差别。上图中的抽象类DistributionChannel只是一个概念而不是BidRequest中实际存在的对象名称，它用于代指与BidRequest相关联的Site或者App，Site和App不能同时存在。Site和App都可以通过publisher、content、producer等对象中携带的信息被进一步详细地描述。

在模型图中没有画出ext对象。ext对象没有固定的结构，并且它可以被加在任何其他对象里，用于向上游传递与交易相关但是在规范之外的信息。adx应当将这些ext信息转达给广告主。

下表列出了一些Bid Request模型中的对象和对应章节索引，可以在后续的章节里查看更加详细的解释。

| 对象       | 章节   | 描述                                     |
| ---------- | ------ | ---------------------------------------- |
| BidRequest | 3.2.1  | 顶级对象                                 |
| Source     | 3.2.2  | 略                                       |
| Regs       | 3.2.3  | 监管限制                                 |
| Imp        | 3.2.4  | 用于描述impression，每个请求至少包含一个 |
| Metric     | 3.2.5  | 关于该impression的历史指标               |
| Banner     | 3.2.6  |                                          |
| Video      | 3.2.7  |                                          |
| Audio      | 3.2.8  |                                          |
| Native     | 3.2.9  |                                          |
| Format     | 3.2.10 |                                          |
| Pmp        | 3.2.1  |                                          |
| Deal       | 3.2.12 |                                          |
| Site       | 3.2.13 |                                          |
| App        | 3.2.14 |                                          |
| Publisher  | 3.2.15 |                                          |
| Content    | 3.2.1  |                                          |
| Producer   | 3.2.1  |                                          |
| Device     | 3.2.18 |                                          |
| Geo        | 3.2.19 |                                          |
| User       | 3.2.20 |                                          |
| Data       | 3.2.21 |                                          |
| Segment    | 3.2.22 |                                          |

