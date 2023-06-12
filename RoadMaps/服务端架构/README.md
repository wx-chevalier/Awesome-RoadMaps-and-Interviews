# 服务端开发工程师

这是全栈的时代，我们更多地以业务来划分而非单纯地前后端，[Backend Series | 服务端应用程序开发与系统架构/微服务架构与实践](https://github.com/wx-chevalier/Backend-Notes)承载了笔者在服务端的总结与经验，其包含了**服务端应用程序开发基础，深入浅出 Node.js 全栈架构，Spring Boot 5 与 Spring Cloud 微服务实践**等内容。

- [Backend-Boilerplates](https://github.com/wx-chevalier/Backend-Boilerplates) is Boilerplate for Your Server Side(Backend) Application, Java | Spring(Boot, Cloud) | Node.js(Express, Koa, Egg) | Go | Python | DevOps.

- [winter-boot](https://github.com/wx-chevalier/winter-boot) is Another boot for your Java applications like Spring Boot, but Winter is coming.

# 架构师

根据企业中职责的划分，我们往往可以将软件架构，及关联的架构师划分为以下几类：

- 业务架构/解决方案架构：了解客户/业务方的痛点，项目定义，现有环境；梳理高阶需求和非功能性需求；沟通，方案建议，多次迭代，交付总体架构。

- 应用架构：专注于结合企业需求，定制化 IT 解决方案；大部分需要交付的工作包括总体架构，应用架构，数据架构，甚至部署架构。根据业务场景的需要，设计应用的层次结构，制定应用规范、定义接口和数据交互协议等。并尽量将应用的复杂度控制在一个可以接受的水平，从而在快速的支撑业务发展的同时，在保证系统的可用性和可维护性的同时，确保应用满足非功能属性要求（性能、安全、稳定性等）。

- 数据架构：专注于构建数据中台，统一数据定义规范，标准化数据表达，形成有效易维护的数据资产。打造统一的大数据处理平台，包括数据可视化运营平台、数据共享平台、数据权限管理平台等。

- 中间件架构：专注于中间件系统的构建，需要解决服务器负载，分布式服务的注册和发现，消息系统，缓存系统，分布式数据库等问题，同时架构师要在 CAP 之间进行权衡。

- 运维架构：负责运维系统的规划、选型、部署上线，建立规范化的运维体系。

- 物理架构：物理架构关注软件元件是如何放到硬件上的，专注于基础设施，某种软硬件体系，甚至云平台，包括机房搭建、网络拓扑结构，网络分流器、代理服务器、Web 服务器、应用服务器、报表服务器、整合服务器、存储服务器和主机等。

前文讨论的更多是应用层的知识，而对于更底层的操作系统、数据库、大数据处理等[分布式基础架构](https://github.com/wx-chevalier/Distributed-Infrastructure-Notes)相关内容，都存放在了 [Distributed-Infrastructure-Notes | 深入浅出分布式基础架构](https://github.com/wx-chevalier/Distributed-Infrastructure-Notes)系列中，主要包含分布式计算、分布式系统、数据存储、虚拟化、网络、操作系统等几个部分。

如上文所述，我们需要在重构轮子中成长：

- [Reinvent-MQ](https://github.com/wx-chevalier/Reinvent-MQ) 即是 Multiple home-made Message Queues, LocalMQ(akin RocketMQ), PongoMQ(akin Kafka), etc.

- [Reinvent-DB](https://github.com/wx-chevalier/Reinvent-DB) 即是 Multiple home-made Databases, Godis(akin Redis), HiSQL(akin MySQL), MemDB, DataGo(akin ETL) etc. Understanding DBs by Reinventing It.

- [Focker](https://github.com/wx-chevalier/Focker) 是从零开始自定义的类 Docker 简化版容器实现。

- [SparkChain](https://github.com/wx-chevalier/SparkChain) 即是在区块链方面的实验探索的积累。
