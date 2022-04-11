# 东西要分类好

在前一章我们创建了一个单文件机器人。
但是当机器人的功能越来越复杂的时候，你的这个文件可能会有几百行。
如此一来，你一定会觉得如果能像玩游戏加 MOD（模组）那样把每一个功能都变为一个模组那该多好。

因此，我们可以按照这个思路，把机器人的每一个功能都分开成一个个模组，
每一个模组都是单独的文件，而要做到这一点就需要借助 `Saya` 了。

直至目前，社区已经有不少基于 `graia-saya` 的模组化机器人，
这种**通过模组来管理机器人各个功能**的方式会对你后面写机器人有很大帮助。
而且，某些模组甚至可以完全可以直接通过**复制粘贴文件**的形式直接装到你的机器人上。

::: warning
从本章之后所有示例都会采用 Saya 模组的形式编写及实现，并省略诸如
`import xxx`、`from xxx import xxx`、`channel = Channel.current()` 等语句。

此外，当介绍**BCC 相关概念**时，可能会使用 BCC 而非 Saya进行演示。
:::

## `Saya` 是什么

> Saya 也是 Graia Project 的一个项目  
> 她的名称取自作品 魔女之旅 中的角色 "沙耶(Saya)"  
> “愿所有人的心中都有一位活泼可爱的炭之魔女”

在开始之前，我们先来聊一聊 Python QQ 机器人中比较知名的框架 —— Nonebot。

Nonebot 就是一个比较典型的插件导入式框架（通过导入不同的插件功能来运行），
而通过 `Saya`，`Ariadne` 也可以实现类似的功能。

:::: details Saya 的各个组成部分

这部分看不懂也没关系

| 名称 | 作用 |
| :--: | :-- |
| Saya Controller<br>(控制器) | 负责控制各个模组，分配 Channel，管理模组启停，Behaviour 的注册和调用. |
| Module Channel<br>(模组容器) | 负责对模组服务，收集模组的各式信息，像 模组的名称，作者，长段的描述 之类，并负责包装模组的内容为 Cube，用以 Behaviour 对底层接口的操作. |
| Cube<br>(内容容器) | 对模组提供的内容附加一个由 Schema 实例化来的 metadata，即 "元信息"，用于给 Behaviour 进行处理. |
| Schema<br>(元信息模板) | 用于给模组提供的内容附加不同类型的元信息，给 Behaviour isinstance 处理用. |
| Behaviour<br>(行为) | 根据 Cube 及其元信息，对底层接口(例如 Broadcast，Scheduler 等)进行操作. 包括 allocate 与 uninstall 两个操作. |

说白了，每一个模组，都会有一个 `Channel`，用来保存模组相关的信息。而每一个 `Channel`，都会有一个及以上的 `Cube`。
往简单点来说呢，你可以暂时把 `Cube` 当作 BCC 的 `Listener`。

`Schema` 通过你所传递的信息，加上已经出现的 `Behaviour` 行为动作，并将这些信息传递给底层接口（例如 Broadcast）。

::::

## 如何安装 `Saya`

::: tip 注意
在写本章文档的时候，`Graia-Saya` 的版本为 `0.0.15`  
而最新版本为 <img src="https://img.shields.io/pypi/v/graia-saya?color=2970b6&amp;style=for-the-badge" alt="PyPI版本" style="vertical-align: middle">

假设你之前安装 Ariadne 时用的是以下 2 种选项中的一种，那么你可以直接跳过本小节。

- `graia-ariadne[full]`
- `graia-ariadne[graia]`
- `graia-ariadne[standard]`
:::

:::: code-group
::: code-group-item poetry

```bash
poetry add graia-saya
```

:::
::: code-group-item pip

```bash
pip install graia-saya
```

::::

## 创建一个基础框架

首先，为了降低新人的理解难度，我们直接拿出一个最小实例：

```python{13-16,18-19}
from graia.ariadne.app import Ariadne
from graia.ariadne.model import MiraiSession
from graia.saya import Saya
from graia.saya.builtins.broadcast import BroadcastBehaviour

app = Ariadne(
    MiraiSession(
        # 以下3行请按照你的 MAH 配置来填写
        host="http://localhost:8080",  # 同 MAH 的 port
        verify_key="GraiaxVerifyKey",  # 同 MAH 配置的 verifyKey
        account=1919810,  # 机器人 QQ 账号
    ),
)
saya = app.create(Saya)
saya.install_behaviours(
    app.create(BroadcastBehaviour)
)

with saya.module_context():
    saya.require("modules.ero")

app.launch_blocking()
```

我们来讲解一下**高亮部分**，感兴趣的话可以展开以下内容阅读。

:::: details 原理解析

```python
saya = app.create(Saya)
saya.install_behaviours(
    app.create(BroadcastBehaviour)
)

with saya.module_context():
    saya.require("modules.ero")
```

首先我们使用 Ariadne 创建了一个 `saya` 实例，并为其安装 `BroadcastBehaviour`。

然后在下方有一个上下文，需要注意的是，这个 `with` 上下文处理是很有必要的，
你的所有导入模组操作都必须在这个上下文处理器当中。

::: interlink
什么？你不认识 `with` 语句？那你可以去学学噢~  
要是你想去学的话，你可以看看[这篇来自 Python 官方的文档](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#with)（中文）。
:::
::::

在这段代码中，我们在上下文中使用 `saya.require` 导入了一个名为 `modules.ero` 的模组。

而事实上模组的形式可以是如下两种：
:::: code-group
::: code-group-item 单文件模组

```txt:no-line-numbers
EroEroBot
├─ main.py
├─ pyproject.toml
└─ modules
   └─ ero.py
```

:::
::: code-group-item 文件夹模组

```txt:no-line-numbers
EroEroBot
├─ main.py
├─ pyproject.toml
└─ modules
   └─ ero
      ├─ __init__.py  # 注仅调用 __init__.py 下的内容
      └─ util.py
```

::::

<h3>如何使进行动态导入</h3>

当你以后写了更多模组之后，你想必不可能一个一个的去导入，就像下面这样：

```python
with saya.module_context():
    saya.require("modules.ero1")
    saya.require("modules.ero2")
    ...
```

因此在这里介绍一种使用 Python 标准库 `pkgutil` 的比较简洁的动态导入方法，代码如下：

```python
import pkgutil
...

with saya.module_context():
    for module_info in pkgutil.iter_modules(["modules"]):
        if module_info.name.startswith("_"):
            continue
        saya.require("modules." + module_info.name)

app.launch_blocking()
```

::: tip
你不是必须用我这个用法，假设你的模组都放在一个叫 modules 的文件夹里，你可以遍历这个文件夹：

- 对于文件夹模组，直接 `saya.require(f"modules.{模组的文件夹名字}")`
- 对于 `.py` 的单文件模组，直接 `saya.require(f"modules.{模组文件名[:-3]}")`（即 `modules.` 加上去除 `.py` 后剩下的部分）
:::

<h3>把他们组合起来</h3>

前面啰里啰唆讲了一大堆，想必你一定有点不耐烦，那么就直接给你一个完整的最小实例吧：

```python
import pkgutil

from graia.ariadne.app import Ariadne
from graia.ariadne.model import MiraiSession
from graia.broadcast import Broadcast
from graia.saya.builtins.broadcast import BroadcastBehaviour

app = Ariadne(
    MiraiSession(
        # 以下3行请按照你的 MAH 配置来填写
        host="http://localhost:8080",
        verify_key="GraiaxVerifyKey",
        account=1919810,
    ),
)
saya = app.create(Saya)
saya.install_behaviours(
   app.create(BroadcastBehaviour)
)

with saya.module_context():
    for module_info in pkgutil.iter_modules(["modules"]):
        if module_info.name.startswith("_"):
            continue
        saya.require("modules." + module_info.name)

app.launch_blocking()
```

## 写个 `module`

<h3>举个栗子</h3>

我们以上文代码中提及的 `modules.ero`（即 `modules/ero.py`）为例子，
将上一章例子中的回复功能提取为一个单文件模组：

```python{9,12}
from graia.ariadne.app import Ariadne
from graia.ariadne.event.message import GroupMessage
from graia.ariadne.message.chain import MessageChain
from graia.ariadne.model import Group

from graia.saya import Channel
from graia.saya.builtins.broadcast.schema import ListenerSchema

channel = Channel.current()


@channel.use(ListenerSchema(listening_events=[GroupMessage]))
async def setu(app: Ariadne, group: Group, message: MessageChain):
    if str(message) == "你好":
         await app.sendMessage(
            group,
            MessageChain.create(f"不要说{message.asDisplay()}，来点涩图"),
         )
```

你可能会觉得这一串比之前要长不少，但是当你模组多了之后这样反而更整洁噢。

:::: details 发生了什么？

下面就来看看这个模组中各部分代码的用途吧~

<h4><code>Channel.current()</code></h4>

```python
channel = Channel.current()
```

这一行是帮你获取属于这个模组的 `channel` 实例。
事实上，你可以通过这个 `channel` 中写上一些信息，比如：

```python
channel.name("Ero")
channel.description("发送涩涩！")
channel.author("GraiaX")
```

此外，`Saya` 实例也可以通过类似的代码得到：

```python
saya = Saya.current()
```

然后你可以通过以下代码来获取来获取你导入的所有模组的信息（有可能部分信息会为 `None`）：

```python
for module, channel in saya.channels.items():
    print(
        f"module: {module}\n"
        f"name:{channel.meta['name']}\n"
        f"author:{' '.join(channel.meta['author'])}\n"
        f"description:{channel.meta['description']}"
    )
```

<h4><code>Channel.use()</code></h4>

```python
@channel.use(ListenerSchema(listening_events=[GroupMessage]))
```

你可以将其直接跟 `bcc.receiver` 画上约等号,
因为其中有一些参数的名称可能有所变化：

```txt
event -> listening_events  # 因为是events，所有要传的是有一个及以上 Event 的 list
dispatchers -> inline_dispatchers
```

::::

::: interlink
相关链接：<https://graia.readthedocs.io/extra/saya/start/>
:::