# DEVOPS
## DevOps 理念，(自动化发布互联网产品)
每天发布 10+ 次概念，大大震撼了业界。 几个核心理念：

- 业务快速发展，需要拥抱变更，小步快跑
- Ops 目标不是为了网站稳定和快速，而是推动业务快速发展
- 基于自动化工具提高 Dev/Ops 联接：代码版本管理、监控
- 高效沟通：IRC/IM Robot（ChatBot 套路）
- 信任、透明、高效、互助的沟通文化

## 核心目标
DevOps 目标是"提高业务系统交付速度"，并为之提供相关工具、制度和服务,关心软件/服务的整个生命周期

### DevOps 工作重心
公式

	速度=总量/时间
添上工程行业术语

	交付速度=((功能特性 * 工程质量)/交付时间)* 交付风险
### DevOps KPI

- 工程质量
- 交付时间
- 交付风险

### DevOps 职能
- 管理应用全生命周期（需求、设计、开发、QA、发布、运行）
- 关注全流程效率提升，挖掘瓶颈点并将其解决
- 自动化运维平台设计和研发工作（标准化、自动化、平台化）
- 支持运维系统，包括 虚拟化技术、资源管理技术、监控技术、网络技术

### 团队价值
在工程质量保证的情况下，尽量保加快交付周期，只要是业务团队就需要

# SRE(DevOps 最佳实现 SRE)
## 部门成立的原因
Google 服务于十几亿用户服务，短暂服务不可用会带来致命后果。 因此 Google SRE 产生了。 这个职位为大规模集群服务，小型团队不需要这样职位设定（可能也招不起或用不起真正 SRE ）
### SRE部门成立的目标
这个团队设立目的是帮助 Google 生产环境服务运行更稳定、健壮、可靠,全部是软件工程师(也就是开发)
### 团队价值
减少大规模分布式系统的短暂运行不可用的经济损失

##  DEVOPS 与 SRE
### 工作区别
- DevOps
	- 设定应用生命管理周期制度，扭转流程
	- 应用管理变更
	- 应用管理故障
	- 开发、管理 DEV/QA 工程师使用开发/测试系统的平台和系统
	- 开发、管理 发布系统
	- 开发、选型、管理 运维辅助系统
		- 监控、报警系统
		- 日志系统
		- 安全系统
		- 业务大数据系统
		- ....
	- 开发、管理 权限系统
	- 开发、选型、管理 CMBD
- SRE
	- 管理变更
	- 管理故障
	- 制定 SLA 服务标准
	- 开发、选型、管理 各类中间件
	- 开发、管理 分布式监控系统
	- 开发、管理 分布式追踪系统
	- 开发、管理 性能监控、探测系统（dtrace、火焰图）
	- 开发、选型、培训 性能调优工具

### 工作关注点区别
- devops(是一种做事风格)

	一个 DevOps Team 通常会提供一串工具链， 这其中会包括
	
	- 开发工具
	- 版本管理工具
	- CI 持续交付工具
	- CD 持续发布工具
	- 报警工具
	- 故障处理
- SRE(是一个明确的职位，挑战更大)

	SRE 会涉及具体业务

	- 变更
	- 故障
	- 性能
	- 容量相关问题

	产出工具链会有
	
	- 容量测量工具
	- Logging 日志工具
	- Tracing 调用链路跟踪工具
	- Metrics 性能度量工具
	- 监控报警工具

	工作能力需要
	
	- 计算机体系结构能力；
	- 高吞吐高并发优化能力；
	- 可扩展系统设计能力；
	- 复杂系统设计能力；
	- 业务系统排查能力

### 技术能力要求
#### DevOps
- Operator 技能
	- Linux Basis
		- 基本命令操作
		- Linux FHS（Filesystem Hierarchy Standard 文件系统层次结构标准）
		- Linux 系统（差异、历史、标准、发展）
	- 脚本
		- Bash / Python
	- 基础服务
		- DHCP / NTP / DNS / SSH / iptables / LDAP / CMDB
	- 自动化工具
		- Fabric / Saltstack / Chef / Ansible
	- 基础监控工具
		- Zabbix / Nagios / Cacti
	- 虚拟化
		- KVM 管理 / XEN 管理 / vSphere 管理 / Docker
		- 容器编排 / Mesos / Kubernetes
	- 服务
		- Nginx / F5 / HAProxy / LVS 负载均衡
		- 常见中间件 Operate（启动、关闭、重启、扩容）
- Dev
	- 语言
		- Python
		- Go（可选）
		- Java（了解部署）
	- 流程和理论
		- Application Life Cycle
		- 12 Factor
		- 微服务概念、部署、生命周期
		- CI 持续集成 / Jenkins / Pipeline / Git Repo Web Hook
		- CD 持续发布系统
	- 基础设施
		- Git Repo / Gitlab / Github
		- Logstash / Flume 日志收集
		- 配置文件管理（应用、中间件等）
		- Nexus / JFrog / Pypi 包依赖管理
		- 面向 开发 / QA 开发环境管理系统
		- 线上权限分配系统
		- 监控报警系统
		- 基于 Fabric / Saltstack / Chef / Ansible 自动化工具开发

#### SRE
- 语言和工程实现
	- 深入理解开发语言（假设是 Java）
	- 业务部门使用开发框架
	- 并发、多线程和锁
	- 资源模型理解：网络、内存、CPU
	- 故障处理能力（分析瓶颈、熟悉相关工具、还原现场、提供方案）

- 常见业务设计方案和陷阱（比如 Business Modeling，N+1、远程调用、不合理 DB 结构）
- MySQL / Mongo OLTP 类型查询优化
- 多种并发模型，以及相关 Scalable 设计

- 问题定位工具
	- 容量管理
	- Tracing 链路追踪
	- Metrics 度量工具
	- Logging 日志系统
- 运维架构能力
	- Linux 精通，理解 Linux 负载模型，资源模型
	- 熟悉常规中间件（MySQL Nginx Redis Mongo ZooKeeper 等），能够调优
	- Linux 网络调优，网络 IO 模型以及在语言里面实现
	- 资源编排系统（Mesos / Kubernetes）
- 理论
	- 容量规划方案
	- 熟悉分布式理论（Paxos / Raft / BigTable / MapReduce / Spanner 等），能够为场景决策合适方案
	- 性能模型（比如 Pxx 理解、Metrics、Dapper）
	- 资源模型（比如 Queuing Theory、负载方案、雪崩问题）
	- 资源编排系统（Mesos / Kurbernetes）

## 参考
[DevOps和SRE的区别](https://zhuanlan.zhihu.com/p/87598465)			