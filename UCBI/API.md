# 接口

<!-- TOC -->

- [请求的发起方式](#请求的发起方式)
- [响应](#响应)
- [接口列表](#接口列表)

<!-- /TOC -->

接口调用方面，非常简单，主要就是「获取登录号信息」「发送消息」这两种操作，当然有些情况下可能会有获取所有好友列表、群列表的需求，但很多服务层是无法提供的，这将作为扩展接口由 UCBI 具体实现的适配器来决定是否提供。

## 请求的发起方式

由于 UCBI 的实现需要确定当前调用需要指派给哪个开放层的适配器，因此每个调用都需要传入一个 `context`，这个 `context` 的含义和 [事件的 `context` 字段](Event.md#context-字段) 是一样的，因此比较合理的妥协是所有接口调用全部采用 POST 请求。并且，为了支持可扩展接口，需要执行的操作不应该放在 URL 路径中，而应该直接放在 POST 正文中的 `action` 字段，和 `context` 一同发送，作为一个 JSON 字符串，它可以拥有更高的自由度。而操作所需的参数，组成 JSON 对象作为 `params` 字段传入。

有了前面的说明，接口调用的过程就非常清晰了，假设一个 UCBI 的实现，将接口的地址配置为根路径 `/`，IP、端口为 `127.0.0.1:8080`，则一个「获取登录号信息」的调用的 HTTP 请求可能是这样：

```http
POST / HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/json

{
    "action": "get_login_info",
    "context": {
        "via": "coolq-http-api",
        "extra": {
            "config_id": 2
        }
    }
}
```

而一个「发送私聊消息」的调用可能是这样：

```http
POST / HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/json

{
    "action": "send_private_message",
    "context": {
        "via": "coolq-http-api",
        "user_id": "12345678",
        "extra": {
            "config_id": 2
        }
    },
    "params": {
        "message": [
            {
                "type": "text",
                "text": "你好啊～"
            },
            {
                "type": "image",
                "text": "[图片]",
                "data": {"path": "/tmp/pic.jpg"}
            }
        ]
    }
}
```

注意这里的 `context` 字段通常是直接把收到的事件的 `context` 传入，如果某些特殊使用场景下需要自行构造，具体必须的字段由 UCBI 的开放层适配器来指定。

## 响应

接口请求之后如果操作成功会返回状态码 200 或 204，否则，返回 4xx，具体根据情况而定，由 UCBI 实现确定。

响应的正文如果有，则同样使用 JSON 格式，直接包含所请求的数据，而不用包含类似 `code`、`data` 这样的成功与否的标记。

## 接口列表

下面给出接口所支持的 `action` 值和所需的参数：

| `action` 字段值 | 说明 | `params` 的内容 | 响应数据 |
| -------------- | --- | ------- | ------- |
| `get_login_info` | 获取登录号信息 | 无 | `user_id`：ID<br>`user_tid`：临时 ID<br>`user_name`：昵称／用户名 |
| `send_private_message` | 发送私聊消息 | `user_id`：目标用户 ID<br>`user_tid`：目标用户临时 ID<br>`message`：消息内容 | 无 |
| `send_group_message` | 发送群组消息 | `group_id`：目标群组 ID<br>`group_tid`：目标群组临时 ID<br>`message`：消息内容 | 无 |
| `send_discuss_message` | 发送讨论组消息 | `discuss_id`：目标讨论组 ID<br>`discuss_tid`：目标讨论组临时 ID<br>`message`：消息内容 | 无 |
| `send_message` | 发送消息 | `type`：消息类型，`private`、`group`、`discuss` 之一<br>其它所需的相关 ID<br>`message`：消息内容 | 无 |
| 其它接口 | 其它某些开放层特有的接口，以星号（`*`）开头，如 `*get_group_member_list` | 由具体接口确定 | - |

和之前的所有叙述相同，这里的 `xxx_id` 和 `xxx_tid` 两者有其一即可。另外，ID 相关的参数和 `send_message` 的 `type` 参数，如果在 `params` 中没有，就会去 `context` 中找，因此当把事件的 `context` 直接传入时，`params` 中无需再传入这些参数，而只需 `message`。

发送消息接口的 `message` 参数的格式和 [消息事件中的 `message`](Event.md#消息类型) 基本相同，其中，消息段格式的 `rich` 格式不支持发送，其它格式是否支持取决于开放层是否支持。同时，支持 `message` 直接传入字符串，将默认作为 `text` 格式发送。
