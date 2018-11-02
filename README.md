# tensorflow estimator 个人使用经验总结

### tf.estimator.train_and_evaluate 简介
字面理解这个 API 就是用来 train 然后 evaluate 一个 Estimator 的，函数的原型如下：
tf.estimator.train_and_evaluate(
    estimator,
    train_spec,
    eval_spec
)
按照文档的说明，这个函数除了 train 和 evaluate 之外，还可选的提供了模型的导出功能，这样就可以把一个训练好的模型直接转交给业务部门来使用了。可以算是“产学研”一条龙服务了。所需要的参数也简单明了：一个要被训练的 Estimator 实例，用来指定训练参数的 TrainSpec，以及用琮指定评估和导出参数的 EvalSpec。

实际上，如果直接使用 Estimator 的 API，完成 train 和 evaluate 已经是很简单的任务了，为什么我们还要使用 train_and_evaluate 这个函数呢？按照文档里的说明：这个函数提供了一种在本地训练和在分布式环境下训练的行为一致性保证。也就是说，只要你使用 Estimator 和 train_and_evaluate 这一对“搭档”，就可以实现在本地训练和在分布式集群上训练的双重保证，而不需要修改任何代码。可以想像一下，在完成了本地 CPU 训练的测试之后，直接 push 到 Cloud ML Engine 上，分分钟完成一个模型的训练，甚至还可以直接使用 TPU 集群（只要你保证模型里的 op 都是对 TPU 兼容的），这是多么方便的一个工具啊！

当然，方便的背后一般都有代价。首先就是，现在这个函数只支持 between-graph replication 这种模式进行分布式的训练。其次呢，为了保证代码在本地和集群上都可以正常终止，所以只能使用 Estimator 的 max_steps 模式设定终止条件。所以，如果想使用别的方式终止训练，可能就需要一些“技巧”了。
