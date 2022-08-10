# DPDK关键技术

## 1.DPDK技术要点
* UIO用户空间的I/O技术；
* 内存池技术；
* 大页内存分配；
* 无锁循环队列；
* NUMA 通过proc提供内存信息，使用CPU核心尽量使用靠近其所在节点的内存，避免跨numa节点访问内存的性能问题；


## 2.DPDK核心组件
1）环形缓冲管理（librte_ring）  
2）内存池管理（librte_mempool）  
3）网络报文缓冲区管理(librte_mbuf)  
4）定时器管理（librte_timer）  
5）以太网轮询驱动架构  
6）报文转发算法支持  
7）网络协议库（librte_net）  


## 3.DPDK循序渐进
1）认识DPDK  
2）Cache和内存  
3）并行计算  
4）同步互斥机制  
5）报文转发  
6）PCIe与包处理I/O  
7）网卡性能优化  
8）流分类与多队列  
9）硬件加速与功能卸载  
10）X86平台的IO虚拟化  
11）半虚拟化Virtio  
12）加速包处理的vhost优化方案  
13）DPDK与网络功能虚拟化  
14）Open vSwith中的DPDK性能加速  
15）基于DPDK的存储软件优化



## 4.DPDK专家之路
DPDK源码  
1）内核驱动  
2）内存  
3）协议  
4）虚拟化  
5）CPU  
6）安全  

DPDK网络  
1）网络协议栈项目（arp，icmp，udp，ip，tcp）  
2）组件项目（mp，acl，kni，timer，bpf，mubuf）  
3）经典项目（dns，gateway，ddos，firewall，switch，pktgen）  

DPDK框架  
1）VPP  
2）虚拟交换机OVS  
3）golang网卡开发框架nff-go  
4）轻量级switch框架snabb（lua）  
5）高效磁盘IO读写SPDK  

性能测试  
1）性能指标（bps，pps，并发，最大时延，最小时延，平均时延，负载，包速fps，丢包率）  
2）测试方法（vpp sanbox，perf3灌包，rfc2544）  
3）测试工具（perf3，trex，testpmd，pktgen-dpdk）  

