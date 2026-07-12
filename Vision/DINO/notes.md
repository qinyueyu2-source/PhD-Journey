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

这就是local-to-global correspondence。
