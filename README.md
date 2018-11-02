# tensorflow estimator 个人使用经验总结

## tf.estimator.train_and_evaluate 简介
1. 字面理解这个 API 就是用来 train 然后 evaluate 一个 Estimator 的，函数的原型如下：
   tf.estimator.train_and_evaluate(
    estimator,
    train_spec,
    eval_spec
    )
   按照文档的说明，这个函数除了 train 和 evaluate 之外，还可选的提供了模型的导出功能，这样就可以把一个训练好的模型直接转交给业务部门来使用了。可以    算是“产学研”一条龙服务了。所需要的参数也简单明了：一个要被训练的 Estimator 实例，用来指定训练参数的 TrainSpec，以及用琮指定评估和导出参数的      EvalSpec。

   实际上，如果直接使用 Estimator 的 API，完成 train 和 evaluate 已经是很简单的任务了，为什么我们还要使用 train_and_evaluate 这个函数呢？按    照文档里的说明：这个函数提供了一种在本地训练和在分布式环境下训练的行为一致性保证。也就是说，只要你使用 Estimator 和 train_and_evaluate 这一    对“搭档”，就可以实现在本地训练和在分布式集群上训练的双重保证，而不需要修改任何代码。可以想像一下，在完成了本地 CPU 训练的测试之后，直接 push 到    Cloud ML Engine 上，分分钟完成一个模型的训练，甚至还可以直接使用 TPU 集群（只要你保证模型里的 op 都是对 TPU 兼容的），这是多么方便的一个工具    啊！

   当然，方便的背后一般都有代价。首先就是，现在这个函数只支持 between-graph replication 这种模式进行分布式的训练。其次呢，为了保证代码在本地和集    群上都可以正常终止，所以只能使用 Estimator 的 max_steps 模式设定终止条件。所以，如果想使用别的方式终止训练，可能就需要一些“技巧”了。

2. 参数说明
   上面我们已经知道 train_and_evaluate 有三个参数，第一个先放在一边，因为这个参数就是创建一个 Estimator 的实例。我们先来看一下另外两个参数：

   2.1 TrainSpec
   TrainSpec 的内容很简单：

   __new__(
        cls,
        input_fn,
        max_steps=None,
        hooks=None
    )
   指定 input_fn，这个是 Estimator 必备的一个参数。然后就是刚才说的 max_steps，是训练的唯一终止条件。最后是可以挂一些         
   tf.train.SessionRunHook，用来在 session 运行的时候做一些额外的操作，比如记录一些 TensorBoard 日志什么的。

   2.2 EvalSpec
   EvalSpec 稍微复杂一点，因为这个任务不但包含了对模型性能的评估，还包含了导出模型到外部存储的功能：

    __new__(
        cls,
        input_fn,
        steps=100,
        name=None,
        hooks=None,
        exporters=None,
        start_delay_secs=120,
        throttle_secs=600
    )
    前两个参数和 TrainSpec 的意思一样。这里的 name 这个参数，是说如果要对不同的数据集进行测试的话，可以给第个数据庥指定一个名字，这个就可以区分出     不同的数据集（这是文档里的说明）。不过，我感觉这个参数的用处主要就是区分 train 和 test 两个阶段。而 train 那个阶段没有一个 name 参数，真是很     不方便。默认 train 就是放在 model_dir 里。hooks 还是那些 hooks，不解释。

    exporters 这个参数只在要 Estimator 导出的时候才有用，因为现在还是研究阶段，所以就先不管这个了。

    start_delay_secs 和 throttle_secs 都是控制在什么时候进行 evaluate。这个参数不知道怎么相互作用的。首先，要在 start_delay_secs 秒之后才     开始一次 evaluate，然后每过 throttle_secs 再进行一次。大概是这样子。不知道我理解得对不对。
