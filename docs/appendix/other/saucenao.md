<script setup lang="ts">
import origin from '@/chat/appendix/other/saucenao/origin.vue'
import target from '@/chat/appendix/other/saucenao/target.vue'
</script>

# 群搜图小助手

:::tsukkomi
已经不是热点了~

<!-- 如果有空可以改改放进正文 -->

:::

就在今天（2022年5月17日），一则合并消息传遍了整个 QQ 群

<origin />

看起来，这个合并消息很大程度激发了大家对于 QQ 机器人的乐趣

所以，我决定现在在这里再新加一节教你如何制作**搜涩图**的机器人

## 0. 先做一个能用的机器人

假设你并没有看过该文档的其他章节，
那请先到[目录](/before/)，
跟着我们的教程看完「从零开始的~~异世界~~涩图机器人教程」的前三章。

## 1. 申请一个 saucenao 的 apikey

根据我们对这次事件的核心机器人 [真寻bot](https://github.com/HibiKier/zhenxun_bot) 的解析，
我们可以知道，机器人的搜图功能事实上是通过调用 [Saucenao](https://www.saucenao.com) 的 api 来实现的。
所以，让我们先注册一个 apikey

:::tip
你可能很好奇为什么这里没有图  
因为是特别篇章嘛，写的有点急，嘿嘿
:::

1. 打开 [Saucenao](https://www.saucenao.com)，点击右下角的 `Account`
2. 在 `Register` 中注册你的账号
3. 注册登录成功后，点击个人页面左上角的 `api`
4. 然后在那一大串英文中出现的 `api key` 就是我们需要的东西啦

## 2. 安装 `saucenao-api`

:::code-group

```sh [pdm]
pdm add saucenao-api
```

```sh [poetry]
poetry add saucenao-api
```

```sh [pip]
pip install saucenao-api
```

:::

## 3. 在插件文件夹里创建一个新的插件并且粘贴一下代码

:::tip
以下代码参考了 [Abot-graia](https://github.com/djkcyl/ABot-Graia)

~~听我说👂👂👂谢谢你🙏🙏🙏因为有你👉👉👉温暖了四季🌈🌈🌈~~
:::

:::code-group

```python [Twilight]
from graia.ariadne.app import Ariadne
from graia.ariadne.event.message import GroupMessage
from graia.ariadne.message.chain import MessageChain
from graia.ariadne.message.element import *
from graia.ariadne.message.parser.twilight import Twilight, FullMatch, ElementMatch, ElementResult
from graia.ariadne.model import Group, Member
from graia.saya import Channel
from graia.saya.builtins.broadcast.schema import ListenerSchema
from saucenao_api import AIOSauceNao
from saucenao_api.errors import SauceNaoApiError

channel = Channel.current()

channel.name("Saucenao")
channel.description("以图搜图")
channel.author("I_love_study")

apikey = "xxx" # 请输你自己的，谢谢


@channel.use(
    ListenerSchema(
        listening_events=[GroupMessage],
        inline_dispatchers=[
            Twilight(
                FullMatch("以图搜图"),
                FullMatch("\n", optional=True),
                "img" @ ElementMatch(Image, optional=True),
            ),
        ],
    )
)
async def saucenao(app: Ariadne, group: Group, member: Member, img: ElementResult, source: Source):
    await app.send_group_message(group, MessageChain("正在搜索，请稍后"), quote=source.id)
    async with AIOSauceNao(apikey, numres=3) as snao:
        try:
            results = await snao.from_url(img.result.url)
        except SauceNaoApiError as e:
            await app.send_message(group, MessageChain("搜索失败desu"))
            return

    fwd_nodeList = []
    for results in results.results:
        if len(results.urls) == 0:
            continue
        urls = "\n".join(results.urls)
        fwd_nodeList.append(
            ForwardNode(
                target=app.account,
                senderName="爷",
                time=datetime.now(),
                message=MessageChain(
                    f"相似度：{results.similarity}%\n标题：{results.title}\n节点名：{results.index_name}\n链接：{urls}"
                )))

    if len(fwd_nodeList) == 0:
        await app.send_message(group, MessageChain("未找到有价值的数据"), quote=source.id)
    else:
        await app.send_message(group, MessageChain(Forward(nodeList=fwd_nodeList)))
```

```python [Alconna]
from graia.ariadne.app import Ariadne
from graia.ariadne.message.element import *
from graia.ariadne.model import Group
from graia.saya import Channel
from arclet.alconna import Args, CommandMeta
from arclet.alconna.graia import Alconna, alcommand, ImgOrUrl, Match
from saucenao_api import AIOSauceNao
from saucenao_api.errors import SauceNaoApiError

channel = Channel.current()

channel.name("Saucenao")
channel.description("以图搜图")
channel.author("RF-Tar-Railt")

apikey = "xxx" # 请输你自己的，谢谢


search = Alconna(
    ".搜图",
    Args["content", ImgOrUrl],
    meta=CommandMeta(
      "以图搜图，搜图结果会自动发送给你",
      usage="你既可以传入图片, 也可以传入图片链接",
      example=".搜图 [图片]"
    )
)


@alcommand(search, private=False)
async def saucenao(app: Ariadne, group: Group, source: Source, content: Match[str]):
    if not content.available:
        return await app.send_group_message(group, MessageChain("啊嘞，你传入了个啥子东西"), quote=source.id)
    await app.send_group_message(group, MessageChain("正在搜索，请稍后"), quote=source.id)
    async with AIOSauceNao(apikey, numres=3) as snao:
        try:
            results = await snao.from_url(content.result)
        except SauceNaoApiError as e:
            await app.send_message(group, MessageChain("搜索失败desu"))
            return

    fwd_nodeList = []
    for results in results.results:
        if len(results.urls) == 0:
            continue
        urls = "\n".join(results.urls)
        fwd_nodeList.append(
            ForwardNode(
                target=app.account,
                senderName="爷",
                time=datetime.now(),
                message=MessageChain(
                    f"相似度：{results.similarity}%\n标题：{results.title}\n节点名：{results.index_name}\n链接：{urls}"
                )))

    if len(fwd_nodeList) == 0:
        await app.send_message(group, MessageChain("未找到有价值的数据"), quote=source.id)
    else:
        await app.send_message(group, MessageChain(Forward(nodeList=fwd_nodeList)))
```

:::

这样，你的搜图机器人就做好力

<target />

## 4. 背后原理

假设你真的很想知道这些东西背后的原理，你可以直接参考以下章节

- [好大的奶](/guide/forward_message.md) —— 合并消息的构建与解析
- [来点网络上的涩图](/guide/image_from_internet.md) —— `aiohttp` 的超简单运用
- [Twilight](/guide/message_parser/twilight.md) —— `Kanata` 的精神续作
- [Alconna](/guide/message_parser/alconna.md) —— 外 星 来 客
