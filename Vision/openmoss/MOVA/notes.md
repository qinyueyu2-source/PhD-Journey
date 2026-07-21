1.latent既然已经压缩了，为什么还能恢复？

因为：

Encoder和Decoder是一起训练的。

例如：

原图

↓

Encoder

↓

latent

↓

Decoder

↓

重建图片

训练的时候：

AI不断比较：

原图

和

重建图

是不是一样。

如果不像：

就修改Encoder和Decoder。

训练几百万次以后。

Encoder越来越会压缩。

Decoder越来越会恢复。

最后：

形成一对配合默契的搭档。

所以：

Encoder

↓

latent

↓

Decoder

就像：

中文

↓

翻译成英文

↓

再翻译回中文

如果两个翻译都很厉害。

最后：

意思基本不会变。
