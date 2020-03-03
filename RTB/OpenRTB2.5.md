# OpenRTB规范 V2.5

> 本文参考[OpenRTB API Specification Version 2.5 FINAL](https://iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf)，面向google翻译整理而成，本人英语水平欠佳(语文也不怎么样)，无法做到信雅达，故文中会存在语句不通、信息错误等情况，欢迎留言勘误。
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

RTB交易发起于adx或者publisher向广告主发送bid request时。bid request由顶级请求对象组成，包含至少一个impression对象以及其他可选的可能包含impression上下文的对象。

### 对象模型 （Object Model）

下图是bid request的对象模型。在模型中，顶级对象（在json中为没有命名的最外层对象）被描述为BidRequest。在技术上来说，对于BidRequest的直接子对象，只有Imp是必备的，因为它是描述impression的基础，并且Imp至少要包含Banner（可以允许多种格式）、Video、Audio和Native中的一种，用于定义impression的类型。

![Bid Request object model](https://pic1.zhimg.com/v2-733438f59a1fad666aff5fbcefa6688c_r.jpg)

其他BidRequest的直接子对象提供各种信息来辅助广告主判断是否出价以及出价价格。这些信息包括用户信息(User)、设备信息(Device)、地理位置信息(Geo)、监管限制信息(Regs)、展示广告的载体信息(DistributionChannel)。

对于展示广告的载体，Site和app之间会存在差别。上图中的抽象类DistributionChannel只是一个概念而不是BidRequest中实际存在的对象名称，它用于代指与BidRequest相关联的Site或者App，Site和App不能同时存在。Site和App都可以通过publisher、content、producer等对象中携带的信息被进一步详细地描述。

在模型图中没有画出ext对象。ext对象没有固定的结构，并且它可以被加在任何其他对象里，用于向上游传递与交易相关但是在规范之外的信息。adx应当将这些ext信息转达给广告主。

下表列出了一些Bid Request模型中的对象和对应章节索引，可以在后续的章节里查看更加详细的解释。

| 对象       | 章节   | 描述                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| BidRequest | 3.2.1  | 顶级对象                                                     |
| Source     | 3.2.2  | Request source details on post-auction decisioning (e.g., header bidding). |
| Regs       | 3.2.3  | 监管限制                                                     |
| Imp        | 3.2.4  | 用于描述一个特定展示的详细信息，每个请求至少包含一个         |
| Metric     | 3.2.5  | 关于该impression的历史指标                                   |
| Banner     | 3.2.6  | banner展示(包含in-banner vedio)或video随播广告的详细信息     |
| Video      | 3.2.7  | 视频展示的详细信息                                           |
| Audio      | 3.2.8  | 音频展示的详细信息                                           |
| Native     | 3.2.9  | 符合Dynamic Native Ads API的原生广告的详细信息               |
| Format     | 3.2.10 | 符合banner展示的尺寸                                         |
| Pmp        | 3.2.1  | 适用于该展示的PMP(private marketplace)交易                   |
| Deal       | 3.2.12 | 买卖双方对于该展示制定的交易条款                             |
| Site       | 3.2.13 | 展示广告的网站信息                                           |
| App        | 3.2.14 | 展示广告的APP信息                                            |
| Publisher  | 3.2.15 | 用于展示广告的网站orAPP的发布者，rtb中的卖方                 |
| Content    | 3.2.1  | Details about the published content itself, within which the ad will be shown. |
| Producer   | 3.2.1  | content的生产者，不一定是发布者(例如 联合发布)               |
| Device     | 3.2.18 | 显示广告或content的设备的详细信息                            |
| Geo        | 3.2.19 | 设备的位置或者用户住址的位置，取决于父对象                   |
| User       | 3.2.20 | 设备的使用者，广告受众                                       |
| Data       | 3.2.21 | 来自特定数据源的其他用户定位数据集合                         |
| Segment    | 3.2.22 | 一个用户定位数据（例如兴趣爱好等）                           |

### 对象规范 （Object Specifications）

接下来的小节对bid request中的每一个对象都作出了详细的介绍，下面几点约定适用于本章节所有内容：

- 缺失标注了"request"的属性会在技术上破坏协议。

- 一些可选的属性对于业务较为重要，因此会被标注为"recommended"

- 没有标注"require"和"recommended"....略（Unless a default value is explicitly specified, an omitted attribute is interpreted as "unknown".）

#### BidRequest对象

request顶层对象包含唯一一个出价请求和请求id。`id` 和`imp` 是必须的，其中`imp`包含至少一个impression对象。其他所属于顶层对象下的对象是非必要的，其所建立规则和限制条件适用于该请求的所有impression。

还有一些下级对象可以为潜在买家提供详细数据。例如 `site` 和 `app`对象，它们指明了广告会展示在APP还是网站上，虽然在技术上不是必要的，但强烈建议提供其中一个（site对象和app对象不能同时存在）。提供`site`对象表示广告将会在基于浏览器的网页上展示，提供`app`对象则表示广告将会在不依赖于浏览器的应用程式（俗称APP）上展现。

| 属性    | 类型               | 描述                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| id      | 字符串 `require`   | 出价请求的唯一id，由adx提供                                  |
| imp     | 对象数组 `require` | imp对象列表，用于描述提供的impression，至少包含一个imp对象   |
| site    | 对象 `recommended` | publisher的网站的详细信息，仅适用于网站且建议提供。          |
| app     | 对象 `recommended` | publisher的app的详细信息，仅适用于APP且建议提供。            |
| device  | 对象 `recommended` | 展示广告的设备的详细信息                                     |
| user    | 对象 `recommended` | 设备使用者的详细信息；广告受众                               |
| test    | 整数 默认是0       | 用于表示该请求是否计费；0表示计费；1表示测试模式，不计费     |
| at      | 整数 默认是2       | 出价类型，1代表一价，2代表二价；自定义的出价类型可以使用大于500的数来表示。 |
| tmax    | 整数               | 请求超时时间                                                 |
| wseat   | 字符串数组         |                                                              |
| bseat   | 字符串数组         |                                                              |
| allimps | 整数 默认为0       |                                                              |
| cur     | 字符串数组         |                                                              |
| wlang   | 字符串数组         |                                                              |
| bcat    | 字符串数组         |                                                              |
| badv    | 字符串数组         |                                                              |
| bapp    | 字符串数组         |                                                              |
| source  | 对象               |                                                              |
| regs    | 对象               |                                                              |
| ext     | 对象               |                                                              |

