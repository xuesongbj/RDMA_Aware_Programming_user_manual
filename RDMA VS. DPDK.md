# RDMA VS. DPDK

## DPDK

DPDK网络层：

 - 硬件中断->放弃中断流程；
 - 用户层通过设备映射取包->进入用户层协议栈->逻辑层->业务层；
 
### 核心技术
 - 将协议栈上移到用户态，利用UIO技术直接将设备数据映射拷贝到用户态
 - 利用大页(large Page)，降低TLB cache miss，提高TLB访问命中率
 - 通过CPU亲和性，绑定网卡和线程到固定的core，减少CPU任务切换
 - 通过无锁队列，减少资源的竞争
 
### 优势
- 减少中断次数
- 减少内存拷贝次数
- 绕过Linux 的协议栈，用户获得协议栈的控制权，能够定制化协议栈以降低复杂度

### 劣势
- 内核栈转移至用户层增加了开发成本
- 低负荷服务器不实用，会造成CPU空转


## RDMA
网卡硬件收发包并进行协议栈封装/解析，然后将数据存放到指定内存地址，而不需要CPU干预。

![此处输入图片的描述][1]


### 技术核心

协议栈硬件offload:

![此处输入图片的描述][2]
  
### 优势

 - 协议栈offload，解放cpu
 - 减少了中断和内存拷贝，降低时延
 - 高带宽

### 劣势

 - 特定网卡才支持，成本开销相对较大
 - RDMA提供了完全不同于传统网络编程的API，一般需要对现有APP进行改造，引入额外开发成本


## 总结

### 相同点
- 两者均为kernel bypass技术，可以减少中断次数，消除内核态到用户态的内存拷贝

### 不同点
- DPDK是将协议栈上移到用户态，而RDMA是将协议栈下沉到网卡硬件，DPDK仍然会消耗CPU资源
- DPDK的并发度取决于CPU核数，而RDMA的收包速率完全取决于网卡的硬件转发能力
- DPDK在低负荷场景下会造成CPU的无谓空转，RDMA不存在此问题
- DPDK用户可获得协议栈的控制权，可自主定制协议栈；RDMA则无法定制协议栈
 
 
  [1]: https://raw.githubusercontent.com/xuesongbj/RDMA_Aware_Programming_user_manual/master/rdma.png
  [2]: https://raw.githubusercontent.com/xuesongbj/RDMA_Aware_Programming_user_manual/master/rdma_1.png