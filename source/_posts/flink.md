---
title: flink
date: 2023-04-05 20:35:56
tags:
---
SchedulingTopology

DefaultExecutionTopology



DefaultSchedulingPipelinedRegion





在 Apache Flink 中，Task（任务）是执行作业的最小单位，每个任务负责处理数据流中的一个子集。具体来说，Task 会从数据源（如 Kafka、HDFS 等）中读取数据，然后对数据进行转换、处理、聚合等操作，最后将结果输出到数据接收器（如文件系统、数据库等）中。

在 Flink 中，每个任务都有一个对应的执行实例，称为 Task Execution（任务执行）。每个任务执行实例都有自己的状态和缓存，可以独立地读取和处理数据。Flink 会根据数据流的并行度和任务的并行度动态地将任务分配给不同的 Task Execution 实例来执行，以实现高效的并行计算。

任务之间可以通过网络进行通信和数据交换，以实现数据流的处理和转换。Flink 中的任务调度和数据流管理都是自动化的，用户无需手动编写任务调度逻辑，只需要定义好作业的拓扑结构和数据处理逻辑，Flink 会自动将其转化为任务并执行。









```
Executor
```

源码中的Task 其实是SubTask

Pipelined Region Scheduling和Eager Scheduling   怎么确定blocking还是pipelined

共享slot





在Flink中，一个Slot是一个可用资源单元，通常是一个节点上的一个进程，用于执行Flink任务中的Subtask。为了最大化利用资源，Flink允许在一个Slot上共享执行多个Subtask，这种方式被称为Slot共享（Slot sharing）。

在Flink任务中，可以通过设置任务中的Operator之间的边界（即Task之间的连接方式）来控制Slot的共享。Flink中有两种边界类型，分别是Pipelined和Blocking，它们的区别在于数据是否需要被缓存以等待下一个Task的执行。

- Pipelined边界：当Operator之间的边界类型为Pipelined时，数据会流式传输，即生产者将数据发送给消费者的同时，可以立即继续执行下一个Task，不需要等待消费者的结果。这种方式可以最大化地利用Slot，允许多个Task共享一个Slot，从而提高任务的并行度和性能。
- Blocking边界：当Operator之间的边界类型为Blocking时，生产者必须等待消费者的结果，即必须将数据缓存在内存或磁盘中，等待消费者准备好处理数据。这种方式会导致Slot被占用，不能被其他Task共享，从而限制任务的并行度和性能。

需要注意的是，Slot共享可能会导致Subtask之间互相干扰，从而影响任务的性能和稳定性。因此，在使用Slot共享时，应该根据任务的特点和数据的流量进行适当的配置，避免出现资源争用和性能下降的情况。



抱歉，我的表述有些混淆。实际上，一个Slot默认情况下只能运行一个Subtask，但是在一些情况下，可以通过Slot共享的方式，在一个Slot上同时运行多个Subtask。

具体来说，Slot共享可以分为两种方式：基于Pipeline的Slot共享和基于Co-location的Slot共享。其中，基于Pipeline的Slot共享是指多个Operator之间通过Pipeline连接，共享一个Slot。这种方式可以在保证任务性能的前提下最大化地利用Slot资源。而基于Co-location的Slot共享是指将多个Operator分配到同一个TaskManager上，并设置相同的Slot共享组，从而在同一个TaskManager上共享Slot。这种方式可以确保相关的Operator在同一个TaskManager上运行，减少网络传输和通信开销。

需要注意的是，Slot共享可能会导致Subtask之间互相干扰，从而影响任务的性能和稳定性。因此，在使用Slot共享时，应该根据任务的特点和数据的流量进行适当的配置，避免出现资源争用和性能下降的情况。