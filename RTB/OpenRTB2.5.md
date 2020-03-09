# OpenRTB规范 V2.5 FINAL

> 本文参考[OpenRTB API Specification Version 2.5 FINAL](https://iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf)，面向google翻译整理而成，本人英语水平欠佳(语文也不怎么样)，无法做到信雅达，故文中会存在语句不通、信息错误等情况，欢迎在github提issue。
>
> 转载请署名

## 1 介绍 （Introduction）

### 概览（Mission / Overview）

OpenRTB是数字媒体自动化交易的一套开放标准，用于规范自动化交易，为买方和卖方之间的对接提供开放的行业标准。这些标准涉及多个方面，包括但不限于实时出价协议，信息分类，离线配置同步等等。

### 资源（Resources）

- [OpenRTB GitHub Repository](https://github.com/openrtb/OpenRTB/)

- [Development Community Mailing List](https://groups.google.com/group/openrtb-dev)

- [User Community Mailing List](https://groups.google.com/group/openrtb-user)

### 术语（Terminology）

注：带*的术语原文没有，是我自己添加的

| 术语        | 解释                                                         |
| :---------- | ------------------------------------------------------------ |
| RTB         | Real-Time-Biding 一次实时竞价交易<br/>*for individual impressions in real-time (i.e., while a consumer is waiting* |
| Exchange    | 在多个广告主之间进行拍卖的服务<br/>*A service that conducts an auction among bidders per impression* |
| Bidder      | 为了获得展示而参与实时竞价的实体<br/>*An entity that competes in real-time auctions to acquire impressions.* |
| Seat        | 有广告预算的一方(广告主，媒体)<br/>*An advertising entity (e.g., advertiser, agency) that wishes to obtain impressions and uses bidders to act on their behalf; a customer of a bidder and usually the owner of the advertising budget.* |
| Publisher   | 出售广告位的一方<br/>*An entity that operates one or more sites.* |
| Site        | 网站或者app<br/>*Ad supported content including web and applications unless otherwise specified.* |
| Deal        | Publisher和Site提前达成的协议，用于按特定条款购买展示<br/>*A pre-arranged agreement between a Publisher and a Seat to purchase impressions under certain terms.* |
| *Adx        | Ad Exchange， 广告交易平台                                   |
| *impression | 指一次展示，大概是指在某广告位展示广告的行为                 |

## 2 OpenRTB基础（OpenRTB Basics）

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

## 3 Bid Request规范（Bid Request Specification）

RTB交易发起于adx或者publisher向广告主发送bid request时。bid request由顶级请求对象组成，包含至少一个impression对象以及其他可选的可能包含impression上下文的对象。

### 3.1 对象模型 （Object Model）

下图是bid request的对象模型。在模型中，顶级对象（在json中为没有命名的最外层对象）被描述为BidRequest。在技术上来说，对于BidRequest的直接子对象，只有Imp是必备的，因为它是描述impression的基础，并且Imp至少要包含Banner（可以允许多种格式）、Video、Audio和Native中的一种，用于定义impression的类型。

![Bid Request object model](https://pic1.zhimg.com/v2-733438f59a1fad666aff5fbcefa6688c_r.jpg)

其他BidRequest的直接子对象提供各种信息来辅助广告主判断是否出价以及出价价格。这些信息包括用户信息(User)、设备信息(Device)、地理位置信息(Geo)、监管限制信息(Regs)、展示广告的载体信息(DistributionChannel)。

对于展示广告的载体，Site和app之间会存在差别。上图中的抽象类DistributionChannel只是一个概念而不是BidRequest中实际存在的对象名称，它用于代指与BidRequest相关联的Site或者App，Site和App不能同时存在。Site和App都可以通过publisher、content、producer等对象中携带的信息被进一步详细地描述。

在模型图中没有画出ext对象。ext对象没有固定的结构，并且它可以被加在任何其他对象里，用于向上游传递与交易相关但是在规范之外的信息。adx应当将这些ext信息转达给广告主。

下表列出了一些Bid Request模型中的对象和对应章节索引，可以在后续的章节里查看更加详细的解释。

| 对象       | 章节   | 描述                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| BidRequest | 3.2.1  | 顶级对象<br/>*Top-level object*                              |
| Source     | 3.2.2  | *Request source details on post-auction decisioning (e.g., header bidding).* |
| Regs       | 3.2.3  | 监管限制 <br/>*Regulatory conditions in effect for all impressions in this bid request.* |
| Imp        | 3.2.4  | 用于描述一个特定展示的详细信息，每个请求至少包含一个 <br/>*Container for the description of a specific impression; at least 1 per request.* |
| Metric     | 3.2.5  | 关于该impression的历史指标 <br/>*A quantifiable often historical data point about an impression.* |
| Banner     | 3.2.6  | banner展示(包含in-banner vedio)或video随播广告的详细信息 <br/>*Details for a banner impression (incl. in-banner video) or video companion ad.* |
| Video      | 3.2.7  | 视频展示的详细信息<br/>*Details for a video impression.*     |
| Audio      | 3.2.8  | 音频展示的详细信息 <br/>*Container for an audio impression.* |
| Native     | 3.2.9  | 符合Dynamic Native Ads API的原生广告的详细信息<br/>*Container for a native impression conforming to the Dynamic Native Ads API.* |
| Format     | 3.2.10 | 符合banner展示的尺寸<br/>*An allowed size of a banner*       |
| Pmp        | 3.2.1  | 适用于该展示的PMP(private marketplace)交易 <br/>*Collection of private marketplace (PMP) deals applicable to this impression.* |
| Deal       | 3.2.12 | 买卖双方对于该展示制定的交易条款<br/>*Deal terms pertaining to this impression between a seller and buyer.* |
| Site       | 3.2.13 | 展示广告的网站信息<br/>*Details of the website calling for the impression.* |
| App        | 3.2.14 | 展示广告的APP信息<br/>*Details of the application calling for the impression* |
| Publisher  | 3.2.15 | 用于展示广告的网站orAPP的发布者，rtb中的卖方<br/>*Entity that controls the content of and distributes the site or app.* |
| Content    | 3.2.1  | *Details about the published content itself, within which the ad will be shown.* |
| Producer   | 3.2.1  | content的生产者，不一定是发布者(例如 联合发布) <br/>*Producer of the content; not necessarily the publisher (e.g., syndication).* |
| Device     | 3.2.18 | 显示广告或content的设备的详细信息<br/>*Details of the device on which the content and impressions are displayed.* |
| Geo        | 3.2.19 | 设备的位置或者用户住址的位置，取决于父对象<br/>*Location of the device or user’s home base depending on the parent object.* |
| User       | 3.2.20 | 设备的使用者，广告受众<br/>*Human user of the device; audience for advertising.* |
| Data       | 3.2.21 | 来自特定数据源的其他用户定位数据集合 <br/>*Collection of additional user targeting data from a specific data source.* |
| Segment    | 3.2.22 | 用户定位数据（例如兴趣爱好等）<br/>*Specific data point about a user from a specific data source.* |

### 3.2 对象规范 （Object Specifications）

接下来的小节对bid request中的每一个对象都作出了详细的介绍，下面几点约定适用于本章节所有内容：

- 缺失标注了"request"的属性会在技术上破坏协议。

- 一些可选的属性对于业务较为重要，因此会被标注为"recommended"

- 没有标注"require"和"recommended"....略（Unless a default value is explicitly specified, an omitted attribute is interpreted as "unknown".）

#### 3.2.1 BidRequest对象

request顶层对象包含唯一一个出价请求和请求id。`id` 和`imp` 是必须的，其中`imp`包含至少一个impression对象。其他所属于顶层对象下的对象是非必要的，其所建立规则和限制条件适用于该请求的所有impression。

还有一些下级对象可以为潜在买家提供详细数据。例如 `site` 和 `app`对象，它们指明了广告会展示在APP还是网站上，虽然在技术上不是必要的，但强烈建议提供其中一个（site对象和app对象不能同时存在）。提供`site`对象表示广告将会在基于浏览器的网页上展示，提供`app`对象则表示广告将会在不依赖于浏览器的应用程式（俗称APP）上展现。

| 属性    | 类型               | 描述                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| id      | 字符串 `require`   | 出价请求的唯一id，由adx提供<br/>*Unique ID of the bid request, provided by the exchange.* |
| imp     | 对象数组 `require` | imp对象列表，用于描述提供的impression，至少包含一个imp对象<br/>*Array of Imp objects (Section 3.2.4) representing the impressions offered. At least 1 Imp object is required.* |
| site    | 对象 `recommended` | publisher的网站的详细信息，仅适用于网站且建议提供。<br/>*Details via a Site object (Section 3.2.13) about the publisher’s website. Only applicable and recommended for websites.* |
| app     | 对象 `recommended` | publisher的app的详细信息，仅适用于APP且建议提供。<br/>*Details via an App object (Section 3.2.14) about the publisher’s app (i.e., non-browser applications). Only applicable and recommended for apps.* |
| device  | 对象 `recommended` | 展示广告的设备的详细信息<br/>*Details via a Device object (Section 3.2.18) about the user’s device to which the impression will be delivered.* |
| user    | 对象 `recommended` | 设备使用者的详细信息；广告受众<br/>*Details via a User object (Section 3.2.20) about the human user of the device; the advertising audience.* |
| test    | 整数 默认是0       | 用于表示该请求是否计费；0表示计费；1表示测试模式，不计费<br/>*Indicator of test mode in which auctions are not billable, where 0 = live mode, 1 = test mode.* |
| at      | 整数 默认是2       | 出价类型，1代表一价，2代表二价；自定义的出价类型可以使用大于500的数来表示。<br/>*Auction type, where 1 = First Price, 2 = Second Price Plus. Exchange-specific auction types can be defined using values greater than 500.* |
| tmax    | 整数               | 请求超时时间<br/>*Maximum time in milliseconds the exchange allows for bids to be received including Internet latency to avoid timeout. This value supersedes any a priori guidance from the exchange.* |
| wseat   | 字符串数组         | 买方(如广告主、代理商)白名单，adx和竞价者必须提前沟通好seat id对应的买方信息。wseat和bseat在同一个bidrequest中至多只能存在其中一个。如果都没有则代表对买方没有限制。<br/>*White list of buyer seats (e.g., advertisers, agencies) allowed to bid on this impression. IDs of seats and knowledge of the buyer’s customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions.* |
| bseat   | 字符串数组         | 买方(如广告主、代理商)黑名单，adx和竞价者必须提前沟通好seat id对应的买方信息。wseat和bseat在同一个bidrequest中至多只能存在其中一个。如果都没有则代表对买方没有限制。<br/>*Block list of buyer seats (e.g., advertisers, agencies) restricted from bidding on this impression. IDs of seats and knowledge of the buyer’s customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions.* |
| allimps | 整数 默认为0       | *Flag to indicate if Exchange can verify that the impressions offered represent all of the impressions available in context (e.g., all on the web page, all video spots such as pre/mid/post roll) to support road-blocking. 0 = no or unknown, 1 = yes, the impressions offered represent all that are available.* |
| cur     | 字符串数组         | 支持的货币列表(遵循ISO 4217标准)，建议只在支持多种货币交易时使用。<br/>*Array of allowed currencies for bids on this bid request using ISO-4217 alpha codes. Recommended only if the exchange accepts multiple currencies.* |
| wlang   | 字符串数组         | 素材支持的语言种类列表(遵循ISO-639-1-alpha-2标准)。缺失则表示对语言没有特别要求，但建议买方参考`Device`或`Content`中的`language`字段。<br/>*White list of languages for creatives using ISO-639-1-alpha-2. Omission implies no specific restrictions, but buyers would be advised to consider language attribute in the Device and/or Content objects if available.* |
| bcat    | 字符串数组         | 广告类型黑名单，IAB定义的广告类型列表详见5.1章节<br/>*Blocked advertiser categories using the IAB content categories. Refer to List 5.1.* |
| badv    | 字符串数组         | 广告主域名黑名单(例：ford.com)<br/>*Block list of advertisers by their domains (e.g., “ford.com”).* |
| bapp    | 字符串数组         | App包名黑名单，iOS的包名为一串数字。<br/>*Block list of applications by their platform-specific exchangeindependent application identifiers. On Android, these should be bundle or package names (e.g., com.foo.mygame). On iOS, these are numeric IDs.* |
| source  | 对象               | source对象，提供 <br/> *A Sorce object ([Section 3.2.2](#3.2.2)) that provides data about the inventory source and which entity makes the final decision.* |
| regs    | 对象               | regs对象，指定适用该请求的行业、法律或政府条例<br/>*A Regs object (Section 3.2.3) that specifies any industry, legal, or governmental regulations in force for this request.* |
| ext     | 对象               | 自定义携带信息<br/>*Placeholder for exchange-specific extensions to OpenRTB.* |

#### <span id="3.2.2">3.2.2 Source对象</span>

