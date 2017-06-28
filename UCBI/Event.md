# 事件

<!-- TOC -->

- [事件的分类](#事件的分类)
- [JSON 数据格式](#json-数据格式)
    - [根对象](#根对象)
    - [Context 字段](#context-字段)
    - [Data 字段](#data-字段)
        - [「消息」类型](#消息类型)
        - [「通知」类型](#通知类型)

<!-- /TOC -->

## 事件的分类

首先，将事件分为三类，分别为「消息 message」「通知 notice」「请求 request」。

「消息」类型的事件，是 UCBI 着重进行统一化的事件，顾名思义，这个类型就是用户发送的消息，这种事件在普通用户的客户端上通常表现为在聊天界面显示的气泡中的内容，可以是纯文本、图片、图文混杂，也可以是音频、视频、链接分享等富媒体形式，UCBI 通过可扩展的消息表示方式来防止部分特殊类型的消息无法表示。

「通知」类型的事件对于不同消息平台差别可能较大，UCBI 会对其中最常见的一些进行统一化，如成功添加好友、新成员入群等，这类事件在用户的客户端上通常表现为显示在聊天界面中但和消息明显区分的样式。

「请求」类型的事件表现在用户客户端上通常为聊天界面之外的其它界面，如加好友请求、加群请求等，它们和**聊天**机器人这件事本身关联不大，并且需要一种方式来唯一标记请求，不同的一、二层程序（开放层和服务层）会有非常不同的做法，因此统一化的意义不大。

## JSON 数据格式

下面，定义事件的 JSON 格式。

注：

1. 下面备注为可选的字段，是可能在某些服务层无法提供的，在实现时允许没有这些字段，也就是说上层应用在使用时需要对这些字段的存在与否进行判断。
2. 所有允许以星号开头的特殊字段或值，都是随开放层程序的不同而不同的，它们是否统一化取决于 UCBI 实现中的相应适配器，因此即使同一个星号开头的字段名，在开放层（即 `context` 中的 `via`）不同的情况下，也可能具有完全不同的含义，需谨慎使用。

### 根对象

首先最外层是一个 JSON 对象，对象中固定有以下几个字段：

| 字段名 | 数据类型 | 说明 |
| ----- | ------- | --- |
| `type` | string | 事件类型，值为 `message` 或 `notice`，对应前面的事件分类 |
| `time` | number | 事件发生的时间戳 |
| `context` | object | 事件发生的上下文，通过这项必须可以直接唯一确定回复时的发送目标，若无上下文，则 `null` |
| `data` | object | 事件的实际信息，随事件类型的不同而不同 |

### Context 字段

`context` 的内容如下：

| 字段名 | 数据类型 | 说明 |
| ----- | ------- | --- |
| `platform` | string | 聊天平台名称，如 `wechat`、`slack` |
| `via` | string | 开放层程序名称，如 `coolq-http-api`、`mojo-weixin-openwx` |
| `type` | string | 上下文类型，值为 `private`、`group`、`discuss`，分别为私聊、群组、讨论组 |
| `user_id` | string | 发送者 ID |
| `user_tid` | string | 发送者临时 ID |
| `group_id` | string | 群组 ID |
| `group_tid` | string | 群组临时 ID |
| `discuss_id` | string | 讨论组 ID |
| `discuss_tid` | string | 讨论组临时 ID |
| `extra` | - | 其它附加信息 |

由于某些服务层无法提供真实的用户 ID，而只能提供一个临时的、重启失效的 ID（如 [Mojo-Weixin](https://github.com/sjdy521/Mojo-Weixin)），所以这里也允许使用 `xxx_tid` 来取代 `xxx_id`，但无论如何，此处至少有 `xxx_id` 和 `xxx_tid` 其中之一，否则应认为无上下文，对 `context` 置 `null`。

`context` 字段的内容应当能够直接传给后面「接口调用」的消息发送接口来作为发送目标。

当 `type` 为 `group` 或 `discuss` 时，最好同时也给出 `user_id` 或 `user_tid`，来标记消息／通知的发送／触发者。

`extra` 用于标记 UCBI 具体实现可能会需要的额外信息，例如消息源的唯一标识等，从而可以在 HTTP 无状态的情况下，对回复消息的发送途径进行确定。它具体包含的内容会由 UCBI 具体实现的适配器来确定。

### Data 字段

`data` 字段中的某些内容会和 `context` 有所重复，以便于使用。另外，它的内容随事件类型的不同而不同。

#### 「消息」类型

| 字段名 | 数据类型 | 说明 | 备注 |
| ----- | ------- | --- | --- |
| `type` | string | 消息类型，值为 `private`、`group`、`discuss`，分别为私聊、群组、讨论组 | - |
| `message` | array | 通过一种可扩展的方式来表示多种形式的消息内容，后面具体给出 | - |
| `sender_id` | string | 发送者 ID | - |
| `sender_tid` | string | 发送者临时 ID | - |
| `sender_name` | string | 发送者昵称／用户名 | - |
| `sender_markname` | string | 发送者备注名 | 可选 |
| `sender` | string | 发送者显示名（当有备注名时为备注名，否则为昵称／用户名） | - |
| `group_id` | string | 群组 ID |
| `group_tid` | string | 群组临时 ID |
| `group_name` | string | 群组名称 | 可选 |
| `group_markname` | string | 群组备注名 | 可选 |
| `group` | string | 群组显示名（当有备注名时为备注名，否则为群组真实名称） | 可选 |
| `discuss_id` | string | 讨论组 ID |
| `discuss_tid` | string | 讨论组临时 ID |
| `discuss_name` | string | 讨论组名称 | 可选 |
| `discuss_markname` | string | 讨论组备注名 | 可选 |
| `discuss` | string | 讨论组显示名（当有备注名时为备注名，否则为讨论组真实名称） | 可选 |
| `sender_role` | string | 发送者身份，仅当群消息或讨论组消息时存在，值为 `member`、`admin`、`owner`、`unknown`，分别对应普通成员、管理员、群主、未知（无法获得） |
| 额外字段 | - | 对于某些开放层特有的信息，允许使用额外字段来记录，字段名应以星号（`*`）开头，如 `*receiver` | 可选 |

`message` 是一个 JSON 数组，之所以用数组类型，是因为某些服务层可以支持多种内容混杂在一条消息中，例如 QQ 允许图片、文字、表情混在一起发送，如果采用在消息中通过特殊格式来标记的形式表示，则通常需要对其它普通文字进行转义（如 [酷 Q](https://cqp.cc/) 的「CQ 码」），这会对消息的解读造成不便，因此这里使用数组来对消息进行分段表示，每一段都是一段纯文字、一张图片等。

数组中至少有一个元素。每个元素称为一个「消息段」，每个消息段是一个对象，它的内容如下：

| 字段 | 数据类型 | 说明 |
| --- | ------- | --- |
| `type` | string | 消息段类型，后面具体说明 |
| `text` | string | 消息段内容，无论什么格式，都有这一项，对于纯文本消息段，这里就是文字内容，对于其它消息段，这里是人类可读的表示，如 `[图片]`、`[语音]`、`[表情:微笑]` |
| `data` | object | 其它信息，对于非纯文本消息段，这里存放具体的细节信息，后面具体说明 |

消息段类型支持以下若干种：

| 类型 | 说明 | 对应的 `data` 内容 |
| --- | ---- | ------------------ |
| `text` | 纯文本，包括 emoji | 无 |
| `at` | @ 用户 | `user_id`：被 @ 的用户 ID<br>`user_tid`：被 @ 的用户临时 ID<br>`user_name`：被 @ 的用户昵称／用户名，可选<br>`user_markname`：被 @ 的用户备注名，可选<br>`user`：被 @ 的用户显示名，可选 |
| `image` | 图片 | `url`：网络路径<br>`path`：本地路径，和上面至少有一个<br>`data`：Base64 编码，可选<br>`media_id`：媒体 ID，用于重复发送，可选 |
| `audio` | 语音／音频 | 同上 |
| `video` | 视频 | 同上 |
| `file` | 文件 | 同上 |
| `link` | 卡片链接（不同于纯文本的链接） | `url`：链接地址<br>`title`：标题<br>`content`：内容预览<br>`image`：图片链接，可选 |
| `location` | 位置／地址 | `latitude`：纬度<br>`longitude`：经度<br>`description`：地址的文字表示，可选 |
| `contact` | 联系人名片 | `user_id`：用户 ID<br>`user_tid`：用户临时 ID<br>`user_name`：用户昵称／用户名，可选 |
| `group` | 群组名片 | `group_id`：群组 ID<br>`group_tid`：群组临时 ID<br>`group_name`：群组名称，可选 |
| `rich` | 其它富媒体，如应用分享、音乐分享等 | `url`：链接地址<br>`title`：标题 |
| 其它类型 | 其它特殊类型，以星号（`*`）开头，如 `*face` | - |

对于上面有定义的类型，消息段的 `data` 中也可以包含特殊字段，同样以星号开头。

#### 「通知」类型

| 字段名 | 数据类型 | 说明 | 备注 |
| ----- | ------- | --- | --- |
| `notice` | string | 通知名称，有固定的几个，后面具体给出 | - |
| `content` | string | 供人类阅读的通知内容 | 可选 |
| `user_id` | string | 用户 ID | - |
| `user_tid` | string | 用户临时 ID | - |
| `user_name` | string | 用户昵称／用户名 | - |
| `user_markname` | string | 用户备注名 | 可选 |
| `user` | string | 用户显示名（当有备注名时为备注名，否则为昵称／用户名） | - |
| `group_id` | string | 群组 ID |
| `group_tid` | string | 群组临时 ID |
| `group_name` | string | 群组名称 | 可选 |
| `group_markname` | string | 群组备注名 | 可选 |
| `group` | string | 群组显示名（当有备注名时为备注名，否则为群组真实名称） | 可选 |
| `discuss_id` | string | 讨论组 ID |
| `discuss_tid` | string | 讨论组临时 ID |
| `discuss_name` | string | 讨论组名称 | 可选 |
| `discuss_markname` | string | 讨论组备注名 | 可选 |
| `discuss` | string | 讨论组显示名（当有备注名时为备注名，否则为讨论组真实名称） | 可选 |
| 额外字段 | - | 对于某些开放层特有的信息，允许使用额外字段来记录，字段名应以星号（`*`）开头，如 `*operator_id` | 可选 |

`notice` 所支持的通知名称如下：

| 通知名称 | 说明 | 备注 |
| ------- | --- | --- |
| `add_contact` | 新增联系人／好友 | `data` 中包含 `user` 相关字段 |
| `lose_contact` | 失去联系人／好友 | 同上 |
| `join_group` | 加入群组 | `data` 中包含 `group` 相关字段 |
| `leave_group` | 离开群组 | 同上 |
| `add_group_member` | 群组成员增加 | `data` 中包含 `user` 和 `group` 相关字段 |
| `lose_group_member` | 群组成员离开 | 同上 |
| `join_discuss` | 加入讨论组 | `data` 中包含 `discuss` 相关字段 |
| `leave_discuss` | 离开讨论组 | 同上 |
| `add_discuss_member` | 讨论组成员增加 | `data` 中包含 `user` 和 `discuss` 相关字段 |
| `lose_discuss_member` | 讨论组成员离开 | 同上 |
| 其它通知 | 对于某些有其它类型的通知的开放层，允许使用特殊值，应以星号（`*`）开头，如 `*set_group_admin` | - |
