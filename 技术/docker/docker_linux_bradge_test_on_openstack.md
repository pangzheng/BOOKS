# docker linux bradge test on openstack 
## 1 创建基础环境
### 1.1 物理层
- 交换机在物理网段建立独立 vlan
- 这个网段 vlan 交换机接口开 truck，否则夸主机vm不通
- 网卡开放混淆模式(neutron 初始化会设置)
- 物理层网关地址
- 物理层 ip 段获取

### 1.2 openstack 层
- neutron 创建一个网络，指定物理刚建立物理 vlan，配置路由使用物理网关。
    - 命令行配置 
    
        用命令行创建网络  # 创建指定vlan的网络
    
            neutron net-create --tenant_id <tenant_id> net_1 \
                       --provider:network_type=vlan \
                       --provider:physical_network=physnet1 \
                       --provider:segmentation_id=999
                       
        创建自定义的子网，比如创建172.31.0.0/24网段的网络，ip范围为start=172.31.0.10,end=172.31.0.30（注意这个ip范围不能与传统网络的已有ip冲突），不启用gateway，因为传统网络里应该会有网关，并设置默认网关为172.31.0.1。
    
        这个命令默认会启用dhcp，但是这个dhcp不会对传统网络有影响，因为neutron的dhcp是ip和mac绑定的，所以它不会给别的机器分配ip，注意传统网络里不能有dhcp server，不然虚机可能会分配到传统网络的ip。

            neutron subnet-create --tenant_id <tenant_id> \
                          --no-gateway  \
                          --allocation-pool start=172.31.0.10,end=172.31.0.30 \
                          --host-route destination=0.0.0.0/0,nexthop=172.31.0.1 net_1 172.31.0.0/24
    - 界面模式
    
        在全局界面创建
- 通过管理界面，将网段添加到 vm 主机
                                  
## 2 将新环境加入现有的数人云
- 创建新主机
- 新主机加入两个网卡
    - A 网卡(容器使用)
        
        新的 neutron 创建的网络
    - B 网卡(数人云安装)
        
        原先的网络网段 

## 3 创建 linux bradge 环境
## 4 通过数人云运行服务
## 5 知识点
### 5.1 truck
TRUNK (端口汇聚)功能是将交换机的多个物理端口汇聚在一起形成一个逻辑上的物理端口，同一汇聚组内的多条链路则可视为一条逻辑链路。端口汇聚可以实现用多条链路汇聚成一条逻辑链路增加带宽；同时，同一汇聚组的各个成员端口之间彼此动态备份，提高连接可靠性。

Trunk 功能比较适合于以下情况的具体应用

1. Trunk 功能用于交换机与服务器之间的相联，为服务器提供独享的高带宽。
- Trunk功能用于交换机之间的级联，为交换机之间的数据交换提供高带宽的数据传输能力，提高网络速度，突破网络瓶颈，进而大幅提高网络性能（主要应用）。

Trunk功能举例——例如：为增加带宽，提高连接可靠性，某网吧电影服务器是双网卡且作了绑定，与中心交换机的23、24端口连接；二层交换机的1、2端口与中心交换机的1、2端口连接，如下图所示，那么中心交换机需将1、2端口，23、24端口分别做Trunk。说明：这里的二层交换机也需支持Trunk。

- 每个Trunk中的成员端口不能少于2个。
- Trunk成员的行为需一致，也就是成员的端口参数需一致。
- Trunk与VLAN之间的影响：
在设置Trunk的时候，Trunk所有成员需要在同一个VLAN中，而且其缺省VID和Untag帧处理规则需要一致。无论是创建、修改或删除一个Trunk，其现有VLAN架构均不会改变。即Trunk不会改变既有VLAN的端口成员。
- Trunk与端口安全、端口监控、MTU VLAN之间的影响：
设置成Trunk成员的端口不能再启用端口安全，并且不能设置为监控端口和被监控端口，反之一样。若交换机当前启用了MTU VLAN模式，则不能设置任何Trunk，相反，若设置了Trunk口，则不能启用MTU VLAN模式。
- Trunk带宽的计算：
当使用四个全双工1000Mbps端口构成Trunk时，由于每一个端口上行和下行各是1000Mbps，所以每一个端口的带宽为2000Mbps。它们使用Trunk技术聚合在一起形成的总带宽为8000Mbps（2000Mbps X 4）

### 5.2 Neutron 基础知识 Es 验证
1. Neutron 的 vlan 模式是会映射物理网络的vlan场景，所以vlan层面和物理网络一致。
- Neutron 的 dhcp 只能分配 Neutron 管理的 Mac 地址，从数据库映射mac 分配地址，而非动态获取分配。
- 夸 Vlan 的部分通过 Neutron 的路由实现,不受管理的 ip 通过手动增加规则实现 