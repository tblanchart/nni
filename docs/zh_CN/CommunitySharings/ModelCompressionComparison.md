# 滤波器剪枝算法比较

为了初步了解各种滤波器剪枝算法的性能，在一些基准模型和数据集上使用各种剪枝算法进行了广泛的实验。 此文档中展示了实验结果。 此外，还对这些实验的复现提供了友好的说明，以促进对这项工作的进一步贡献。

## 实验设置

实验使用以下剪枝器/数据集/模型进行:

* 模型：[VGG16, ResNet18, ResNet50](https://github.com/microsoft/nni/tree/master/examples/model_compress/models/cifar10)

* 数据集：CIFAR-10

* 剪枝器：
    - 剪枝器包括：
        - Pruners with scheduling : `SimulatedAnnealing Pruner`, `NetAdapt Pruner`, `AutoCompress Pruner`. Given the overal sparsity requirement, these pruners can automatically generate a sparsity distribution among different layers.
        - One-shot pruners: `L1Filter Pruner`, `L2Filter Pruner`, `FPGM Pruner`. The sparsity of each layer is set the same as the overall sparsity in this experiment.
    - Only **filter pruning** performances are compared here.

    For the pruners with scheduling, `L1Filter Pruner` is used as the base algorithm. That is to say, after the sparsities distribution is decided by the scheduling algorithm, `L1Filter Pruner` is used to performn real pruning.

    - All the pruners listed above are implemented in [nni](https://github.com/microsoft/nni/tree/master/docs/en_US/Compressor/Overview.md).

## 实验结果

For each dataset/model/pruner combination, we prune the model to different levels by setting a series of target sparsities for the pruner.

Here we plot both **Number of Weights - Performances** curve and **FLOPs - Performance** curve. As a reference, we also plot the result declared in the paper [AutoCompress: An Automatic DNN Structured Pruning Framework for Ultra-High Compression Rates](http://arxiv.org/abs/1907.03141) for models VGG16 and ResNet18 on CIFAR-10.

实验结果如下图所示：

CIFAR-10, VGG16:

![](../../../examples/model_compress/comparison_of_pruners/img/performance_comparison_vgg16.png)

CIFAR-10, ResNet18:

![](../../../examples/model_compress/comparison_of_pruners/img/performance_comparison_resnet18.png)

CIFAR-10, ResNet50:

![](../../../examples/model_compress/comparison_of_pruners/img/performance_comparison_resnet50.png)

## 分析

从实验结果中，得到以下结论：

* Given the constraint on the number of parameters, the pruners with scheduling ( `AutoCompress Pruner` , `SimualatedAnnealing Pruner` ) performs better than the others when the constraint is strict. However, they have no such advantage in FLOPs/Performances comparison since only number of parameters constraint is considered in the optimization process;
* The basic algorithms `L1Filter Pruner` , `L2Filter Pruner` , `FPGM Pruner` performs very similarly in these experiments;
* `NetAdapt Pruner` can not achieve very high compression rate. This is caused by its mechanism that it prunes only one layer each pruning iteration. This leads to un-acceptable complexity if the sparsity per iteration is much lower than the overall sparisity constraint.

## 实验复现

### 实现细节

* 实验结果都是在 NNI 中使用剪枝器的默认配置收集的，这意味着当我们在 NNI 中调用一个剪枝器类时，我们不会更改任何默认的类参数。

* Both FLOPs and the number of parameters are counted with [Model FLOPs/Parameters Counter](https://github.com/microsoft/nni/blob/master/docs/en_US/Compressor/CompressionUtils.md#model-flopsparameters-counter) after [model speed up](https://github.com/microsoft/nni/blob/master/docs/en_US/Compressor/ModelSpeedup.md). This avoids potential issues of counting them of masked models.

* 实验代码在[这里](https://github.com/microsoft/nni/tree/master/examples/model_compress/auto_pruners_torch.py)。

### 实验结果展示

* 如果遵循[示例](https://github.com/microsoft/nni/tree/master/examples/model_compress/auto_pruners_torch.py)的做法，对于每一次剪枝实验，实验结果将以JSON格式保存如下：
    ``` json
    {
        "performance": {"original": 0.9298, "pruned": 0.1, "speedup": 0.1, "finetuned": 0.7746}, 
        "params": {"original": 14987722.0, "speedup": 167089.0}, 
        "flops": {"original": 314018314.0, "speedup": 38589922.0}
    }
    ```

* 实验结果保存在[这里](https://github.com/microsoft/nni/tree/master/examples/model_compress/experiment_data)。 可以参考[分析](https://github.com/microsoft/nni/tree/master/examples/model_compress/experiment_data/analyze.py)来绘制新的性能比较图。

## 贡献

### 待办事项

* 有 FLOPS/延迟 限制的剪枝器
* 更多剪枝算法/数据集/模型

### 问题
关于算法实现及实验问题，请[发起 issue](https://github.com/microsoft/nni/issues/new/)。
