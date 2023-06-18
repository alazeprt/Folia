<div align=center>
    <img src="./folia.png">
    <br /><br />
    <p><a href="https://github.com/PaperMC/Paper">Paper</a>的分支, 将区域化多线程添加到服务器</p>
</div>

## 关于本项目
##### 本项目是由alazeprt fork PaperMC/Folia 的二次品，原库[PaperMC/Folia](https://github.com/PaperMC/Folia)
本项目提供了Folia中README的汉化，且提供了构建好的Folia成品，正所谓“一寸光阴一寸金，寸金难买寸光阴”，时间就是金钱，谁也不想花那么多时间在构建上，但是由于国内访问境外网站速度实在太慢，且总会进不去（我自己构建时因git无法访问Github烦了好久），更别说新手，可能连构建都不会，所以我便创建了此项目，让任何人都可以直接下载Folia服务端的成品，不需要花费过多时间用在构建上。

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

### The new rules

First, Folia breaks many plugins. To aid users in figuring out which
plugins work, only plugins that have been explicitly marked by the
author(s) to work with Folia will be loaded. By placing
"folia-supported: true" into the plugin's plugin.yml, plugin authors
can mark their plugin as compatible with regionised multithreading.

The other important rule is that the regions tick in _parallel_, and not 
_concurrently_. They do not share data, they do not expect to share data,
and sharing of data _will_ cause data corruption. 
Code that is running in one region under no circumstance can 
be accessing or modifying data that is in another region. Just 
because multithreading is in the name, it doesn't mean that everything 
is now thread-safe. In fact, there are only a _few_ things that were 
made thread-safe to make this happen. As time goes on, the number 
of thread context checks will only grow, even _if_ it comes at a 
performance penalty - _nobody_ is going to use or develop for a 
server platform that is buggy as hell, and the only way to 
prevent and find these bugs is to make bad accesses fail _hard_ at the 
source of the bad access.

This means that Folia compatible plugins need to take advantage of 
API like the RegionScheduler and the EntityScheduler to ensure 
their code is running on the correct thread context.

In general, it is safe to assume that a region owns chunk data
in an approximate 8 chunks from the source of an event (i.e. player
breaks block, can probably access 8 chunks around that block). But,
this is not guaranteed - plugins should take advantage of upcoming
thread-check API to ensure correct behavior.

The only guarantee of thread-safety comes from the fact that a
single region owns data in certain chunks - and if that region is
ticking, then it has full access to that data. This data is 
specifically entity/chunk/poi data, and is entirely unrelated
to **ANY** plugin data.

Normal multithreading rules apply to data that plugins store/access
their own data or another plugin's - events/commands/etc. are called 
in _parallel_ because regions are ticking in _parallel_ (we CANNOT 
call them in a synchronous fashion, as this opens up deadlock issues 
and would handicap performance). There are no easy ways out of this, 
it depends solely on what data is being accessed. Sometimes a 
concurrent collection (like ConcurrentHashMap) is enough, and often a 
concurrent collection used carelessly will only _hide_ threading 
issues, which then become near impossible to debug.

### Current API additions

To properly understand API additions, please read
[Project overview](https://docs.papermc.io/folia/reference/overview).

- RegionScheduler, AsyncScheduler, GlobalRegionScheduler, and EntityScheduler 
  acting as a replacement for  the BukkitScheduler.
  The entity scheduler is retrieved via Entity#getScheduler, and the
  rest of the schedulers can be retrieved from the Bukkit/Server classes.
- Bukkit#isOwnedByCurrentRegion to test if the current ticking region
  owns positions/entities

### Thread contexts for API

To properly understand API additions, please read
[Project overview](https://docs.papermc.io/folia/reference/overview).

General rules of thumb:

1. Commands for entities/players are called on the region which owns
the entity/player. Console commands are executed on the global region.

2. Events involving a single entity (i.e player breaks/places block) are
called on the region owning entity. Events involving actions on an entity
(such as entity damage) are invoked on the region owning the target entity.

3. The async modifier for events is deprecated - all events
fired from regions or the global region are considered _synchronous_, 
even though there is no main thread anymore. 

### Current broken API

- Most API that interacts with portals / respawning players / some
  player login API is broken.
- ALL scoreboard API is considered broken (this is global state that
  I've not figured out how to properly implement yet)
- World loading/unloading
- Entity#teleport. This will NEVER UNDER ANY CIRCUMSTANCE come back, 
  use teleportAsync
- Could be more

### Planned API additions

- Proper asynchronous events. This would allow the result of an event
  to be completed later, on a different thread context. This is required
  to implement some things like spawn position select, as asynchronous
  chunk loads are required when accessing chunk data out-of-region.
- World loading/unloading
- More to come here

### Planned API changes

- Super aggressive thread checks across the board. This is absolutely
  required to prevent plugin devs from shipping code that may randomly
  break random parts of the server in entirely _undiagnosable_ manners.
- More to come here

### Maven information
* Maven Repo (for folia-api):
```xml
<repository>
    <id>papermc</id>
    <url>https://repo.papermc.io/repository/maven-public/</url>
</repository>
```
* Artifact Information:
```xml
<dependency>
    <groupId>dev.folia</groupId>
    <artifactId>folia-api</artifactId>
    <version>1.20.1-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
 ```


## License
The PATCHES-LICENSE file describes the license for api & server patches,
found in `./patches` and its subdirectories except when noted otherwise.

The fork is based off of PaperMC's fork example found [here](https://github.com/PaperMC/paperweight-examples).
As such, it contains modifications to it in this project, please see the repository for license information
of modified files.
