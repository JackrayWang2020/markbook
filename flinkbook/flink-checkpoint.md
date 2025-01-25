# flink-checkpoint笔记



# 0、Flink 开箱即用地提供了两种 Checkpoint 存储类型：

- JobManagerCheckpointStorage
- FileSystemCheckpointStorage

_如果配置了 Checkpoint 目录，将使用 FileSystemCheckpointStorage，否则系统将使用 JobManagerCheckpointStorage。_