# 基于 flow 的生成模型

生成模型的本质，就是希望用一个我们知道的概率模型来拟合所给的数据样本，也就是说，我们得写出一个带参数 θ 的分布 qθ(x)。然而，我们的神经网络只是“万能函数拟合器”，却不是“万能分布拟合器”，也就是它原则上能拟合任意函数，但不能随意拟合一个概率分布，因为概率分布有“非负”和“归一化”的要求。这样一来，我们能直接写出来的只有离散型的分布，或者是连续型的高斯分布。



当然，从最严格的角度来看，图像应该是一个离散的分布，因为它是由有限个像素组成的，而每个像素的取值也是离散的、有限的，因此可以通过离散分布来描述。**这个思路的成果就是 PixelRNN 一类的模型了，我们称之为“自回归流”**，其特点就是无法并行，所以计算量特别大。所以，我们更希望用连续分布来描述图像。当然，图像只是一个场景，其他场景下我们也有很多连续型的数据，所以连续型的分布的研究是很有必要的。

## **1.各显神通**

所以问题就来了，对于连续型的，我们也就只能写出高斯分布了，而且很多时候为了方便处理，我们只能写出各分量独立的高斯分布，这显然只是众多连续分布中极小的一部分，显然是不够用的。为了解决这个困境，我们通过积分来创造更多的分布：

### $q(x) = \int q(z)q(x|zdz)$ 

这里 q(z) 一般是标准的高斯分布，而 qθ(x|z)=qθ(x|z) 可以选择任意的条件高斯分布或者狄拉克分布。这样的积分形式可以形成很多复杂的分布。理论上来讲，它能拟合任意分布。 



现在分布形式有了，我们需要求出参数 θ，那一般就是最大似然，假设真实数据分布为 p̃(x)，那么我们就需要最大化目标：

### $E_{x- \tilde{p(x)}}\left[   \log q(x)    \right]$

然而 qθ(x) 是积分形式的，能不能算下去很难说。



于是各路大神就“八仙过海，各显神通”了。其中，VAE 和 GAN 在不同方向上避开了这个困难。VAE 没有直接优化目标 (2)，而是优化一个更强的上界，这使得它只能是一个近似模型，无法达到良好的生成效果。GAN 则是通过一个交替训练的方法绕开了这个困难，确实保留了模型的精确性，所以它才能有如此好的生成效果。但不管怎么样，GAN 也不能说处处让人满意了，所以探索别的解决方法是有意义的。 

## **2.直面概率积分**

flow 模型选择了一条“硬路”：**直接把积分算出来。** 

具体来说，flow 模型选择 q(x|z) 为狄拉克分布 δ(x−g(z))，而且 g(z) 必须是可逆的，也就是说

### $x = g(z)   \Longleftrightarrow z = f(x)$ 

要从理论上（数学上）实现可逆，那么要求 z 和 x 的维度一样。假设 f,g 的形式都知道了，那么通过 (1) 算 q(x) 相当于是对 q(z) 做一个积分变换 z=f(x)。即本来是：

### $q(z = \frac{1}{(2\pi)^{D/2}}exp \left(    -\frac{1}{2} ||Z||^2  \right))$ 

的标准高斯分布（D 是 z 的维度），现在要做一个变换 z=f(x)。注意概率密度函数的变量代换并不是简单地将 z 替换为 f(x) 就行了，还多出了一个**“雅可比行列式”的绝对值**，也就是：

### $q(x) = \frac{1}{(2\pi)^{D/2}}exp \left(    -\frac{1}{2} ||Z||^2  \right) \left|   det    \left[    \frac{\partial f}{\part x}   \right]  \right|$ 

这样，对 f 我们就有两个要求：

1. 可逆，并且易于求逆函数（它的逆 g 就是我们希望的生成模型）

 	2. 对应的雅可比行列式容易计算。

这样一来：

### $   \log q(x) = -\frac{D}{2} \log (2\pi) -\frac{1}{2} \|  f(x)\|^2 +  log \left|    det\frac{\part f}{\part x }  \right| $ 

这个优化目标是可以求解的。并且由于 f 容易求逆，因此一旦训练完成，我们就可以随机采样一个 z，然后通过 f 的逆来生成一个样本 $ f^{-1}(z) = g(z)  $ ，这就得到了生成模型。





[细水长flow之NICE：流模型的基本概念与实现](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247490842&idx=1&sn=840d5d8038cd923af827eef497e71404&chksm=96e9c29aa19e4b8c45980b39eb28d80408632c8f9a570c9413748b2b5699260190e0d7b4ed16&scene=21#wechat_redirect)

[RealNVP与Glow：流模型的传承与升华](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247491113&idx=1&sn=4b185eb6985fc747a071d00d37d3ed3c&chksm=96e9c1a9a19e48bfc93e0a1252d18c3ce98e7495bc1d05ae93e6bf0354d737c897dd64ec3188&scene=21#wechat_redirect) 

[细水长flow之f-VAEs：Glow与VAEs的联姻](https://mp.weixin.qq.com/s?__biz=MzIwMTc4ODE0Mw==&mid=2247491695&idx=1&sn=21c5ffecfd6ef87cd4f1f754795d2d63&chksm=96ea3fefa19db6f92fe093e914ac517bd118e80e94ae61b581079023c4d29cedaaa559cb376e&scene=21#wechat_redirect) 