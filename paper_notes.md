## Joint Learning of Vessel Segmentation and Artery/Vein Classification with Post-processing 
**任务**：视网膜血管分割 和 动静脉判别

**方法**：

上文已经做了分割，且计算了每个像素的动静脉分类概率值。

实验发现分类结果中常出现两种错误，
1、血管中间。这是由分割网络所致
2、在cross points和branch points附近。
这两种情况分别用intra label修正和inter prediction修正。

段内：intra segment label propagation方法：
1、骨架细化，检测关键点（交叉点和端点），相邻关键点之间的片段称为一个segment
2、每个segments的类别用其中每个像素的（动脉概率-静脉概率）求和，用这个结果判别该片段属于动静脉的概率。

分叉点：Inter-segment Prediction Propagation方法：
通常在分类概率值低的地方容易出现分类错误，这些错误可以通过相邻的高置信度的影响来纠正。当形状、方向、位置相似时，有很高概率他们属于同一根血管，此时他们互相的影响应该很强烈。
基于这种观察，设置了如下的概率传播规则。
![avatar](./source/propagation.png)

其中A、L、T、D是四个参数，分别表示归一化的：
A：两血管相对方向
L：连线和其中一个血管的相对方向、
T：平均粗细相似度，越接近T越靠近1
D：两个点的距离反比

**评价指标**：
    没有针对拓扑结构设计指标，只评价了分割结果和动静脉分类结果。
