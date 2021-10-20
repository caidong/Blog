---
title: SparkCore 核心概念
date: 2021-10-30 23:12:43
tags:
- spark
- sparkCore
categories: 
- spark
---

 **说明：本文大多数定义来自尚硅谷大数据文档，自己摘抄的读书笔记。**

### RDD(Resilient Distributed DataSet) 弹性分布式数据集

 	1. RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。
 	2. 特点：只读 、不可变、可分区、可并行计算
 	3. 弹性
     - 存储弹性：内存与磁盘的自动切换
     - 容错弹性：数据丢失可以自动恢复
     - 计算弹性：计算出错重试机制
     - 分片弹性：根据需要重新分片

### 算子

1. #### 转换 Transformation

   1. map(func) 返回由每个输入元素经过func函数处理后的RDD
   2. mapPartitions(func) 一个函数一次处理所有分区
   3.  mapPartition是WithIndex(func) 带索引值, func参数（Int,Iterator[T]）
   4. flatMap(func) 每一个输入元素可以被映射为0个或多个输出元素
   5. groupBy(func) 分组，按照传入函数的返回值进行分组。将相同的key对应的值放入一个迭代器。
   6. filter(func) 过滤结果为False的值
   7. sample(withReplacement,fraction,seed) 随机抽样，withReplacement 是否有放回，fraction 抽样比例，seed随机数生成器种子
   8. distinct([numTasks]),对源去重，返回新的RDD
   9.  coalesce(numPartitions) 合并分区，用于大数据集过滤后，提高小数据集执行效率
   10. repartition(numPartitions) 重新混洗所有数据
   11. 

2. ####  行动 Action



##### map()与mapPartitions()区别

	1. map 每次处理一条
 	2. mapPartitions 每次处理一个分区
 	3. 使用场景：当内存空间较大时候建议使用mapPartitions(),以提高处理效率

##### coalesce与repartition()区别

