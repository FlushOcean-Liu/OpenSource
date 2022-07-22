# VirtIo详解

## 简介
VirtIO由 Rusty Russell 开发，对准虚拟化 hypervisor 中的一组通用模拟设备IO的抽象。
* Virtio 是一种前后端架构，包括前端驱动（Guest内部）、后端设备（QEMU设备）、传输协议（vring）;
* VirtIo 是一种 I/O 半虚拟化解决方案，是一套通用 I/O 设备虚拟化的程序;
* 对半虚拟化 Hypervisor 中的一组通用 I/O 设备的抽象;
* 提供了一套上层应用与各 Hypervisor 虚拟化设备（KVM，Xen，VMware等）之间的通信框架和编程接口;
* 减少跨平台所带来的兼容性问题，大大提高驱动程序开发效率。
