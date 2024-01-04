<h1 align="center"><a href ="https://ieeexplore.ieee.org/document/9101373">PA-Tree Polled-Mode Asynchronous B+ Tree for NVMe</a></h1>

- [贡献](#贡献)
- [B+ Tree 的问题](#b-tree-的问题)
- [PA-Tree 的解决方法](#pa-tree-的解决方法)


# 贡献

* PA-Tree
  * 特点
    * 新轮询模型、异步范式：工作线程以交错的方式处理多个索引操作
  * 效果
    * 吞吐量：5x
    * 延迟：-30%



# B+ Tree 的问题

B+树索引遵循同步执行范式，通过 NVMe 提交I/O命令时工作线会进行等待，所以

* 需要生成大量并发工作线程。
* 需要保证索引结构的一致性的**同步开销**，上下文切换的成本和操作系统的调度开销。



# PA-Tree 的解决方法

* **将 Operation 交给一个/少量 working thread 管理**
  * 将 Operation 分为多个state（*ready state* or *waiting state*），放入到 R(C) 和 W(C) 中；
  * R(C)依次处理每个 Operation，直到进入 waiting state，放入到 W(C) ，处理 R(C) 中的下一个；
  * **通过 linear regression model 预测当前时间片是否有新的 R0 或 W0 产生来进行 probe**，如果预测当前时间片不会产生新的 R0 或 W0，**yield CPU**，否则 probe 更新 R(C) 和 W(C)。
* **通过 priority queue 维护 Operation priority**
  * 持有写锁的具有最高的优先级；
  * 先提交的有更高的优先级。
* **Buffer**（*read-only buffer* & *read-write buffer*）
* **Latch**（*read latch* & *write latch*）

