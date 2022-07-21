# DPDK关键技术

## 1.DPDK技术要点
* 1）UIO用户空间的I/O技术；
* 2）内存池技术；
* 3）大页内存分配；
* 4）无锁循环队列；
* 5）NUMA 通过proc提供内存信息，使用CPU核心尽量使用靠近其所在节点的内存，避免跨numa节点访问内存的性能问题；


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
