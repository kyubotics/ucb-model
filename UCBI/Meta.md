# 元事件和元接口

<!-- TOC -->

- [元事件](#元事件)
- [元接口](#元接口)

<!-- /TOC -->

## 元事件

元事件是一种特殊类型的事件，它们不是来自于用户发给机器人的消息或机器人所收到的通知，而是来自于 UCBI 程序本身。元事件和普通事件共用一个上报／轮训渠道，数据的格式也还是和普通事件相似（[根对象](Event.md#根对象)），不过没有 `context` 字段，`type` 字段值为 `meta_event`，`data` 字段随具体元事件的不同而不同，其中，固定有 `meta_event` 字段用于存放元事件名称（下面的表格中将不再给出这个字段）。

UCBI 实现应至少支持下面这些元事件：

| 元事件名称 | 说明 | `data` 中除 `meta_event` 字段的内容 |
| -------- | ---- | ------------- |
| `message_source_down` | 消息源宕机 | `platform`：聊天平台名称，如 `wechat`、`slack`<br>`via`：翻译层程序名称，如 `coolq-http-api`、`mojo-weixin-openwx`<br>`extra`：其它附加信息，用于定位具体是哪一个消息源 |
| `message_source_up` | 消息源上线 | `platform`：聊天平台名称，如 `wechat`、`slack`<br>`via`：翻译层程序名称，如 `coolq-http-api`、`mojo-weixin-openwx`<br>`extra`：其它附加信息，用于定位具体是哪一个消息源 |

第一次启动加载时，消息源的宕机和上线是否构成元事件由 UCBI 的具体实现决定。

## 元接口

和元事件一样，元接口所操作的动作和获取的数据，是和 UCBI 程序本身相关的，而非它下面的翻译层所登录的聊天平台。并且，元接口也和普通接口共用一个请求渠道，[请求的发起方式](API.md#请求的发起方式) 也和普通接口相似，不过不用传入 `context` 字段。另外，为了避免未来可能的冲突以及方便区分，所有元接口的 `action` 字段以 `^` 开头，如 `^get_available_contexts`。

UCBI 实现应至少支持下面这些元接口：

| `action` 字段值 | 说明 | 所需参数 | 响应数据 |
| -------------- | --- | ------- | ------- |
| `^get_contexts` | 获取所有定义的消息源的 context | 无 | 结果是一个 JSON 数组，每个元素的字段如下：<br>`platform`：聊天平台名称，如 `wechat`<br>`via`：翻译层程序名称，如 `coolq-http-api`<br>`extra`：其它附加信息，用于定位具体消息源<br>`available`：布尔类型，标记是否可用 |
| `^get_available_contexts` | 获取所有可用的（当前在线的）消息源的 context | 无 | 同上 |

上述 context 也即普通事件和接口调用中出现的 context，见 [事件的 `context` 字段](Event.md#context-字段)，获取到这个 context 之后，就可以在接口请求中传入，见 [请求的发起方式](API.md#请求的发起方式)。

`^get_available_contexts` 接口的一个典型使用场景是上层应用在接收到 `message_source_down` 元事件之后，获取当前可用的消息源，从而通过这些可用的消息源中的某个或若干个来通知「超级用户」。
