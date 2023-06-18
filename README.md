<div align=center>
    <img src="./folia.png">
    <br /><br />
    <p><a href="https://github.com/PaperMC/Paper">Paper</a>的分支, 将区域化多线程添加到服务器</p>
</div>

## 关于本项目
### 概述
##### 本项目是由alazeprt fork PaperMC/Folia 的二次品，原库[PaperMC/Folia](https://github.com/PaperMC/Folia)
本项目提供了Folia中README的汉化，且提供了构建好的Folia成品，正所谓“一寸光阴一寸金，寸金难买寸光阴”，时间就是金钱，谁也不想花那么多时间在构建上，但是由于国内访问境外网站速度实在太慢，且总会进不去（我自己构建时因git无法访问Github烦了好久），更别说新手，可能连构建都不会，所以我便创建了此项目，让任何人都可以直接下载Folia服务端的成品，不需要花费过多时间用在构建上。
### 下载
#### 最新版本
进入[Releases](https://github.com/alazeprt/Folia/releases/tag/latest)界面，找到下方的Assets，其中folia-bundler-x.x.x-SNAPSHOT-mojmap.jar是带Minecraft官方服务端的Folia服务端，其中folia-paperclip-x.x.x-SNAPSHOT-mojmap.jar是不带Minecraft官方服务端的Folia服务端（启动后会自动开始下载Minecraft官方服务端），project.tar是已经applyPatches的Folia源项目。
#### 旧版本
每个旧版本都会有一个分支（例如1.19.4对应的分支为ver/1.19），最新版本的分支则是master。首先点击[Actions](https://github.com/alazeprt/Folia/actions)打开工作流界面，接着点击搜索框，搜索你所需的版本（例如1.19.4就是搜索branch:ver/1.19，搜索时前面加branch意思是指搜索指定的分支），然后选择最靠上面且是打勾的一项，点进去，向下滑找到“Artifacts”，然后点击下方的“Artifacts”下载构建好的包，下载完之后解压，解压完之后会有几个文件，其中folia-bundler-x.x.x-SNAPSHOT-mojmap.jar是带Minecraft官方服务端的Folia服务端，其中folia-paperclip-x.x.x-SNAPSHOT-mojmap.jar是不带Minecraft官方服务端的Folia服务端（启动后会自动开始下载Minecraft官方服务端）。


## 概述

Folia 将附近已加载的区块分组形成一块“独立区域”，有关Folia如何对附近的区块进行分组的详细信息，请见[PaperMC 文档](https://docs.papermc.io/folia/reference/region-logic)。
每个“独立区域”都有自己的tick循环，在常规的Minecraft Tickrate（20TPS）。这些tick循环并行地在线程池上。不再有主线，因为每个区域都有自己tick循环。

对于具有许多分散玩家的服务器，Folia将创建许多“独立区域”，并并行勾选它们的线程池（~~这里实在不知道怎么理解~~）。因此，Folia应该可以很好地扩展这样的服务器。

Folia也是一个独立的项目，Folia在未来将不会与Paper合并。 

更详细~~但抽象~~的概述: [Project overview](https://docs.papermc.io/folia/reference/overview).

## FAQ

### 哪些服务器类型可以从Folia中受益？
自然分散玩家的服务器类型，如skyblock或SMP，将从Folia中受益最大。服务器也应该有相当数量的玩家。

### Folia在什么硬件配置上运行最好？
建议CPU至少有16个核心（不是线程）运行服务器。

### 如何最好的配置Folia？
首先，建议预生成区块，以便大大减少所需的区块工作线程的数量。
以下是一个*非常粗略*的估计，使用Folia在我们的测试服务器上进行测试，该服务器的峰值玩家约为330人。因此，它并不准确，需要进一步调整——只是把它作为一个起点。

应考虑机器上可用的核心的总数。然后，为以下各项分配线程：
- 净IO：每4核心200-300玩家
- 区块系统IO线程：每3核心200-300玩家
- 区块系统工作程序（如果预先生成）：每2核心200-300玩家
- 如果不是预先生成的，就没有对区块系统工作程序的最佳猜测，因为在我们运行的测试服务器上，我们提供了16个线程，但区块生成仍然是只有300人左右。

在所有这些分配之后，系统上的剩余核心直到80%可以分配（分配的线程总数<可用cpu的80%）可以
分配给tickthreads（在全局配置下，配置项为：threaded-regions.threads）。

不应分配超过80%的核心的原因是：插件甚至服务
可能会使用额外的线程,您无法配置甚至无法预测。

此外，以上都是基于玩家数量的粗略猜测，但线程分配结果很可能不理想，你需要根据您最终看到的线程的使用情况对其进行调优。

## 插件兼容性

此服务端没有更多的主线程了，我希望存在的每一个插件都需要一定程度的修改才能在Folia中发挥作用，此外，任何类型的多线程都会在插件持有的数据中引入可能的竞争条件，因此，必然需要进行更改。

因此，请将你对插件兼容性的期望值设置为0。

## API计划

目前，有很多API依赖于主线程。我希望基本上没有任何与Paper兼容的插件与Folia兼容。然而，当前有计划添加API，使Folia插件与Paper兼容。例如，Bukkit Scheduler。Bukkit Scheduler依赖于单个主线程。Folia的RegionScheduler和Folia的EntityScheduler允许将任务安排到“下一个tick”，这些可以在常规Paper服务端上实现，除非他们调度到主线程。在这两种情况下，任务的执行都将发生在“拥有”位置或实体的线程上。这一概念适用于一般情况，因为当前的Paper服务端（单线程）可以被视为一个巨大的“区域”，涵盖了所有世界中的所有区块。

尚未决定是将此API直接添加到Paper本身还是添加到Paperlib。

### 新的规则

首先，Folia破坏了许多插件。为了帮助用户弄清楚哪些插件可以工作，只会加载作者明确标记为可以使用Folia的插件。通过将“folia-supported:true”配置项添加到插件的plugin.yml中，就可以让插件兼容Folia服务端。

另一个重要的规则是区域在 _paralle_ 中打勾，而不是 _concurrently_。他们不共享数据，也不希望共享数据，共享数据将导致数据损坏。在任何情况下都不能访问或修改在一个区域中运行的代码。仅仅因为名称中包含了多线程，并不意味着现在一切都是线程安全的。事实上，只有一件事是线程安全的，才能实现这一点。随着时间的推移，线程上下文检查的数量只会增加，即使这会带来性能损失——没有人会使用或开发一个漏洞严重的服务端，而防止和发现这些漏洞的唯一方法就是从错误的源头使错误访问不会出现。

这意味着与Folia兼容的插件需要利用RegionScheduler和EntityScheduler等API，以确保其代码在正确的线程上下文上运行。

通常，可以安全地假设一个区域拥有来自事件源的大约8个区块中的方块数据（即，玩家打破方块，可能可以访问该块周围的8个方块）。但是，这并不能保证插件应该利用即将推出的线程检查API来确保正确的行为。

线程安全的唯一保证来自于一个区域拥有特定区块中的数据这一事实——如果该区域正在运行，那么它就可以完全访问该数据。这些数据是实体/区块/poi数据，与**任何**插件数据完全无关。

正常的多线程规则适用于插件存储/访问自己的数据或其他插件的数据（事件/命令等）。在_parallel_中被调用，因为区域在_parallel_中ticking（我们不能以同步方式调用它们，因为这会引发死锁问题并阻碍性能）。没有简单的方法可以解决这个问题，这完全取决于访问的数据。有时一个并发集合（如ConcurrentHashMap）就足够了，而且通常一个不小心使用的并发集合只会引发线程问题，而这些问题几乎无法调试。

### 当前添加的API

要正确理解API添加内容，请阅读[项目概述](https://docs.papermc.io/folia/reference/overview).

- RegionScheduler、AsyncScheduler、GlobalRegionScheduler和EntityScheduler替代BukkitScheduler。实体调度器通过Entity#getScheduler检索，其余调度器可以从Bukkit/Server类检索。
- Bukkit#isOwnedByCurrentRegion测试当前ticking区域是否拥有位置/实体

### 线程的API上下文

要正确理解API添加内容，请阅读[项目概述](https://docs.papermc.io/folia/reference/overview).

一般经验法则：

1. 实体/玩家的命令在拥有实体/玩家所在的区域调用。控制台命令在全局区域上执行。

2. 涉及单个实体的事件（即玩家休息/放置方块）在区域拥有实体上调用。在拥有目标实体的区域上调用涉及实体上的操作（例如实体受攻击）的事件。

3. 不赞成使用事件的async修饰符——从区域或全局区域激发的所有事件都被认为是同步的，即使不再有主线程。

### 当前损坏的API

- 与交互有关的大多数API（例如：实体穿过传送门/玩家重生/玩家加入服务器）已损坏。
- 所有记分板API都被认为是坏的（这是全局状态，我还没有弄清楚如何正确实现）
- 世界加载/卸载
- Entity#teleport。这不再回来，请使用teleportAsync替代
- 可能有更多

### 计划添加的API

- 适当的异步事件。这将允许稍后在不同的线程上下文中完成事件。这是实现诸如设置生成位置之类的一些事件所需要的，因为在区域外访问区块数据时需要异步块加载。
- 世界加载/卸载
- 将会有更多

### 计划修改的API 

- 进行超强的线程检查。这是绝对必要的，以防止插件开发人员发送可能以完全不可识别的方式随机破坏服务器随机部分的代码。
- More to come here

### Maven 信息
* Maven Repo (用于 folia-api):
```xml
<repository>
    <id>papermc</id>
    <url>https://repo.papermc.io/repository/maven-public/</url>
</repository>
```
* 工件信息:
```xml
<dependency>
    <groupId>dev.folia</groupId>
    <artifactId>folia-api</artifactId>
    <version>1.20.1-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
 ```


## 协议
PATCHES-LICENSE文件描述了api和服务器补丁程序的许可证，位于“/patches”及其子目录，除非另有说明。