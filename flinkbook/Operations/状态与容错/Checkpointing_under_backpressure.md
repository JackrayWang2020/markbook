# Flink 中的反压下的检查点

通常情况下，对齐 Checkpoint 的时长主要受 Checkpointing 过程中的同步和异步两个部分的影响。 然而，当 Flink 作业正运行在严重的背压下时，Checkpoint 端到端延迟的主要影响因子将会是传递 Checkpoint Barrier 到 所有的算子/子任务的时间。这在 checkpointing process) 的概述中有说明原因。并且可以通过高 alignment time and start delay metrics 观察到。 当这种情况发生并成为一个问题时，有三种方法可以解决这个问题：

- 1、消除背压源头，通过优化 Flink 作业，通过调整 Flink 或 JVM 参数，抑或是通过扩容。
- 2、减少 Flink 作业中缓冲在 In-flight 数据的数据量。
- 3、启用非对齐 Checkpoints。 这些选项并不是互斥的，可以组合在一起。本文档重点介绍后两个选项。

# 缓冲区 Debloating

Flink 1.14 引入了一个新的工具，用于自动控制在 Flink 算子/子任务之间缓冲的 In-flight 数据的数据量。缓冲区 Debloating 机 制可以通过将属性taskmanager.network.memory.buffer-debloat.enabled设置为true来启用。

此特性对对齐和非对齐 Checkpoint 都生效，并且在这两种情况下都能缩短 Checkpointing 的时间，不过 Debloating 的效果对于 对齐 Checkpoint 最明显。 当在非对齐 Checkpoint 情况下使用缓冲区 Debloating 时，额外的好处是 Checkpoint 大小会更小，并且恢复时间更快 (需要保存 和恢复的 In-flight 数据更少)。

有关缓冲区 Debloating 功能如何工作以及如何配置的更多信息，可以参考 network memory tuning guide。 请注意，您仍然可以继续使用在前面调优指南中介绍过的方式来手动减少缓冲在 In-flight 数据的数据量。


# 非对齐 Checkpoints

从Flink 1.11开始，Checkpoint 可以是非对齐的。 Unaligned checkpoints 包含 In-flight 数据(例如，存储在缓冲区中的数据)作为 Checkpoint State的一部分，允许 Checkpoint Barrier 跨越这些缓冲区。因此， Checkpoint 时长变得与当前吞吐量无关，因为 Checkpoint Barrier 实际上已经不再嵌入到数据流当中。

如果您的 Checkpointing 由于背压导致周期非常的长，您应该使用非对齐 Checkpoint。这样，Checkpointing 时间基本上就与 端到端延迟无关。请注意，非对齐 Checkpointing 会增加状态存储的 I/O，因此当状态存储的 I/O 是 整个 Checkpointing 过程当中真 正的瓶颈时，您不应当使用非对齐 Checkpointing。

为了启用非对齐 Checkpoint，您可以：


```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// 启用非对齐 Checkpoint
env.getCheckpointConfig().enableUnalignedCheckpoints();
```

或者在 flink-conf.yml 配置文件中增加配置：

```
ecution.checkpointing.unaligned: true
```

# 对齐 Checkpoint 的超时操作
