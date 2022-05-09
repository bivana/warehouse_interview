在《数据仓库工具箱》一书中，提到了数据仓库设计中的十大陷阱。在着手进行数据仓库项目之前，可以先了解这10个常见陷阱，避免在工作中的弯路。

### 陷阱10
**过于迷恋技术和数据，而忘了重点是业务需求和目标**

数仓归根结底是要解决业务问题的，对于技术人员，各种高大上的数据架构和层出不穷的新技术通常会比去了解用户需求更具有吸引力。 但是，世界上不存在完美的技术架构，只要是能够满足当下及未来可见的业务需求即可，合适才是最好的。应当把时间投入到理解和梳理业务上，这样才能构建出相对合理的数据模型，从而提高模型的复用性，及时响应业务需求。
### 陷阱9
**没有或找不到一个有影响力的、精通业务、明白事理的高级管理人员作为数仓建设的发起人**

说到这点，内心深有体会。之前入职的一家公司，规模也不小，在C轮拿到了阿里几亿的战略投资。我入职后负责从0到1的搭建，心痛的是，全公司竟然找不到一个精通业务还动数据的管理人员，在我多次邮件中申请，才又招了一位高级管理人员。可是毕竟人刚上岗，对业务又不是很熟悉，各个部门都不是很配合，可想而知数仓建设的推进多么痛苦......

数仓建设是多部门合作的结果，只有这样才能够真正的实现数据赋能业务。所以没有高层的支持和重视，数仓的建设将会很难推进。缺乏远见，热情，支持以及公司的资金投资，注定会走向失败。

### 陷阱8
**将项目处理为一个巨大的持续多年的项目，而不是追求更容易管理的、虽然仍然具有挑战性的迭代开发工作**

这是一个经常出现的陷阱，试图建设一个庞大的，无所不包的系统，通常是不可取的。似乎只要建设一个“巨型无比“的系统就可以完成任何工作，解决任何问题一样，其实结果往往会适得其反。更糟的是，管理这些项目的人往往没有与业务进行足够详细的协商，从而开发有用的产品。一言以蔽之，挂历上的模特，中看不中用。

### 陷阱7
**分配大量的精力去构建规范化数据结构，在最终呈现数据之前，用尽所有预算**

这个陷阱不像其他陷阱一样重要，当然公司有资金，可以随便玩。在Kimball方法论中，对维度模型进行更改所带来的业务风险要比更改源事务数据库小得多，所以应该留出足够的资源来构建它们，但是很少有中小型企业在资源上进行投资以创建完全一致的事实和维度表，更不用说OLAP数据立方体了，所以再多的理论也解决不了实际的问题，先跑起来最重要，不管姿势是否优雅。

### 陷阱6
**将主要精力投入到后端操作型性能和易开发性，而没有重点考虑前端查询的性能和易用性**

为用户提供易于阅读的数据展示形式并具有良好的查询性能是优先考虑。

目前我们正在着手做用户路径查询的功能模块，由于前端设计sql过于复杂，性能一直在优化，直到最近上了kudu，体验感好了许多。

### 陷阱5
**存在于应用层的可查询数据设计得过于复杂，应该用过简化解决方案开发出更适合需要的产品**

通常，大多数业务用户都希望简化数据表示方式。此外，对这些数据的访问应限于尽可能少入口。提高获取数据的易用性，会大大提升数仓的价值。

其实，这点就引入了数据湖的概念，将数据异构同质化，数据湖的概念会专篇细讲。同时也能发现，无论多么高大上的概念都是基于实际开发中需要解决的问题，脱离了实际，一切都是空谈。

### 陷阱4
**烟囱式开发，不考虑使用可共享的、一致性维度将数据模型联系在一起**

开发中常提到维度的一致性，当维度在整个数据仓库中不一致时，就是典型的烟囱式开发。其实，我们使用的维度在本质上是相同的，但是由于数据来自于不同的业务源，并会被随意更新。

典型的例子是“时间”维度，在维模型不一致的情况下，最终用户通常完全不知道为什么一个报表中的数据可能与其他地方生成的报表有显着差异。一种好的做法是将数据模型与主数据管理解决方案联系在一起，该解决方案包含可以在整个数据仓库中普遍使用的参考数据。

### 陷阱3
**只将汇总数据加载到数据仓库中**

在事务数据库和数据仓库之间创建的每个ETL过程中，必须保证要有至少一份原子数据存储到数仓中，方便溯源，即将数据同步一份放在准备区(ODS层)

### 陷阱2
**臆想业务、业务需求及分析，其涉及的数据及支持技术都是静态的**

我们常提到面向企业级的数据模型，什么意思呢？就是尽量不要开发仅限于某个特定业务需求和分析的数据模型，因为业务在不断地发生变化。一个差劲的模型设计通常是开发了重复的数据模型以及命名约定不一致的。

在设计一个“完美”的事实表、维表与规范化程度之间取得平衡并不是一件容易的事情，但是开发出可伸缩的以适应业务发展的数据模型是非常重要的。这就对业务的理解能力要求很高了。

### 陷阱1 
**数据仓库是否成功直接取决于业务人员。如果用户不买账，那么所有的工作都是徒劳**

这个是很致命的陷阱，如果从一开始都没有得到业务和高层的重视和认可，那么数仓项目多半是会夭折。从用户的角度出发，如果用户对数仓的数据存在怀疑，对易用性存在吐槽，或者根本点讲未将数据仓库系统当成他们制定决策的基础，那么根本就不会去使用它，结局只会game over。