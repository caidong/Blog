#### 官方介绍

HDFS( Hadoop Distributed File System) 是Hadoop下的 分布式文件系统个，具有*分布式、高吞吐*等特性，并可以利用低成本的硬件。

###  HDFS 组件

![img](https://camo.githubusercontent.com/087df9047d11025b32d665904354f17867d6fe7681c505ac8694387448d69926/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f686466736172636869746563747572652e706e67)

HDFS遵从主/从架构，由单个主NN(NameNode)和多个DN(DataNode)组成：

- DataNode :负责提供来自文件系统客户端的读写请求，执行块的创建、删除等操作。

- NameNode :负责执行有关 *文件系统命名空间*的操作，例如打开、关闭、重命名文件和目录等。它同时还负责集群元数据的存储，记录着文件中各个数据块的位置信息。

#### 文件命名空间

HDFS的 文件系统命名空间 的层次结构与大多数文件系统类似（Linux）
