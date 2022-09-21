## 配置

min.compaction.lag.ms：消息在写入和压缩之间的最小间隔

max.compaction.lag.ms：消息在写入和压缩之间的最大间隔

log.cleaner.min.cleanable.ratio：消息压缩的最小比例，只有当需要压缩的消息比例超过设置值，并且消息大于min.compaction.lag.ms或者超过max.compaction.lag.ms时才会进行消息压缩。

