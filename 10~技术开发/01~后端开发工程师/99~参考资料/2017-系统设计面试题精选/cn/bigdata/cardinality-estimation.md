如何计算数据流中不同元素的个数？例如，独立访客(Unique Visitor，简称 UV)统计。这个问题称为基数估计(Cardinality Estimation)，也是一个很经典的题目。

### 方案 1: HashSet

首先最容易想到的办法是用 HashSet，每来一个元素，就往里面塞，HashSet 的大小就所求答案。但是在大数据的场景下，HashSet 在单机内存中存不下。

### 方案 2: bitmap

HashSet 耗内存主要是由于存储了元素的真实值，可不可以不存储元素本身呢？bitmap 就是这样一个方案，假设已经知道不同元素的个数的上限，即基数的最大值，设为 N，则开一个长度为 N 的 bit 数组，地位跟 HashSet 一样。每个元素与 bit 数组的某一位一一对应，该位为 1，表示此元素在集合中，为 0 表示不在集合中。那么 bitmap 中 1 的个数就是所求答案。

这个方案的缺点是，bitmap 的长度与实际的基数无关，而是与基数的上限有关。假如要计算上限为 1 亿的基数，则需要 12.5MB 的 bitmap，十个网站就需要 125M。关键在于，这个内存使用与集合元素数量无关，即使一个网站仅仅有一个 1UV，也要为其分配 12.5MB 内存。该算法的空间复杂度是$$O(N_{max})$$。

实际上目前还没有发现在大数据场景中准确计算基数的高效算法，因此在不追求绝对准确的情况下，使用近似算法算是一个不错的解决方案。

### 方案 3: Linear Counting

Linear Counting 的基本思路是：

- 选择一个哈希函数 h，其结果服从均匀分布
- 开一个长度为 m 的 bitmap，均初始化为 0(m 设为多大后面有讨论)
- 数据流每来一个元素，计算其哈希值并对 m 取模，然后将该位置为 1
- 查询时，设 bitmap 中还有 u 个 bit 为 0，则不同元素的总数近似为 $$-m\log\dfrac{u}{m}$$

在使用 Linear Counting 算法时，主要需要考虑的是 bitmap 长度`m`。m 主要由两个因素决定，基数大小以及容许的误差。假设基数大约为 n，允许的误差为 ϵ，则 m 需要满足如下约束，

$$m > \dfrac{\epsilon^t-t-1}{(\epsilon t)^2}$$, 其中 $$t=\dfrac{n}{m}$$

精度越高，需要的 m 越大。

Linear Counting 与方案 1 中的 bitmap 很类似，只是改善了 bitmap 的内存占用，但并没有完全解决，例如一个网站只有一个 UV，依然要为其分配 m 位的 bitmap。该算法的空间复杂度与方案 2 一样，依然是$$O(N_{max})$$。

### 方案 4: LogLog Counting

LogLog Counting 的算法流程：

1. 均匀随机化。选取一个哈希函数 h 应用于所有元素，然后对哈希后的值进行基数估计。哈希函数 h 必须满足如下条件，

   1. 哈希碰撞可以忽略不计。哈希函数 h 要尽可能的减少冲突
   1. h 的结果是均匀分布的。也就是说无论原始数据的分布如何，其哈希后的结果几乎服从均匀分布（完全服从均匀分布是不可能的，D. Knuth 已经证明不可能通过一个哈希函数将一组不服从均匀分布的数据映射为绝对均匀分布，但是很多哈希函数可以生成几乎服从均匀分布的结果，这里我们忽略这种理论上的差异，认为哈希结果就是服从均匀分布）。
   1. 哈希后的结果是固定长度的

1. 对于元素计算出哈希值，由于每个哈希值是等长的，令长度为 L
1. 对每个哈希值，从高到低找到第一个 1 出现的位置，取这个位置的最大值，设为 p，则基数约等于$$2^p$$

如果直接使用上面的单一估计量进行基数估计会由于偶然性而存在较大误差。因此，LLC 采用了分桶平均的思想来降低误差。具体来说，就是将哈希空间平均分成 m 份，每份称之为一个桶（bucket）。对于每一个元素，其哈希值的前 k 比特作为桶编号，其中$$2^k=m$$，而后 L-k 个比特作为真正用于基数估计的比特串。桶编号相同的元素被分配到同一个桶，在进行基数估计时，首先计算每个桶内最大的第一个 1 的位置，设为`p[i]`，然后对这 m 个值取平均后再进行估计，即基数的估计值为$$2^{\frac{1}{m}\Sigma_{i=0}^{m-1} p[i]}$$。这相当于做多次实验然后去平均值，可以有效降低因偶然因素带来的误差。

LogLog Counting 的空间复杂度仅有$$O(\log_2(\log_2(N_{max})))$$，内存占用极少，这是它的优点。不过 LLC 也有自己的缺点，当基数不是很大的时候，误差比较大。

关于该算法的数学证明，请阅读原始论文和参考资料里的链接，这里不再赘述。

### 方案 5: HyperLogLog Counting

HyperLogLog Counting（以下简称 HLLC）的基本思想是在 LLC 的基础上做改进，

- 第 1 个改进是使用调和平均数替代几何平均数，调和平均数可以有效抵抗离群值的扰。注意 LLC 是对各个桶取算术平均数，而算术平均数最终被应用到 2 的指数上，所以总体来看 LLC 取的是几何平均数。由于几何平均数对于离群值（例如 0）特别敏感，因此当存在离群值时，LLC 的偏差就会很大，这也从另一个角度解释了为什么基数 n 不太大时 LLC 的效果不太好。这是因为 n 较小时，可能存在较多空桶，这些特殊的离群值干扰了几何平均数的稳定性。使用调和平均数后，估计公式变为 $$\hat{n}=\frac{\alpha_mm^2}{\Sigma_{i=0}^{m-1} p[i]}$$，其中$$\alpha_m=(m\int_0^{\infty}(log_2(\frac{2+u}{1+u}))^mdu)^{-1}$$
- 第 2 个改进是加入了分段偏差修正。具体来说，设 e 为基数的估计值，

  - 当 $$e \leq \frac{5}{2}m$$时，使用 Linear Counting
  - 当 $$\frac{5}{2}m<e\leq \frac{1}{30}2^{32}$$时，使用 HyperLogLog Counting
  - 当 $$e>\frac{1}{30}2^{32}$$时，修改估计公式为$$\hat{n}=-2^{32}\log(1-e/2^{32})$$

关于分段偏差修正的效果分析也可以在原论文中找到。

### 参考资料

- [解读 Cardinality Estimation 算法（第一部分：基本概念）](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-i.html)
- [解读 Cardinality Estimation 算法（第二部分：Linear Counting）](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-ii.html)
- [解读 Cardinality Estimation 算法（第三部分：LogLog Counting）](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-iii.html)
- [解读 Cardinality Estimation 算法（第四部分：HyperLogLog Counting 及 Adaptive Counting）](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-iv.html)
