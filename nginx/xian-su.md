# 限速

nginx一个很有用但是经常被错误使用的特性之一就是**限速\[rate limiting\]**。它可以帮你限制在一段时间内的http请求数。

**基本限速配置**

限速配置主要依靠两个指令，`limit_req_zone` 和 `limit_req`

```text
http {
  ...
  limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

  server {
      location /login/ {
          limit_req zone=mylimit;
                    ...
      }
  }
  ...
}
```

`limit_req_zone` 指令最好是定义在http块内，以便用于多个上下文。`limit_req_zone` 定义了三个参数，下面对该指令进行详解。

```text
limit_req_zone key zone rate;
```

* **key**
  * `ngx_http_limit_req_module` 模块根据key来对请求进行限速。例子中的key为`$binary_remote_addr` \[客户端IP的二进制\]，则表示同一个ip请求速率为10r/s\[第三个参数rate\]。key可以是其他的参数，如果要对接口限速，可以使用`$uri`；如果要对某个参数进行限速，可以使用`$arg_xxx`\[xxx为参数名\]。
* **zone**
  * `zone`定义了储存key状态的区域名称以及大小。该信息储存在共享内存中，所以它可以在每个nginx进程中共享。`zone`包含两部分，区域名称以及区域内存大小，它们之间用冒号`：`连接，例子中的`zone=mylimit:10m`表示定义了一个10M字节的mylimit区域。储存16,000个ip地址的状态信息大约需要1M字节，因此mylimit区域可以储存160,000个信息。
  * 如果nginx在区域内添加新key状态时，内存空间已耗尽，则会删除最久未使用的key。如果删除后空间仍旧不足，nginx则会返回503状态码\[可修改，后面会提及\]。因此，为了防止内存空间耗尽，nginx在每添加一个新key时会最多删除两个在过去60秒内都无请求的key。
* **rate**
  * `rate`定义了每个key的最大请求速率。在🌰例子中`rate=10r/s`表示1秒内最多处理10个请求，即每100ms才会处理一个请求。因此，如果两个请求间的时差如果小于100ms，后一个请求则会返回503。

`limit_req_zone` 指令只是对限速进行了定义，实际上并没有限速的功能。因此我们需要配合`limit_req`指令一起使用。`limit_req`需要定义在location块中，该指令也有三个参数，上面的例子只列举了一个参数`zone`，用于指定使用哪个限速速率。在上面的例子中，我们对/login/进行了限速，因此同一个ip的两个请求时差不能小于100ms。

**burst配置**

但实际上，严格规定同一个key每100ms只接收一个请求是很不合理的，我们有时候需要在100ms内处理多个请求。`limit_req` 指令的第二个参数可以解决这个问题。

```text
limit_req zone=mylimit burst=10;
```

