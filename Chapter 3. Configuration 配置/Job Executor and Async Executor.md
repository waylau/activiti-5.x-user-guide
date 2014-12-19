##Job Executor and Async Executor (since version 5.17.0)

从版本 5.17.0 开始，除了 Job Executor（作业执行器）之外， Activiti 还提供了一个 Async executor （异步执行器）。 Async executor 在Activiti 引擎中 是一个更好的性能和对数据库更友好的执行异步作业的方式。因此建议切换到  Async executor，在默认情况下仍然使用旧的 job executor 。更多的信息可以在用户指南的高级部分找到。