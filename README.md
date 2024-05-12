<div align=center>
    <img src="./folia.png">
    <br /><br />
    <p><a href="https://github.com/PaperMC/Paper">Paper</a>的分支, 将区域化多线程添加到服务器</p>
</div>

## 关于本项目
### 概述
##### 本项目是由alazeprt fork PaperMC/Folia 的二次品, 原库[PaperMC/Folia](https://github.com/PaperMC/Folia)
本项目提供了Folia中README的汉化, 且提供了构建好的Folia成品, 正所谓“一寸光阴一寸金, 寸金难买寸光阴”, 时间就是金钱, 谁也不想花那么多时间在构建上, 但是由于国内访问境外网站速度实在太慢, 且总会进不去(我自己构建时因git无法访问Github烦了好久), 更别说新手, 可能连构建都不会, 所以我便创建了此项目, 让任何人都可以直接下载Folia服务端的成品, 不需要花费过多时间用在构建上, 
### 下载
#### 最新版本
进入[Releases](https://github.com/alazeprt/Folia/releases/tag/latest)界面, 找到下方的Assets, 其中folia-bundler-x.x.x-SNAPSHOT-mojmap.jar是带Minecraft官方服务端的Folia服务端, 其中folia-paperclip-x.x.x-SNAPSHOT-mojmap.jar是不带Minecraft官方服务端的Folia服务端(启动后会自动开始下载Minecraft官方服务端), project.tar是已经applyPatches的Folia源项目, 
#### 旧版本
每个旧版本都会有一个分支(例如1.19.4对应的分支为ver/1.19), 最新版本的分支则是master, 首先点击[Actions](https://github.com/alazeprt/Folia/actions)打开工作流界面, 接着点击搜索框, 搜索你所需的版本(例如1.19.4就是搜索branch:ver/1.19, 搜索时前面加branch意思是指搜索指定的分支), 然后选择最靠上面且是打勾的一项, 点进去, 向下滑找到“Artifacts”, 然后点击下方的“Artifacts”下载构建好的包, 下载完之后解压, 解压完之后会有几个文件, 其中folia-bundler-x.x.x-SNAPSHOT-mojmap.jar是带Minecraft官方服务端的Folia服务端, 其中folia-paperclip-x.x.x-SNAPSHOT-mojmap.jar是不带Minecraft官方服务端的Folia服务端(启动后会自动开始下载Minecraft官方服务端), 
#### Github下载加速
在中国大陆内, Github下载速度十分缓慢(我一般只有几十kb每秒), 所以需要一些加速的方法, 我们可以在Releases中找到我们需要下载的东西(**注意: 此加速方法无法加速Artifacts**), 右键我们需要下载的东西, 点击”复制链接“按钮, 复制到的链接应是类似这样的：https://github.com/alazeprt/Folia/releases/download/latest/folia-paperclip-1.20.1-R0.1-SNAPSHOT-mojmap.jar
<br>
接着我们在复制的链接前加上https://gh.ddlc.top/
, 例如：
https://gh.ddlc.top/https://github.com/alazeprt/Folia/releases/download/latest/folia-paperclip-1.20.1-R0.1-SNAPSHOT-mojmap.jar
<br>
接着我们把这整个链接粘贴到浏览器地址栏, 按下回车, 就会开始下载了(速度一般1-3MB/s左右), 

## 概述

Folia 将附近已加载的区块分组为一个“独立区域”, 每个独立区域都有自己的刻循环, 该循环以常规的 Minecraft 刻率(20TPS)运行, 刻循环在线程池中并行执行, 现在没有主线程了, 因为每个区域实际上都有自己的“主线程”, 用于执行整个刻循环, 对于一个拥有许多分散玩家的服务器, Folia 将创建许多分散的区域, 并在一个可配置大小的线程池中并行运行它们, 因此, Folia 应该能够很好地扩展这类服务器

Folia 也是一个独立的项目, 在未来预计不会合并到 Paper 中

更详细但抽象的概述：[Overview | PaperMC Docs](https://docs.papermc.io/folia/reference/overview) 

## 常见问题

### 哪些类型的服务器可以从 Folia 中受益?

空岛或 SMP 等类型的服务器, 玩家会在不同的区域分散, 将从 Folia 中受益最多

*服务器还应该有相当数量的玩家

### Folia 在哪种硬件上运行最佳?

理想情况下, 至少有 16 个 _核心_(不是线程), 

### 如何最好地配置 Folia?

首先, 建议预先生成世界, 这样可以大大减少所需的区块系统工作线程数量

以下是一个非常粗略的估计, 基于在拥有约 330 名玩家的测试服务器上进行的测试, 因此, 这并不精确, 需要进一步调整(仅将其作为起点), 应该考虑机器上可用的核心总数, 然后, 为以下内容分配线程: 
- netty IO：每 200-300 名玩家约 4 个
- 区块系统 IO 线程：每 200-300 名玩家约 3 个
- 如果预先生成, 则区块系统工作线程每 200-300 名玩家约 2 个
- 如果没有预先生成, 则无法估计最佳猜测的区块系统工作线程数量, 因为在我们的测试服务器上, 即使分配了 16 个线程, 但在约 300 名玩家时区块生成仍然很慢, 
- GC 设置：???? 但是, GC 设置确实分配了并发线程, 这通常通过 `-XX:ConcGCThreads=n` 参数来设置, 不要将该标志与 `-XX:ParallelGCThreads=n` 混淆, 因为并行 GC 线程仅在应用程序被 GC 暂停时运行, 因此不应考虑它们, 

在分配了以上所有内容之后, 可以将系统上剩余的核心分配给刻线程(全局配置中的`threaded-regions.threads`), 直到分配到CPU核心的 80% (不应该分配超过 80% 的核心, 因为插件或甚至服务器可能会使用你无法配置或预测的额外线程)

此外, 以上所有内容都是基于玩家数量的粗略估计, 但线程分配可能并不理想, 你需要根据实际使用情况进行调整

## 插件兼容性

现在没有主线程了, 所以 _每一个_ 插件都需要 _某种_ 级别的修改才能在 Folia 中运行, 此外, 任何类型的 _多线程_ 都会在插件持有的数据中引入可能的数据竞争条件, 因此, 肯定需要做一些更改

[支持Folia的插件](https://github.com/BlockhostOfficial/folia-plugins)

## API 计划

目前, 有很多 API 依赖于主线程, 在我看来, 基本上没有兼容 Paper 的插件与 Folia 兼容

但是, 有计划添加 API, 以允许 Folia 插件与 Paper 的 API 相兼容

例如, Bukkit 调度器, Bukkit 调度器本质上是依赖于单个主线程的, Folia 的 RegionScheduler 和 Folia 的 EntityScheduler 允许将任务安排到拥有位置或实体“下一个刻”, 这些可以在常规的 Paper 上实现, 不同之处在于它们安排到主线程 - 在两种情况下, 任务的执行将发生在拥有位置或实体的线程上, 这个概念在一般情况下适用, 因为当前的 Paper(单线程)可以被视为一个包含所有世界中所有区块的巨大“区域”, 

尚未决定是否直接将此 API 添加到 Paper 本身, 还是添加到 Paperlib

### 新规则

首先, Folia 会破坏许多插件, 为了帮助用户弄清楚哪些插件有效, 只有作者明确标记为与 Folia 兼容的插件才会被加载, 
通过在插件的 plugin.yml 中放置 "folia-supported: true", 插件作者可以将其插件标记为与 Folia 兼容, 
另一个重要的规则是, 区域 _并行_ 而不是 _并发_ 刻, 它们不共享数据, 也不期望会共享数据, 共享数据将导致**数据损坏**, 
在某个区域运行的代码在任何情况下都不能访问或修改另一个区域的数据, 虽然名字中有多线程, 但并不意味着现在一切都是线程安全的

实际上, 只有 _几个_ 东西做到了线程安全以实现这一点, 随着时间推移, 线程上下文检查的数量只会增加, 即使这可能带来性能上的代价, _没有人_ 会使用或开发一个充满错误的服务器平台, 防止和发现这些错误的唯一方法是在发生错误的地方阻止这些错误继续发生

这意味着 Folia 兼容的插件需要利用像 RegionScheduler 和 EntityScheduler 这样的 API 来确保它们的代码在正确的线程上下文中运行
一般来说, 可以安全地假设一个区域拥有事件源附近大约 8 个区块的区块数据(即, 玩家破坏方块, 可能可以访问该方块周围 8 个区块), 但是并不保证能实现, 插件应该利用即将推出的线程检查 API 来确保正确的行为

线程安全的唯一保证来自于一个区域拥有某些区块的特定数据 - 如果该区域正在进行刻循环, 那么它将完全访问该数据, 这些数据特别是指实体/区块/POI 数据, 与 **任何** 插件数据完全无关

对于插件存储/访问他们自己的数据或另一个插件的数据的常规多线程规则适用于数据, 事件/命令等在 _并行_ 中被调用, 因为区域在 _并行_ 中的刻(我们不能以同步方式调用它们, 因为这会打开死锁问题并降低性能), 没有简单的出路, 这完全取决于正在访问的数据, 有时一个并发集合(如 ConcurrentHashMap)就足够了, 而草率使用并发集合常常只会 _隐藏_ 线程问题, 这使得调试变得几乎不可能

### 当前 API 所添加的内容

要正确理解 API 添加内容, 请阅读 [项目概述](https://docs.papermc.io/folia/reference/overview), 
- RegionScheduler、AsyncScheduler、GlobalRegionScheduler 和 EntityScheduler 作为 BukkitScheduler 的替代, 
  实体调度器可以通过 Entity#getScheduler 获取, 其他调度器可以从 Bukkit/Server 类中获取, 
- Bukkit#isOwnedByCurrentRegion 测试当前刻区域是否拥有位置/实体

### API 的线程上下文

要正确理解 API 添加内容, 请阅读 [项目概述](https://docs.papermc.io/folia/reference/overview), 

一般的规则：
1. 实体/玩家的命令在拥有该实体/玩家的区域上调用, 控制台命令在全局区域上执行
2. 涉及单个实体的事件(例如玩家破坏/放置方块)在拥有实体的区域上调用, 涉及对实体采取行动的事件(例如实体伤害)在拥有目标实体的区域上触发
3. 异步修饰符对于事件已被弃用 - 所有从区域或全局区域触发的事件都被认为是 _同步的_, 虽然现在没有主线程了

### 当前已损坏的 API

- 大多数关于传送门 / 重生玩家 / 玩家登录 的交互的 API 已损坏
- 所有计分板 API 被认为是损坏的(这是我还没有想出如何正确实现的的全局状态)
- 世界加载/卸载
- Entity#teleport, 这在任何情况下都不会回来, 使用 teleportAsync
- 可能还有更多
  
### 计划添加的 API

- 正确的异步事件, 这将允许事件的结果稍后在不同的线程上下文中完成, 这对于实现一些需要异步区块加载的东西(如生成位置选择)是必需的
- 世界加载/卸载
- 更多即将到来
  
### 计划变更的 API

- 在整个服务端上进行积极的线程检查, 这是绝对必要的, 以防止插件开发者发布可能会以完全无法诊断的方式随机破坏服务器随机部分的代码
- 更多即将到来
  
### Maven 信息

* Maven 仓库(用于 folia-api)：
```xml
<repository>
    <id>papermc</id>
    <url>https://repo.papermc.io/repository/maven-public/</url>
</repository>
```

* 依赖信息：
```xml
<dependency>
    <groupId>dev.folia</groupId>
    <artifactId>folia-api</artifactId>
    <version>1.20.4-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
 ```
## 许可证
PATCHES-LICENSE 文件描述了 api & server patches 的许可证, 位于 `./patches` 及其子目录中
