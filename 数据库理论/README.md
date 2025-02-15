# 数据库理论

数据库、操作系统和编译器并称为三大系统，可以说是整个计算机软件的基石。其中数据库更靠近应用层，是很多业务的支撑。这一领域经过了几十年的发展，不断的有新的进展。任何数据库管理系统的主要工作都是可靠地存储数据并使数据可供用户使用，即当你把数据交给数据库时，它应当把数据存储起来；而后当你向数据库要数据时，它应当把数据返回给你。我们常常使用数据库作为主要数据源，帮助我们在应用程序的不同部分之间共享数据库。每次创建新应用程序时，我们都没有使用存储和检索信息的方法，而是发明了一种新的组织数据的方法，而是使用数据库。这样，我们可以专注于应用程序逻辑而不是基础架构。

# 数据库的挑战

数据库最根本的功能是能把数据存下来，所以我们从这里开始。保存数据的方法很多，最简单的方法是直接在内存中建一个数据结构，保存用户发来的数据。比如用一个数组，每当收到一条数据就向数组中追加一条记录。这个方案十分简单，能满足最基本，并且性能肯定会很好，但是除此之外却是漏洞百出，其中最大的问题是数据完全在内存中，一旦停机或者是服务重启，数据就会永久丢失。

为了解决数据丢失问题，我们可以把数据放在非易失存储介质（比如硬盘）中。改进的方案是在磁盘上创建一个文件，收到一条数据，就在文件中 Append 一行。但是还不够好，假设这块磁盘出现了坏道呢？我们可以做 RAID（Redundant Array of Independent Disks），提供单机冗余存储。如果整台机器都挂了呢？比如出现了火灾，RAID 也保不住这些数据。我们还可以将存储改用网络存储，或者是通过硬件或者软件进行存储复制。但是做复制过程中是否能保证副本之间的一致性？也就是在保证数据不丢的前提下，还要保证数据不错。保证数据不丢不错只是一项最基本的要求，还有更多令人头疼的问题等待解决：

- 能否支持跨数据中心的容灾？
- 写入速度是否够快？
- 数据保存下来后，是否方便读取？
- 保存的数据如何修改？如何支持并发的修改？
- 如何原子地修改多条记录？

# 数据库的分类

数据库管理系统可以达到不同的目的：一些主要用于临时热数据，一些用作长期的冷存储，一些允许复杂的分析查询，一些仅允许通过键访问值，一些经过优化以存储时间序列 数据，有些则有效地存储了较大的 Blob 为了理解差异和区别，我们从简短的分类和概述开始，因为这有助于我们理解进一步讨论的范围。数据库有许多分类方法，譬如内存与磁盘数据库，行列数据库等，其他的某些资料来源将 DBMS 分为三大类：

- Online transaction processing (OLTP) databases: 它们处理大量面向用户的请求和事务查询通常是预定义的并且是短暂的。

- Online analytical processing (OLAP) databases: 这些处理复杂的聚合 OLAP 数据库通常用于分析和数据仓库，并且能够处理长期运行的复杂临时查询。

- Hybrid transactional and analytical processing (HTAP): 集合了 OLAP 与 OLTP 的优点。

还有许多其他术语和分类：键值存储，关系数据库，面向文档的存储和图形数据库这些概念未在此处定义，因为假定读者具有高级知识并了解其功能因为我们在这里讨论的概念可以广泛应用，并且以一定的能力在大多数提到的商店类型中使用，所以完整的分类对于进一步的讨论不是必需的，也不是重要的。

# 数据库架构

数据库管理系统设计没有共同的蓝图每个数据库的构建都略有不同，并且组件边界很难看到和定义即使这些界限存在于纸上（例如在项目文档中），在代码中看似独立的组件也可能由于性能优化，处理边缘情况或体系结构决策而被耦合。

数据库管理系统使用客户端/服务器模型，其中数据库系统实例（节点）充当服务器角色，而应用程序实例充当客户端角色。数据库也是模块化系统，由多个部分组成：传输层接受请求，确定运行查询最有效方式的查询处理器，执行操作的执行引擎以及存储引擎。

![Architecture of a database management system](https://s2.ax1x.com/2020/02/10/1IQQ9e.png)

收到后，传输子系统将查询移交给查询处理器，由后者解析，解释和验证稍后，将执行访问控制检查，因为只有在解释查询后才能完全完成访问控制检查。解析的查询将传递给查询优化器，查询优化器首先消除查询中不必要和多余的部分，然后尝试根据内部统计信息（索引基数，近似交集大小等）和数据找到执行该查询的最有效方法位置（集群中的哪些节点保存数据以及与数据传输相关的成本）优化器既处理查询解析所需的关系操作（通常表示为依赖关系树），又处理优化（如索引排序，基数估计和选择访问方法）。

查询通常以执行计划（execution plan，或查询计划）的形式表示：必须执行一系列操作才能将其结果视为完整的。由于可以使用效率不同的不同执行计划来满足相同的查询，因此优化程序会选择最佳的可用计划。执行计划由执行引擎处理，执行引擎收集本地和远程操作的执行结果。远程执行可能涉及到集群中其他节点之间的数据读写，以及复制。

本地查询（直接来自客户端或其他节点）由存储引擎执行存储引擎具有几个专门负责的组件：

- Transaction manager：该管理器计划事务并确保它们不能使数据库处于逻辑不一致的状态。

- Lock manager：该管理器锁定正在运行的事务的数据库对象，确保并发操作不会违反物理数据完整性。

- Access methods (storage structures)：这些管理磁盘上的访问和组织数据访问方法包括堆文件和存储结构，例如 B 树或 LSM 树。

- Buffer manager：该管理器将数据页缓存在内存中。

- Recovery manager：如果发生故障，此管理器将维护操作日志并还原系统状态。

事务和锁管理器共同负责并发控制（Concurrency Control），它们在确保逻辑和物理数据完整性的同时，确保并发操作尽可能高效地执行。
