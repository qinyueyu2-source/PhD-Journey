1.为什么 Teacher 继承自 Student，却还能监督 Student？

Teacher 并不会通过反向传播学习，它只是 Student 参数的指数滑动平均（EMA），因此可以看作是 Student 历史参数的平滑版本。由于 Teacher 更新速度远慢于 Student，所以它能够提供一个更加稳定（stable）的目标表示（target representation）。

DINO 中没有任何类别标签，Teacher 一开始也不知道什么是狗、什么是猫。训练时，Teacher 只接收 Global View，Student 接收 Global View 和 Local View，并通过 Cross Entropy Loss 让 Student 的 representation 去匹配 Teacher 的 representation。由于 Student 经常只能看到局部（例如狗耳朵、狗鼻子），为了与看到整只狗的 Teacher 保持一致，它只能逐渐学会将属于同一物体的不同局部映射到相同或相近的表示空间（representation space）。因此，"狗耳朵""狗鼻子""狗身体"等不同局部会逐渐对应到同一个语义表示，而这种语义能力并不是标签直接教会模型的，而是在保持不同视图表示一致性的训练目标下**自然涌现（emerge）**出来的。

2.DINO 通过约束同一物体不同视图之间的表示一致性，学习以对象为中心（object-centric）的表示。

3.刚开始：

Teacher

整只狗

↓

EmbeddingA

Student：

狗耳朵

↓

EmbeddingB

因为输入不同：

EmbeddingA ≠ EmbeddingB

于是Loss：

要求：

EmbeddingB

↓

越来越接近

EmbeddingA

注意！

不是让网络输出一样的数。

而是：

让不同视角（Global / Local）得到一致的语义表示。

再往后一点

训练几万步以后。

Teacher：

整只狗

↓

"狗"的表示

Student：

狗耳朵

↓

也输出接近"狗"的表示

是不是说明：

Student已经学会：

狗耳朵

↓

狗

4.刚开始训练时，Teacher 并不是一个已经学会识别世界的模型，它只是 Student 参数的指数滑动平均（EMA），因此也不知道什么是狗、什么是猫。Student 学习 Teacher，并不是因为 Teacher "更正确"，而是因为 Teacher 的表示更加稳定（stable）。在 DINO 中，Teacher 只接收 Global View，因此能够得到更完整、更稳定的图像表示；而 Student 同时接收 Global View 和 Local View，并通过 Cross Entropy Loss 去匹配 Teacher 的输出分布。这样，Student 就必须学会 从局部推断整体（local-to-global correspondence），例如看到狗耳朵也能产生与整只狗相似的表示。随着训练不断进行，Teacher 也会随着 Student 的提升而不断提升，因此语义知识并不是 Teacher 一开始就拥有的，而是在 Teacher-Student 相互促进、不断保持一致的过程中**逐渐涌现（emerge）**出来的。DINO 的监督信号并不是"正确答案"，而是一个稳定的目标（stable target），这也是现代自监督学习的核心思想之一。

现代自监督学习最重要的思想之一就是：模型学习的不是标签，而是世界中的一致性（consistency）；真正驱动模型获得语义能力的，不是人工监督，而是数据本身蕴含的结构。


5.原始图片空间里，一只狗和一只猫可能因为姿态、光照、背景不同而非常复杂，很难用一条直线分开。但如果经过一个优秀的表示学习模型，它们在 embedding 空间中已经自然聚成两团，那么最后只需要一个线性分类器（相当于画一个超平面）就能完成分类。

所以，一个好的表示学习模型，不是让分类器变得更复杂，而是让分类任务变得更简单。

6.Centering and Sharpening
在 DINO 论文中，还有两个不得不提的点便是 Centering 和 Sharpening，这是用于防止模式崩塌的两种有效方式。

在自监督学习中，mode collapse是指网络的学习过程中出现了多样性减少的现象。具体来说，当网络学习到一组特征表示时，往往会出现多个输入数据映射到相同的特征表示的情况，这就是所谓的模式崩塌。这种现象通常是由于网络在优化过程中陷入了局部最优解，只能考虑到一部分数据的特征表示，而忽略了其它数据样本的模式和特征，从而导致了多样性缺失的现象，因此会对模型的鲁棒性产生很大的负面影响。

先来看下 Centering。首先，教师模型的输出经过一个 EMA 的操作，从原始激活值中减去得到一个新的结果.

这个操作的目的是使得激活值有时候是正的（当它们高于平均值时），有时候是负的（当它们低于平均值时）。由于 softmax 函数在处理负数时会给出较小的概率值，而在处理正数时会给出较大的概率值，因此这种操作能够防止任何一个特征占据统治地位，因为平均值会在值的范围中间。

最后，再看看 Sharpening。这种技巧通过在 softmax 函数中加入一个 temperature 参数，来强制让模型将概率分布更加尖锐化。由于小差异会被夸大，这会防止所有激活值都是相同的，因为小的差异也会被放大。这个技巧和中心化操作搭配使用，可以使得激活值不断变化，从而引导学生模型更好地了解哪些特征应该变得更加强大。

7.Teacher 的作用是为 Student 提供一个包含完整对象信息（Global Context）的稳定目标表示。Student 经常只能看到局部区域（如狗耳朵、狗鼻子），为了与 Teacher 的全局表示保持一致，它必须学习局部与整体之间的对应关系（Local-to-Global Correspondence），从而逐渐形成以对象为中心（Object-centric）的表示。如果 Teacher 也只输入 Local View，那么 Student 只需要学习"局部对应局部"，而无需理解这些局部属于哪个完整对象，最终学到的更多是纹理、边缘等低层特征，而不是具有语义的对象表示。


这就是local-to-global correspondence。
