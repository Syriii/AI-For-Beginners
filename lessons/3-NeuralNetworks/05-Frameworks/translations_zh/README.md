# 神经网络框架

正如我们已经了解到的，要高效地训练神经网络，我们需要做两件事：

* 在张量上进行操作，例如乘法、加法，以及计算一些函数，如 sigmoid 或 softmax
* 计算所有表达式的梯度，以便执行梯度下降优化

## [课前测验](https://red-field-0a6ddfd03.1.azurestaticapps.net/quiz/105)

虽然 `numpy` 库可以完成第一部分，但我们需要某种机制来计算梯度。在我们在上一节中开发的 [我们的框架](../../../../../lessons/3-NeuralNetworks/04-OwnFramework/OwnFramework.ipynb) 中，我们不得不手动编写 `backward` 方法中的所有导数函数，该方法执行反向传播。理想情况下，一个框架应该让我们有机会计算我们可以定义的 *任何表达式* 的梯度。

另一个重要的事情是能够在 GPU 或任何其他专用计算单元上执行计算，例如 [TPU](https://en.wikipedia.org/wiki/Tensor_Processing_Unit)。深度神经网络的训练需要 *大量* 的计算，能够在 GPU 上并行化这些计算是非常重要的。

> ✅ “并行化”一词意味着将计算分布到多个设备上。

目前，两个最流行的神经网络框架是：[TensorFlow](http://TensorFlow.org) 和 [PyTorch](https://pytorch.org/)。这两个框架都提供了低级 API，以便在 CPU 和 GPU 上操作张量。在低级 API 之上，还有更高级的 API，分别称为 [Keras](https://keras.io/) 和 [PyTorch Lightning](https://pytorchlightning.ai/)。

低级 API | [TensorFlow](http://TensorFlow.org) | [PyTorch](https://pytorch.org/)
--------------|-------------------------------------|--------------------------------
高级 API| [Keras](https://keras.io/) | [PyTorch Lightning](https://pytorchlightning.ai/)

**两个框架中的低级 API** 允许您构建所谓的 **计算图**。这个图定义了如何使用给定的输入参数计算输出（通常是损失函数），并可以在可用的情况下推送到 GPU 上进行计算。有函数可以对这个计算图进行微分并计算梯度，这些梯度可以用于优化模型参数。

**高级 API** 将神经网络视为 **层的序列**，使构建大多数神经网络变得更加简单。训练模型通常需要准备数据，然后调用 `fit` 函数来完成任务。

高级 API 使您能够非常快速地构建典型的神经网络，而无需担心许多细节。与此同时，低级 API 提供了对训练过程的更多控制，因此在处理新的神经网络架构时，研究人员通常使用它们。

还需要理解的是，您可以同时使用这两种 API，例如，您可以使用低级 API 开发自己的网络层架构，然后在使用高级 API 构建和训练的更大网络中使用它。或者，您可以使用高级 API 将网络定义为层的序列，然后使用自己的低级训练循环进行优化。这两种 API 使用相同的基本概念，并且设计上能够很好地协同工作。

## 学习

在本课程中，我们为 PyTorch 和 TensorFlow 提供了大部分内容。您可以选择自己喜欢的框架，只需查看相应的笔记本。如果您不确定选择哪个框架，可以阅读一些关于 **PyTorch 与 TensorFlow** 的讨论。您也可以查看这两个框架以获得更好的理解。

在可能的情况下，我们将使用高级 API 以简化操作。然而，我们认为从基础开始理解神经网络的工作原理是重要的，因此在开始时我们将使用低级 API 和张量进行工作。不过，如果您想快速入门，并且不想花费大量时间学习这些细节，您可以跳过这些内容，直接进入高级 API 笔记本。

## ✍️ 练习：框架

继续在以下笔记本中学习：

低级 API | [TensorFlow+Keras 笔记本](../../../../../lessons/3-NeuralNetworks/05-Frameworks/IntroKerasTF.ipynb) | [PyTorch](../../../../../lessons/3-NeuralNetworks/05-Frameworks/IntroPyTorch.ipynb)
--------------|-------------------------------------|--------------------------------
高级 API| [Keras](../../../../../lessons/3-NeuralNetworks/05-Frameworks/IntroKeras.ipynb) | *PyTorch Lightning*

掌握框架后，让我们回顾一下过拟合的概念。

# 过拟合

过拟合是机器学习中一个极其重要的概念，正确理解它非常重要！

考虑以下近似 5 个点的问题（在下面的图中用 `x` 表示）：

![linear](../../../../translated_images/overfit1.f24b71c6f652e59e6bed7245ffbeaecc3ba320e16e2221f6832b432052c4da43.zh.jpg) | ![overfit](../../../../translated_images/overfit2.131f5800ae10ca5e41d12a411f5f705d9ee38b1b10916f284b787028dd55cc1c.zh.jpg)
-------------------------|--------------------------
**线性模型，2 个参数** | **非线性模型，7 个参数**
训练误差 = 5.3 | 训练误差 = 0
验证误差 = 5.1 | 验证误差 = 20

* 在左侧，我们看到一个良好的直线近似。由于参数数量适当，模型正确理解了点分布背后的概念。
* 在右侧，模型过于强大。由于我们只有 5 个点，而模型有 7 个参数，因此它可以调整到通过所有点，使得训练误差为 0。然而，这阻止了模型理解数据背后的正确模式，因此验证误差非常高。

在模型的丰富性（参数数量）和训练样本数量之间取得正确平衡是非常重要的。

## 为什么会出现过拟合

* 训练数据不足
* 模型过于强大
* 输入数据中噪声过多

## 如何检测过拟合

正如您从上面的图表中看到的，过拟合可以通过非常低的训练误差和高的验证误差来检测。通常在训练过程中，我们会看到训练误差和验证误差都开始下降，然后在某个时刻，验证误差可能会停止下降并开始上升。这将是过拟合的迹象，并且是我们应该在此时停止训练（或至少对模型进行快照）的指示。

![overfitting](../../../../translated_images/Overfitting.408ad91cd90b4371d0a81f4287e1409c359751adeb1ae450332af50e84f08c3e.zh.png)

## 如何防止过拟合

如果您发现过拟合发生，您可以采取以下措施之一：

* 增加训练数据量
* 降低模型的复杂性
* 使用一些 [正则化技术](../../4-ComputerVision/08-TransferLearning/TrainingTricks.md)，例如 [Dropout](../../4-ComputerVision/08-TransferLearning/TrainingTricks.md#Dropout)，我们稍后将讨论。

## 过拟合与偏差-方差权衡

过拟合实际上是统计学中一个更一般性问题的情况，称为 [偏差-方差权衡](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff)。如果我们考虑模型中可能的误差来源，我们可以看到两种类型的误差：

* **偏差误差** 是由于我们的算法无法正确捕捉训练数据之间的关系造成的。这可能是因为我们的模型不够强大（**欠拟合**）。
* **方差误差** 是由于模型近似输入数据中的噪声而非有意义的关系（**过拟合**）造成的。

在训练过程中，偏差误差会减少（因为我们的模型学习近似数据），而方差误差会增加。重要的是在检测到过拟合时手动停止训练，或者通过引入正则化自动停止训练，以防止过拟合。

## 结论

在本课中，您了解了两个最流行的 AI 框架 TensorFlow 和 PyTorch 的各种 API 之间的差异。此外，您还学习了一个非常重要的话题，过拟合。

## 🚀 挑战

在随附的笔记本中，您会在底部找到“任务”；请逐步完成笔记本并完成任务。

## [课后测验](https://red-field-0a6ddfd03.1.azurestaticapps.net/quiz/205)

## 复习与自学

对以下主题进行一些研究：

- TensorFlow
- PyTorch
- 过拟合

问自己以下问题：

- TensorFlow 和 PyTorch 之间有什么区别？
- 过拟合和欠拟合之间有什么区别？

## [作业](lab/README.md)

在这个实验中，您需要使用 PyTorch 或 TensorFlow 解决两个分类问题，使用单层和多层全连接网络。

* [说明](lab/README.md)
* [笔记本](../../../../../lessons/3-NeuralNetworks/05-Frameworks/lab/LabFrameworks.ipynb)

**免责声明**：
本文件使用机器翻译服务进行翻译。尽管我们努力追求准确性，但请注意，自动翻译可能包含错误或不准确之处。原始文件的母语版本应被视为权威来源。对于关键信息，建议使用专业的人类翻译。我们对因使用此翻译而产生的任何误解或错误解释不承担责任。