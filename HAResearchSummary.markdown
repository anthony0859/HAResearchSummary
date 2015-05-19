# 高可用解决方案总结
-------

**高可用性**（[High Availability](http://en.wikipedia.org/wiki/High_availability)）是指系统满足能够检测并容忍处理硬件、软件、环境和操作等失误造成的故障，从而保持服务的高度可用。对一个系统或者服务架构体系来说这是一个非常重要的属性特征。对于一个应用服务体系来说，如果某个应用服务组件发生故障之后，那么整个服务体系就会瘫痪，这也就是我们常说的**单点失效**（[Single Point of Failure](http://en.wikipedia.org/wiki/Single_point_of_failure)）问题。  

系统可用性可以用这样一个公式来衡量：   
**可用性 = MTTF / (MTTF + MTTR) * 100%**  
MTTF：mean time to failure   平均无故障时间  
MTTR：mean time to repair    平均维修时间

随着云计算等技术的发展，平台所提供的计算资源越来越充足，人们往往更加重视应用服务的高可用性这一特征属性。对于一个服务架构体系来说，如果发生某一组件或者多个组件宕机，导致系统服务不可用，人工修复是一种直观有效的解决方法，但是往往需要花费较长的维修时间，而且成本较高。对于一些可用性要求非常高的场景，比如金融、医疗、航空等行业，即使人工可修复，服务瘫痪时间多延长一点往往也会造成无法挽回的损失，所以我们的目标是使服务瘫痪时间降到最少。   
另外一方面，现在主流 PaaS 平台，比如 [Cloud Foundry](http://www.cloudfoundry.org)，Heroku，Google App Engine ，面向应用服务实例的高可用部署方法是多个实例分散在多个可用区域（Availability Zone）内平行化部署，或者是监控服务实例，当发现实例异常或者失效后，重新部署应用服务。这样的往往会给平台本身的某些组件造成负担，而且重新部署实例需要的时间往往较长，可能达不到理想的高可用性要求。目前，工业界和学术界都已经有很多比较成熟的先验性高可用解决方案，现将一些主流方案总结于此。  

根据高可用对象特性可以大致有三种情况：
> * 数据存储级
> * 服务组件级
> * 虚拟机级

PS：这三个层级划分没有严格的横向或者纵向关系，只是根据对象的特性进行一个汇总，下面介绍每一类情况相应的高可用解决方案。


## 数据存储级
数据存储级是面向持久化数据这一类服务对象的高可用。

### 1. [RAID](http://en.wikipedia.org/wiki/RAID)

RAID 技术作为高性能、高可靠的存储技术，已经得到了非常广泛的应用。 RAID 主要利用数据条带、镜像和数据校验技术来获取高性能、可靠性、容错能力和扩展性，根据运用或组合运用这三种技术的策略和架构，可以把 RAID 分为不同的等级，以满足不同数据应用的需求。目前业界公认的标准是 RAID0 ~ RAID5 ，除 RAID2 外的四个等级被定为工业标准，而在实际应用领域中使用最多的 RAID 等级是 RAID0 、 RAID1 、 RAID3 、 RAID5 、 RAID6 和 RAID10 。

从实现角度看，RAID 主要分为软 RAID 、硬 RAID 以及软硬混合 RAID 三种。软 RAID 所有功能均有操作系统和 CPU 来完成，没有独立的 RAID 控制 / 处理芯片和 I/O 处理芯片，效率自然最低。硬 RAID 配备了专门的 RAID 控制 / 处理芯片和 I/O 处理芯片以及阵列缓冲，不占用 CPU 资源，但成本很高。软硬混合 RAID 具备 RAID 控制 / 处理芯片，但缺乏 I/O 处理芯片，需要 CPU 和驱动程序来完成，性能和成本 在软 RAID 和硬 RAID 之间。

RAID 每一个等级代表一种实现方法和技术，等级之间并无高低之分。在实际应用中，应当根据用户的数据应用特点，综合考虑可用性、性能和成本来选择合适的 RAID 等级，以及具体的实现方式。

### 2. [SAN](http://en.wikipedia.org/wiki/Storage_area_network) 架构集群存储

基于SAN架构的集群存储系统，后端存储采用中高端磁盘阵列子系统，支持RAID0、1、5、6、10等不同级别RAID等级，并通过光纤FC接口连接到各个集群节点。SAN磁盘阵列通过不同RAID等级对数据进行保护，通过冗余机制提供高可用性，同时降低了一定程度的存储利用率。在这种架构下，如果再采用副本或者纠删码来提供集群服务高可性，就会进一步降低存储利用率或者大幅降低集群系统性能。当集群节点服务器发生故障时，后端SAN存储通常仍然处于正常工作状态，存储在其上面的数据也是完整一致的。因此，完全可以从正常工作的其他集群节点中选择一个节点来接管故障节点的资源和服务，继续对外提供数据服务，保证业务的连续性。面向基于SAN架构的集群存储系统，可以采用全活HA架构技术，不仅接管故障节点的IP和服务进程资源，而且接管故障节点的存储软件服务进程和物理存储资源，支持NFS/CIFS/HTTP/FTP/ISCSI等协议协议。利用TCP/IP协议的连接重连技术，还可以实现类似CTDB对故障节点的透明接管，不会产生接管期间的业务中断。这些方法保证集群存储系统的存储利用率以及系统性能不会受到影响，并且可以透明接管完整的系统资源，提供更高的系统可用性。想要详细了解集群存储有关技术可以通过这篇[博客](http://blog.csdn.net/liuaigui/article/details/8882141)。

### 3. 云平台[对象存储](http://en.wikipedia.org/wiki/Object_storage)服务

我们所做的工作主要是基于 [OpenStack](http://www.openstack.org/) 开源平台，所以只对OpenStack的Swift项目进行了调研。Swift对象存储实现HA主要是基于一致性散列、最终数据一致性模型等原理，采用完全对称、面向资源的分布式系统架构设计，所有组件都可扩展，避免因单点失效而扩散并影响整个系统运转；通信方式采用非阻塞式 I/O 模式，提高了系统吞吐和响应能力。这篇[文章](http://www.ibm.com/developerworks/cn/cloud/library/1310_zhanghua_openstackswift/)比较详细的介绍了Swift原理、架构和API。

目前主流的对象存储服务有下面这些：
> * Amazon S3
> * OpenStack Swift
> * 阿里云OSS 等

### 4. 其他解决方案
比如： rysc+inotify 组合实现数据的实时同步等。 

## 服务组件级
服务组件级是面向驻在内存中正在运行并且提供服务的一类组件，由一些进程、线程等实体组成。对于服务组件这一级高可用，主要是通过网络和高可用集群软件将冗余的高可用硬件和软件组件组合成集群系统，从而能够消除单点故障，将宕机时间降到最少。而高可用集群软件的主要作用就是实现故障检查和业务切换的自动化。

根据应用服务特性，一般可以把服务分为两类：**无状态服务** 和 **有状态服务**。
1）无状态服务：（Stateless Service）对单次请求的处理，不依赖其他请求，也就是说，处理一次请求所需的全部信息，要么都包含在这个请求里，要么可以从外部获取到（比如说数据库），服务器本身不存储任何信息，不记录状态，只负责单独响应请求。
2）有状态服务：（Stateful Service）则相反，它会在自身保存一些数据，先后的请求是有关联的，服务器在运行和响应过程中，需要在内存中不断更新自己的状态。

[高可用集群](http://en.wikipedia.org/wiki/High-availability_cluster)通常规模是两个节点，因为这是提供冗余的最低要求。
服务高可用主要有两种模式：**主备模式** 和 **主主模式**。
1）主备模式：（Active/Passive）为服务组件提供完整的冗余实例作为从属节点，当关联的主节点失效时，从属节点才立即上线进行替代。
    
2）主主模式：（Active/Active）为应用服务提供两个或者多个相同的实例同时运行，服务请求在网络中一般通过负载均衡转发到服务实例作出响应。

一般情况下，同一种服务根据实际需求情况，我们可以选择部署成主备模式或者主主模式。

面向服务组件，我们总结出高可用部署方案如下表：
<table>
	<tr>
		<th></th>
		<th>主备模式</th>
		<th>主主模式</th>
	</tr>
	<tr>
		<td>无状态服务</td>
		<td>集群冗余</td>
		<td>负载均衡</td>
	</tr>
	<tr>
		<td>有状态服务</td>
		<td>集群冗余 / 共享存储机制</td>
		<td>负载均衡 / 状态一致性保持机制</td>
	</tr>
</table>
下面简单介绍一下比较流行的集群资源管理、负载均衡、共享存储和各种不同的状态一致性保持机制技术。

###集群资源管理

[Linux-HA](http://www.linux-ha.org/wiki/Main_Page) 提供了一套Linux平台上进行集群搭建提供服务高可用的解决方案，包含了多个子项目。目前工业界主流的集群管理工具是：[Pacemaker](http://clusterlabs.org/) + Corosync 组合。Pacemaker 是一个集群资源管理器，Corosync 是执行高可用应用程序的通信组系统。[Heartbeat](http://linux-ha.org/wiki/Heartbeat) 也负责心跳服务和集群通信，跟Corosync类似，但运行机制不及Corosync，远不如后者流行。

###负载均衡
负载均衡器一般分为硬件级和软件级。常用的负载均衡软件：[LVS](http://www.linuxvirtualserver.org/)、[Nginx](http://nginx.org/cn/)、[HAProxy](http://www.haproxy.org/) 。关于这三者的对比，参考这篇[博客](http://yuhongchun.blog.51cto.com/1604432/697466)。
负载均衡软件本身也需要实现高可用，[Keepalived](http://www.keepalived.org/index.html) 基于虚拟路由冗余协议（[VRRP](http://en.wikipedia.org/wiki/Virtual_Router_Redundancy_Protocol)），模拟路由器的高可用，实现轻量级前端高可用，支持Active/Passive、Active/Active两种模式。   
常用的组合：LVS + Keepalived，Nginx + Keepalived，HAProxy + Keepalived。

###共享存储
对于有状态服务，在主备模式下，需要共享存储机制才能保证服务状态数据对于主备实例都可用。    
[DRBD](http://drbd.linbit.com/)：Linux内核的存储层中的一个分布式存储系统，可用DRBD在两台服务器之间共享块设备，通过TCP层共享文件系统和数据。类似于一个网络RAID-1的功能  
[NFS](http://en.wikipedia.org/wiki/Network_File_System)：容许不同的客户端及服务端通过一组RPC分享相同的文件系统。
还包括：iSCSI等。

###状态一致性保持
对于有状态服务，在主主模式下，需要状态一致性保持机制才能保证服务状态数据在多个实例之间的同步。由于服务的多样性，状态一致性保持机制也各不相同。  
下面是几种常见的解决方案：   
[MySQL](http://dev.mysql.com/doc/refman/5.0/en/ha-overview.html)：Cluster / Amoeba+keepalived+MMM(Master-Master Replication) / Galera  
Web session share：[Memcached](http://memcached.org/) / [Redis](http://redis.io/) / Zookeeper / Session Replication  
Memcached：Repcached / MemcachedHA  
Redis：Keepalived / Sentinel  
[RabbitMQ](https://www.rabbitmq.com)：[Mirrored queue](https://www.rabbitmq.com/ha.html)

## 虚拟机级

业界虚拟化巨头 VMware 已经支持高质量的 VM level high availability 服务，由于其商业性没法对其进行研究。我们主要基于 OpenStack 平台，研究虚拟机实例这一通常用来给我们提供计算资源（CPU、内存、外存等）的平台高可用。

### 1. 实例迁移
基于 Nova 共享存储[配置](http://docs.openstack.org/admin-guide-cloud/content/section_configuring-compute-migrations.html)，实现实例迁移功能。OpenStack提供了 [Evacuate](https://wiki.openstack.org/wiki/Evacuate) 这一接口，其实在底层和 Rebuild 对应同一个操作，从而可以达到虚拟机实例元数据和磁盘级数据高可用的要求。

这种方案优点在于：OpenStack 官方提供支持，用到的技术比较成熟，配置相对简单，另外实例迁移功能还可以用来对云计算资源进行动态管理；缺点是：高可用性不够透明，而且只能保证实例元数据和磁盘级数据高可用，不能满足一些高可用性要求比较高的场景。

### 2. 双主实例
Project [Remus](http://wiki.xen.org/wiki/Remus) 是British Columbia大学计算机技术学院开发的高可用方案。通过在备份服务器上保留实时更新的副本来为运行在 Xen hypervisor 平台上的虚机提供高可用性。Remus 只能支持Xen平台，在Xen 4.0版本后得到较好的支持，它是建立在Xen的Live Migration功能之上的。

学术界经常基于 Remus 项目，在一些开源云平台中，比如 [Eucalyptus](https://www.eucalyptus.com/)、OpenStack 等，部署虚拟机级高可用场景，结合具体实例备份放置算法，透明实现虚拟机高可用。

这种方案优点在于：底层对虚拟机状态进行高频检查异步复制，增量备份，能够对整个实例状态实现高可用，透明性也很强，同时部署者可以自定义实例备份放置算法，比较灵活高效；缺点是：只支持Xen hypervisor，配置也相对复杂，业界应用还不是很广泛。