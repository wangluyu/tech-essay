# OpenRTB规范 V2.5 FINAL（中文）

> 本文参考[OpenRTB API Specification Version 2.5 FINAL](https://iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf)，面向google翻译整理而成，本人英语水平欠佳(语文也不怎么样)，无法做到信雅达，故文中会存在语句不通、信息错误等情况，欢迎在github提issue。
>
> 作者：wangluyu
>
> 商业转载请联系作者获得授权，非商业转载请注明出处

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
| *impression | 广告展示机会                                                 |

## 2 OpenRTB基础（OpenRTB Basics）

下图描述了RTB的流程。

- 首先publisher向adx发起一个ad request。
- adx收到ad request后，将请求封装何曾bid request转发给多个广告主，收到bid response后，adx根据计费方式（一价/二价等）选出竞价成功的广告主，并给竞价成功的广告主发送win notice，给没有竞价成功的广告主发送loss notice。
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

| 对象       | 章节                             | 描述                                                         |
| ---------- | -------------------------------- | ------------------------------------------------------------ |
| BidRequest | [3.2.1](#3.2.1)                  | 顶级对象<br/>*Top-level object*                              |
| Source     | [3.2.2](#3.2.2)                  | *Request source details on post-auction decisioning (e.g., header bidding).* |
| Regs       | [3.2.3](#3.2.3)                  | 监管限制 <br/>*Regulatory conditions in effect for all impressions in this bid request.* |
| Imp        | [3.2.4](#3.2.4)                  | 用于描述一个特定展示的详细信息，每个请求至少包含一个 <br/>*Container for the description of a specific impression; at least 1 per request.* |
| Metric     | [3.2.5](#3-2-5-metric-dui-xiang) | 关于该impression的历史指标 <br/>*A quantifiable often historical data point about an impression.* |
| Banner     | [3.2.6](#3-2-6-banner-dui-xiang) | banner展示(包含in-banner vedio)或video随播广告的详细信息 <br/>*Details for a banner impression (incl. in-banner video) or video companion ad.* |
| Video      | [3.2.7](#3.2.7)                  | 视频展示的详细信息<br/>*Details for a video impression.*     |
| Audio      | [3.2.8](#3.2.8)                  | 音频展示的详细信息 <br/>*Container for an audio impression.* |
| Native     | [3.2.9](#3.2.9)                  | 符合Dynamic Native Ads API的原生广告的详细信息<br/>*Container for a native impression conforming to the Dynamic Native Ads API.* |
| Format     | [3.2.10](#3.2.10)                | 符合banner展示的尺寸<br/>*An allowed size of a banner*       |
| Pmp        | [3.2.11](#3.2.11)                | 适用于该展示的PMP(private marketplace)交易 <br/>*Collection of private marketplace (PMP) deals applicable to this impression.* |
| Deal       | [3.2.12](#3.2.12)                | 买卖双方对于该展示制定的交易条款<br/>*Deal terms pertaining to this impression between a seller and buyer.* |
| Site       | [3.2.13](#3.2.13)                | 展示广告的网站信息<br/>*Details of the website calling for the impression.* |
| App        | [3.2.14](#3.2.14)                | 展示广告的APP信息<br/>*Details of the application calling for the impression* |
| Publisher  | [3.2.15](#3.2.15)                | 用于展示广告的网站orAPP的发布者，rtb中的卖方<br/>*Entity that controls the content of and distributes the site or app.* |
| Content    | [3.2.16](#3.2.16)                | *Details about the published content itself, within which the ad will be shown.* |
| Producer   | [3.2.17](#3.2.17)                | content的生产者，不一定是发布者(例如 联合发布) <br/>*Producer of the content; not necessarily the publisher (e.g., syndication).* |
| Device     | [3.2.18](#3.2.18)                | 显示广告或content的设备的详细信息<br/>*Details of the device on which the content and impressions are displayed.* |
| Geo        | [3.2.19](#3.2.19)                | 设备的位置或者用户住址的位置，取决于父对象<br/>*Location of the device or user’s home base depending on the parent object.* |
| User       | [3.2.20](#3.2.20)                | 设备的使用者，广告受众<br/>*Human user of the device; audience for advertising.* |
| Data       | [3.2.21](#3.2.21)                | 来自特定数据源的其他用户定位数据集合 <br/>*Collection of additional user targeting data from a specific data source.* |
| Segment    | [3.2.22](#3.2.22)                | 用户定位数据（例如兴趣爱好等）<br/>*Specific data point about a user from a specific data source.* |

### 3.2 对象规范 （Object Specifications）

接下来的小节对bid request中的每一个对象都作出了详细的介绍，下面几点约定适用于本章节所有内容：

- 缺失标注了"request"的属性会在技术上破坏协议。

- 一些可选的属性对于业务较为重要，因此会被标注为"recommended"

- 没有标注"require"和"recommended"....略（Unless a default value is explicitly specified, an omitted attribute is interpreted as "unknown".）

#### <span id="3.2.1">3.2.1 BidRequest对象</span> 

request顶层对象包含唯一一个出价请求和请求id。`id` 和`imp` 是必须的，其中`imp`包含至少一个impression对象。其他所属于顶层对象下的对象是非必要的，其所建立规则和限制条件适用于该请求的所有impression。

还有一些下级对象可以为潜在买家提供详细数据。例如 `site` 和 `app`对象，它们指明了广告会展示在APP还是网站上，虽然在技术上不是必要的，但强烈建议提供其中一个（site对象和app对象不能同时存在）。提供`site`对象表示广告将会在基于浏览器的网页上展示，提供`app`对象则表示广告将会在不依赖于浏览器的应用程式（俗称APP）上展现。

| 属性              | 类型               | 描述                                                         |
| ----------------- | ------------------ | ------------------------------------------------------------ |
| id                | 字符串 `require`   | 出价请求的唯一id，由adx提供<br/>*Unique ID of the bid request, provided by the exchange.* |
| [imp](#3.2.4)     | 对象数组 `require` | [imp对象列表](#3.2.4)，用于描述提供的impression，至少包含一个imp对象<br/>*Array of Imp objects ([Section 3.2.4](#3.2.4)) representing the impressions offered. At least 1 Imp object is required.* |
| [site](#3.2.13)   | 对象 `recommended` | publisher的网站的详细信息，仅适用于网站且建议提供。<br/>*Details via a Site object ([Section 3.2.13](#3.2.13)) about the publisher’s website. Only applicable and recommended for websites.* |
| [app](#3.2.14)    | 对象 `recommended` | publisher的app的详细信息，仅适用于APP且建议提供。<br/>*Details via an App object ([Section 3.2.14]()#3.2.14) about the publisher’s app (i.e., non-browser applications). Only applicable and recommended for apps.* |
| [device](#3.2.18) | 对象 `recommended` | 展示广告的设备的详细信息<br/>*Details via a Device object ([Section 3.2.18](#3.2.18)) about the user’s device to which the impression will be delivered.* |
| [user](#3.2.20)   | 对象 `recommended` | 设备使用者的详细信息；广告受众<br/>*Details via a User object ([Section 3.2.20](#3.2.20)) about the human user of the device; the advertising audience.* |
| test              | 整数 默认是0       | 用于表示该请求是否计费；0表示计费；1表示测试模式，不计费<br/>*Indicator of test mode in which auctions are not billable, where 0 = live mode, 1 = test mode.* |
| at                | 整数 默认是2       | 出价类型，1代表一价，2代表二价；自定义的出价类型可以使用大于500的数来表示。<br/>*Auction type, where 1 = First Price, 2 = Second Price Plus. Exchange-specific auction types can be defined using values greater than 500.* |
| tmax              | 整数               | 请求超时时间<br/>*Maximum time in milliseconds the exchange allows for bids to be received including Internet latency to avoid timeout. This value supersedes any a priori guidance from the exchange.* |
| wseat             | 字符串数组         | 买方(如广告主、代理商)白名单，adx和竞价者必须提前沟通好seat id对应的买方信息。wseat和bseat在同一个bidrequest中至多只能存在其中一个。如果都没有则代表对买方没有限制。<br/>*White list of buyer seats (e.g., advertisers, agencies) allowed to bid on this impression. IDs of seats and knowledge of the buyer’s customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions.* |
| bseat             | 字符串数组         | 买方(如广告主、代理商)黑名单，adx和竞价者必须提前沟通好seat id对应的买方信息。wseat和bseat在同一个bidrequest中至多只能存在其中一个。如果都没有则代表对买方没有限制。<br/>*Block list of buyer seats (e.g., advertisers, agencies) restricted from bidding on this impression. IDs of seats and knowledge of the buyer’s customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions.* |
| allimps           | 整数 默认为0       | *Flag to indicate if Exchange can verify that the impressions offered represent all of the impressions available in context (e.g., all on the web page, all video spots such as pre/mid/post roll) to support road-blocking. 0 = no or unknown, 1 = yes, the impressions offered represent all that are available.* |
| cur               | 字符串数组         | 支持的货币列表(遵循ISO 4217标准)，建议只在支持多种货币交易时使用。<br/>*Array of allowed currencies for bids on this bid request using ISO-4217 alpha codes. Recommended only if the exchange accepts multiple currencies.* |
| wlang             | 字符串数组         | 素材支持的语言种类列表(遵循ISO-639-1-alpha-2标准)。缺失则表示对语言没有特别要求，但建议买方参考`Device`或`Content`中的`language`属性。<br/>*White list of languages for creatives using ISO-639-1-alpha-2. Omission implies no specific restrictions, but buyers would be advised to consider language attribute in the Device and/or Content objects if available.* |
| bcat              | 字符串数组         | 广告类型黑名单，描述当前竞价请求屏蔽的广告类型，IAB定义的广告类型列表详见5.1章节<br/>*Blocked advertiser categories using the IAB content categories. Refer to List 5.1.* |
| badv              | 字符串数组         | 广告主黑名单，描述当前竞价请求屏蔽的广告主域名(例：ford.com)<br/>*Block list of advertisers by their domains (e.g., “ford.com”).* |
| bapp              | 字符串数组         | App包名黑名单，描述当前竞价请求屏蔽的app，iOS的包名为一串数字。<br/>*Block list of applications by their platform-specific exchangeindependent application identifiers. On Android, these should be bundle or package names (e.g., com.foo.mygame). On iOS, these are numeric IDs.* |
| source            | 对象               | [source对象](#3.2.2)，该数据段描述该广告展示机会竞价决策的细节，是由adx发起还是媒体发起等等信息。 <br/> *A Sorce object ([Section 3.2.2](#3.2.2)) that provides data about the inventory source and which entity makes the final decision.* |
| regs              | 对象               | [regs对象](#3.2.3)，指定适用该请求的行业、法律或政府条例<br/>*A Regs object ([Section 3.2.3](#3.2.3)) that specifies any industry, legal, or governmental regulations in force for this request.* |
| ext               | 对象               | 自定义携带信息<br/>*Placeholder for exchange-specific extensions to OpenRTB.* |

#### <span id="3.2.2">3.2.2 Source对象</span>

该对象描述了此次广告请求实体的性质和行为。其主要作用就是标明请求是由adx发起还是媒体发起，并且记录了该请求的参与者（转了多少手，经过了哪些adx）。其中[header bidding](https://zhuanlan.zhihu.com/p/23675270)模式，就是由媒体来发起并作出最终决策的。

| 属性   | 类型                 | 描述                                                         |
| ------ | -------------------- | ------------------------------------------------------------ |
| fd     | 整数 `recommended`   | 做出最终决策的实体(可以理解为是哪一方发起的bid request)，0代表adx，1代表媒体<br/>*Entity responsible for the final impression sale decision, where 0 = exchange, 1 = upstream source.* |
| tid    | 字符串 `recommended` | 此次出价请求参与者之间共用的交易ID<br/>*Transaction ID that must be common across all participants in this bid request (e.g., potentially multiple exchanges).* |
| pchain | 字符串 `recommended` | 以冒号分隔的payment id链条字符串，大概是这样子 96cabb5fbdde37a7:1017996<br/>*Payment ID chain string containing embedded syntax described in the TAG Payment ID Protocol v1.0.* |
| ext    | 对象                 | 自定义携带信息，通常用于记录该请求的参与者<br/>*Placeholder for exchange-specific extensions to OpenRTB.* |

#### <span id="3.2.3">3.2.3 Regs对象</span>

该对象包含了任何适用该请求的行业、法律或政府条例。`coppa`属性用于表示该请求是否受美国联邦贸易委员会针对美国地区制定的`儿童在线隐私保护法`(COPPA)约束。

| 属性  | 类型 | 描述                                                         |
| ----- | ---- | ------------------------------------------------------------ |
| coppa | 整数 | 是否受COPPA约束，0 = no， 1 = yes。更多细节参照[7.5章节](#7.5)<br/>*Flag indicating if this request is subject to the COPPA regulations established by the USA FTC, where 0 = no, 1 = yes. Refer to Section 7.5 for more information.* |
| ext   | 对象 |                                                              |

#### <span id="3.2.4"> 3.2.4 Imp对象 </span>

此对象描述了一个被拍卖的广告位或者impression。单个出价请求可以包含多个`imp`对象，这意味着支持对指定页面的所有广告位进行交易。每一个`imp`对象都包含一个id，因此可以对每个出价单独考虑。

`Banner`对象、`Video`对象、`Native`对象是`imp`的子对象，它们表明了该impression的类型。publiser可以选择其中一种类型或者根据需要将几种类型进行组合，但对该impression的出价必须符合其中一种类型。

| 属性                              | 类型                | 描述                                                         |
| --------------------------------- | ------------------- | ------------------------------------------------------------ |
| id                                | 字符串 `require`    | 在该出价请求中，每个imp对象的唯一标识符（通常从1开始递增）<br/>*A unique identifier for this impression within the context of the bid request (typically, starts with 1 and increments.* |
| [metric](#3-2-5-metric-dui-xiang) | 对象数组            | metric对象([3.2.5小节](#3-2-5-metric-dui-xiang))数组         |
| [banner](#3-2-6-banner-dui-xiang) | 对象                | 一个banner对象([3.2.6小节](#3-2-6-banner-dui-xiang))；如果该impression是banner类型，则必须提供该对象<br/>*A Banner object ([Section 3.2.6](#3-2-6-banner-dui-xiang)); required if this impression is offered as a banner ad opportunity.* |
| video                             | 对象                | 一个video对象(3.2.7小节)；如果该impression是video类型，则必须提供该对象<br/>*A Video object (Section 3.2.7); required if this impression is offered as a video ad opportunity.* |
| audio                             | 对象                | 一个audio对象(3.2.8小节)；如果该impression是audio类型，则必须提供该对象<br/>*An Audio object (Section 3.2.8); required if this impression is offered as an audio ad opportunity.* |
| native                            | 对象                | 一个native对象(3.2.9小节)；如果该impression是native类型，则必须提供该对象<br/>*A Native object (Section 3.2.9); required if this impression is offered as a native ad opportunity.* |
| pmp                               | 对象                | 一个PMP对象（3.2.11小节）；适用于该impression的私有广告交易。这里放几篇文章，帮助大家更好地理解什么是PMP。 [《PMP私有交易市场——程序化广告的新高度》](http://www.chinawebanalytics.cn/pmp-new-level-of-programmatic/) 、[《半小时读懂PMP私有广告交易市场》](http://www.chinawebanalytics.cn/what-is-pmp-in-half-an-hour/)，作者是宋星。<br/>*A Pmp object (Section 3.2.11) containing any private marketplace deals in effect for this impression.* |
| displaymanager                    | 字符串              | 广告聚合的名称，负责渲染播放广告的SDK（通常用于视频或者移动设备）。推荐app或视频的广告请求提供此属性。<br/>*Name of ad mediation partner, SDK technology, or player responsible for rendering ad (typically video or mobile). Used by some ad servers to customize ad code by partner. Recommended for video and/or apps.* |
| displaymanagerver                 | 字符串              | displaymanager的版本号。<br/>*Version of ad mediation partner, SDK technology, or player responsible for rendering ad (typically video or mobile). Used by some ad servers to customize ad code by partner. Recommended for video and/or apps.* |
| instl                             | 整数；默认为0       | 1代表该广告是插屏广告或全屏广告，0代表不是插屏广告<br/>*1 = the ad is interstitial or full screen, 0 = not interstitial.* |
| tagid                             | 字符串              | 广告位的标识符或用于发起出价请求的广告代码。提供此属性可以帮助定位解决问题，或帮助广告主进行优化。<br/>*Identifier for specific ad placement or ad tag that was used to initiate the auction. This can be useful for debugging of any issues, or for optimization by the buyer.* |
| bidfloor                          | 浮点数；默认为0     | 该impression的最低出价，也可称其为CPM<br/>*Minimum bid for this impression expressed in CPM* |
| bidfloorcur                       | 字符串；默认为"USD" | 遵循ISO 4217标准的货币类型。如果adx允许，出价的货币类型可以与该属性不同。<br/>*Currency specified using ISO-4217 alpha codes. This may be different from bid currency returned by bidder if this is allowed by the exchange.* |
| clickbrowser                      | 整数                | 表明在app中点击广告后打开的浏览器类型，其中0代表嵌入式（在app中打开），1代表原生（跳转到app外，用手机中的默认浏览器打开）。 请注意，就该属性而言，iOS 9.x设备中的Safari View Controller被视为原生浏览器。<br/>*Indicates the type of browser opened upon clicking the creative in an app, where 0 = embedded, 1 = native. Note that the Safari View Controller in iOS 9.x devices is considered a native browser for purposes of this attribute.* |
| secure                            | 整数                | 标记广告素材的url是否需要使用https协议，1是0否。如果省略，则默认使用http协议。<br/>*Flag to indicate if the impression requires secure HTTPS URL creative assets and markup, where 0 = non-secure, 1 = secure. If omitted, the secure state is unknown, but non-secure HTTP support can be assumed.* |
| iframebuster                      | 字符串数组          | 支持的iframe-busters的名称<br/>*Array of exchange-specific names of supported iframe busters.* |
| exp                               | 整数                | 该交易有效的时长（秒数）。<br/>*Advisory as to the number of seconds that may elapse between the auction and the actual impression.* |
| ext                               | 对象                | 占位符<br/>*Placeholder for exchange-specific extensions to OpenRTB.* |

#### <span id="3-2-5-metric-dui-xiang"> 3.2.5 Metric对象 </span>

`metric`对象是与该impression相关的一些指标，如平均展示率，点击率等。这些指标可以让广告主更了解该impression，以便帮助它们做出更好的决策(决定是否出价、出价价格等)。每个`metric`对象由`type`作为标识符，并包含该指标的值，推荐提供数据的来源或供应商。

|  属性  | 类型                  | 描述                                                         |
| :----: | :-------------------- | ------------------------------------------------------------ |
|  type  | 字符串；`require`     | 指标的类型，类型的名称应事先提供给bidders作参考。<br/>*Type of metric being presented using exchange curated string names which should be published to bidders a priori.* |
| value  | 浮点数；`require`     | 指标的值，范围在0.0～1.0之间。<br/>*Number representing the value of the metric. Probabilities must be in the range 0.0 – 1.0.* |
| vendor | 字符串；`recommended` | 数据的来源，供应商的名称应事先提供给bidders作参考。如果该数据为adx自身提供，则该属性推荐传递“EXCHANGE”。<br/>*Source of the value using exchange curated string names which should be published to bidders a priori. If the exchange itself is the source versus a third party, “EXCHANGE” is recommended.* |
|  ext   | 对象                  | 占位符                                                       |

#### <span id="3-2-6-banner-dui-xiang"> 3.2.6 Banner对象 </span>

`Banner`广告是impression中最常见的一种类型。尽管`banner`在其他语境下具有特定的意思，但在这里，它可以用于表示很多东西，例如静态图片、[展开式广告](https://support.google.com/adsense/answer/6005201?hl=zh-Hans)、in-banner video:横幅内嵌视频广告(详情参照3.2.7小节)。视频广告里也可以包含一组`banner`对象，用作[随播广告](https://support.google.com/google-ads/answer/6293542?hl=zh-Hans)。

如果`imp`对象包含`banner`对象，则表明该impression支持banner类型。publisher也可以在该impression里提供`video`、`native`、`audio`对象，但是，广告主对该impression的出价必须符合其中一种类型。

|   属性   | 类型                    | 描述                                                         |
| :------: | :---------------------- | :----------------------------------------------------------- |
|  format  | 对象数组；`recommended` | 一个`format`对象列表，表示允许的banner尺寸。如果没有提供该属性，则推荐使用`h`和`w`属性。<br/>*Array of format objects (Section 3.2.10) representing the banner sizes permitted. If none are specified, then use of the h and w attributes is highly recommended.* |
|    w     | 整数                    | 设备横向DIP(DP)，如果未提供`format`属性，则推荐提供该属性。<br/>*Exact width in device independent pixels (DIPS); recommended if no format objects are specified.* |
|    h     | 整数                    | 设备纵向DIP(DP)，如果未提供`format`属性，则推荐提供该属性。<br/>*Exact height in device independent pixels (DIPS); recommended if no format objects are specified.* |
|  btype   | 整数数组                | banner广告类型黑名单。具体值参考列表5.2<br/>*Blocked banner ad types. Refer to List 5.2.* |
|  battr   | 整数数组                | 素材属性黑名单。具体值参考列表5.3<br/>*Blocked creative attributes. Refer to List 5.3.* |
|   pos    | 整数                    | 广告出现在屏幕的位置。具体值参考列表5.4<br/>*Ad position on screen. Refer to List 5.4.* |
|  mimes   | 字符串数组              | 支持的媒体类型（[MIME](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)），常用的媒体类型有"application/x-shockwave-flash"、"image/jpg"、"image/gif"。<br/>*Content MIME types supported. Popular MIME types may include “application/x-shockwave-flash”, “image/jpg”, and “image/gif”.* |
| topframe | 整数                    | 用于表明banner是否在顶部框架而非iframe中，1代表在顶部框架中，0代表在iframe中。<br/>*Indicates if the banner is in the top frame as opposed to an iframe, where 0 = no, 1 = yes.* |
|  expdir  | 整数数组                | banner可展开的方向，具体值参考列表5.5。<br/>*Directions in which the banner may expand. Refer to List 5.5.* |
|   api    | 整数数组                | 支持的API框架。具体值参考列表5.6。如果未明确列出某API，则表明不支持该API。<br/>*List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported.* |
|    id    | 字符串                  | 该banner对象的唯一标识符。当该`banner`对象是用作`video`的随播广告时，则推荐提供该属性。通常从1开始递增，在同一个impression里，id必须是唯一的。<br/>*Unique identifier for this banner object. Recommended when Banner objects are used with a Video object (Section 3.2.7) to represent an array of companion ads. Values usually start at 1 and increase with each object; should be unique within an impression.* |
|   vcm    | 整数                    | 当且仅当`banner`对象是用作`video`的随播广告时，才需要提供该属性。表明随播广告的渲染模式，0表示与video一起展示（concurrent），1表示在video结束后展示（end-card）。<br/>*Relevant only for Banner objects used with a Video object (Section 3.2.7) in an array of companion ads. Indicates the companion banner rendering mode relative to the associated video, where 0 = concurrent, 1 = end-card.* |
|   ext    | 对象                    | 占位符                                                       |

#### <span id="3-2-7-video-dui-xiang"> 3.2.7 Video对象 </span>

该对象代表一个插播视频广告的impression。下表中很多字段对于最基础的视频广告交易来说是非必要的，但是提供这些字段可以帮助更加精细化地控制。OpenRTB的Video物料通常都默认遵循VAST标准，这样可以在VAST中包含一组[`Banner`对象](#3-2-6-banner-dui-xiang)用于作为随播广告。

如果`imp`对象包含`video`对象，则表明该impression支持video类型。publisher也可以在该impression里提供`banner`、`native`、`audio`对象，但是，广告主对该impression的出价必须符合其中一种类型。

| 属性           | 类型                    | 描述                                                         |
| -------------- | ----------------------- | ------------------------------------------------------------ |
| mimes          | 字符串数组；`require`   | 支持的媒体类型（[MIME](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)）（例如：“video/x-ms-wmv”、“video/mp4”）<br/>*Content MIME types supported (e.g., “video/x-ms-wmv”, “video/mp4”).* |
| minduration    | 整数；`recommended`     | 视频最小时长，单位为秒<br/>*Minimum video ad duration in seconds.* |
| maxduration    | 整数；`recommended`     | 视频最大时长，单位为秒<br/>*Maximum video ad duration in seconds.* |
| protocols      | 整数数组；`recommended` | 支持的视频协议列表。详情参考[5.8章节](#5-8-xie-yi-protocols)。至少要列出一种支持的视频协议。<br/>*Array of supported video protocols. Refer to List 5.8. At least one supported protocol must be specified in either the protocol or protocols attribute.* |
| protocol       | 整数；`recommended`     | 设备横向DIP(DP)<br/>*Width of the video player in device independent pixels (DIPS).* |
| w              | 整数；`recommended`     | 设备纵向DIP(DP)<br/>*Height of the video player in device independent pixels (DIPS).* |
| startdelay     | 整数；`recommended`     | 片头广告，片中广告或片后广告的开始播放延迟秒数。更多通用值请参考[5.2小节](#)<br/>*Indicates the start delay in seconds for pre-roll, mid-roll, or post-roll ad placements. Refer to List 5.12 for additional generic values.* |
| placement      | 整数                    | impression的放置类型，详情参考[5.9章节](#)<br/>*Placement type for the impression. Refer to List 5.9.* |
| linearity      | 整数                    | 用于规定impression是线性的还是非线性的。如果没有指定，则表明都可以。详情参考[5.7小节](#)<br/>*Indicates if the impression must be linear, nonlinear, etc. If none specified, assume all are allowed. Refer to List 5.7.* |
| skip           | 整数                    | 表明播放器是否允许跳过广告，0表示不允许，1表示允许。如果竞价者提供的素材本身是支持跳过的，则应该在bid对象中提供`attr`字段，并且`attr`的值包含16。详情参考[5.3小节](#)<br/>*Indicates if the player will allow the video to be skipped, where 0 = no, 1 = yes. If a bidder sends markup/creative that is itself skippable, the Bid object should include the attr array with an element of 16 indicating skippable video. Refer to List 5.3* |
| skipmin        | 整数；默认为0           | <br/>*Videos of total duration greater than this number of seconds can be skippable; only applicable if the ad is skippable.* |
| skipafter      | 整数；默认为0           | <br/>*Number of seconds a video must play before skipping is enabled; only applicable if the ad is skippable.* |
| sequence       | 整数                    | <br/>*If multiple ad impressions are offered in the same bid request, the sequence number will allow for the coordinated delivery of multiple creatives.* |
| battr          | 整数数组                | <br/>*Blocked creative attributes. Refer to List 5.3.*       |
| maxextended    | 整数                    | <br/>*Maximum extended ad duration if extension is allowed. If blank or 0, extension is not allowed. If -1, extension is allowed, and there is no time limit imposed. If greater than 0, then the value represents the number of seconds of extended play supported beyond the maxduration value.* |
| minbitrate     | 整数                    | <br/>*Minimum bit rate in Kbps.*                             |
| maxbitrate     | 整数                    | <br/>*Maximum bit rate in Kbps.*                             |
| boxingallowed  | 整数；默认为1           | <br/>*Indicates if letter-boxing of 4:3 content into a 16:9 window is allowed, where 0 = no, 1 = yes* |
| playbackmethod | 整数数组                | <br/>*Playback methods that may be in use. If none are specified, any method may be used. Refer to List 5.10. Only one method is typically used in practice. As a result, this array may be converted to an integer in a future version of the specification. It is strongly advised to use only the first element of this array in preparation for this change.* |
| playbackend    | 整数                    | <br/>*The event that causes playback to end. Refer to List 5.11.* |
| delivery       | 整数数组                | <br/>*Supported delivery methods (e.g., streaming, progressive). If none specified, assume all are supported. Refer to List 5.15* |
| pos            | 整数                    | <br/>*Ad position on screen. Refer to List 5.4.*             |
| companionad    | 对象数组                | <br/>*Array of Banner objects (Section 3.2.6) if companion ads are available.* |
| api            | 整数数组                | <br/>*List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported.* |
| companiontype  | 整数数组                | <br/>*Supported VAST companion ad types. Refer to List 5.14. Recommended if companion Banner objects are included via the companionad array. If one of these banners will be rendered as an end-card, this can be specified using the vcm attribute with the particular banner (Section 3.2.6).* |
| ext            | 对象                    | *Placeholder for exchange-specific extensions to OpenRTB*    |



------

To be continued

