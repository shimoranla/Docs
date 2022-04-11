# 不要再戳了~

在看了前两章后，你应该已经得到了一个**每发送一次"你好"就会嚷嚷要涩图的机器人**。

不过，这样还是有点太诡异了<Curtain>不是吗？</Curtain>。
事实上，QQ 除了群消息外还有其他类型的事件，如戳一戳、消息撤回等等等等……

下面教大家如何使用戳一戳事件（`NudgeEvent`）

:::: warning
此处的“戳一戳”所指的是手机 QQ 客户端中**双击头像**的功能，而不是私聊发送戳一戳表情（对应 PC 的**窗口抖动**）

接收 NudgeEvent 需要 Mirai 使用如下 3 种登陆协议中的一种：

- ANDROID_PHONE
- IPAD
- MACOS

即使你不需要“戳一戳”这个功能，我们也**十分建议**你使用以上三种登陆协议。
因为他们是现阶段支持功能最多的协议。

::::

以下是一段示例代码（不完整，请自行插入到合适的地方，注释为与群消息事件的对比）

<CodeGroup>
<CodeGroupItem title="Python <= 3.9">

``` python
# from graia.ariadne.event.message import GroupMessage
from graia.ariadne.event.mirai import NudgeEvent


# 此处注释的意思是用法类比，不是说这里可以用 GroupMessage
# @channel.use(ListenerSchema(listening_events=[GroupMessage]))
@channel.use(ListenerSchema(listening_events=[NudgeEvent]))
async def getup(app: Ariadne, event: NudgeEvent):
    if event.context_type == "group":
        await app.sendGroupMessage(
            event.group_id,
            MessageChain.create("你不要光天化日之下在这里戳我啊")
        )
    elif event.context_type == "friend":
        await app.sendFriendMessage(
            event.friend_id,
            MessageChain.create("别戳我，好痒！")
        )
    else:
        return
```

</CodeGroupItem>
<CodeGroupItem title="Python >= 3.10">

``` python
# 本部分示例使用 Python 3.10 引入的 match...case...语法
# from graia.ariadne.event.message import GroupMessage
from graia.ariadne.event.mirai import NudgeEvent


# 此处注释的意思是用法类比，不是说这里可以用 GroupMessage
# @channel.use(ListenerSchema(listening_events=[GroupMessage]))
@channel.use(ListenerSchema(listening_events=[NudgeEvent]))
async def getup(app: Ariadne, event: NudgeEvent):
    match event.context_type:
        case "group":
            await app.sendGroupMessage(
                event.group_id,
                MessageChain.create("你不要光天化日之下在这里戳我啊")
            )
        case "friend":
            await app.sendFriendMessage(
                event.friend_id,
                MessageChain.create("别戳我，好痒！")
            )
        case _:
            return
```

</CodeGroupItem>
</CodeGroup>

::: tip

1. 对于 **NudgeEvent** 应使用 `event: NudgeEvent` 获取事件实例来获得相关信息
2. 此处之所以使用 `sendGroupMessage` 和 `sendFriendMessage` 是因为 `event.target` 的类型为 `int`，而 `sengMessage` 所需参数的类型为 `Union[MessageEvent, Group, Friend, Member]` 不包含 `int`

:::

此时运行机器人，然后在群里戳一下他，你就会得到如下结果

<ChatWindow title="Graia Framework Community">
  <ChatToast>GraiaX 👉 戳了戳 EroEroBot 的腰</ChatToast>
  <ChatMsg name="EroEroBot" avatar="/avatar/ero.webp">你不要光天化日之下在这里戳我啊</ChatMsg>
</ChatWindow>

::: tip
有关什么是 Broadcast，各种 Event 又是什么，请参阅[这里](../before/Q&A.html#_3-%E4%BB%80%E4%B9%88%E6%98%AF-broadcastcontrol)
:::

::: interlink
**相关链接:** <https://graia.readthedocs.io/basic/params/>
:::