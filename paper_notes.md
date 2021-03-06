## 1、20-Joint Learning of Vessel Segmentation and Artery/Vein Classification with Post-processing 
**任务**：视网膜血管分割 和 动静脉判别

**方法**：  
上文已经做了分割，且计算了每个像素的动静脉分类概率值。

实验发现分类结果中常出现两种错误，  
1、血管中间。这是由分割网络所致  
2、在cross points和branch points附近。  
这两种情况分别用intra label修正和inter prediction修正。

*段内：intra segment label propagation方法：*  
1、骨架细化，检测关键点（交叉点和端点），相邻关键点之间的片段称为一个segment  
2、每个segments的类别用其中每个像素的（动脉概率-静脉概率）求和，用这个结果判别该片段属于动静脉的概率。

*分叉点：Inter-segment Prediction Propagation方法：*
通常在分类概率值低的地方容易出现分类错误，这些错误可以通过相邻的高置信度的影响来纠正。当形状、方向、位置相似时，有很高概率他们属于同一根血管，此时他们互相的影响应该很强烈。

基于这种观察，设置了如下的概率传播规则。
<img src="./source/propagation.png" width = "600" alt="传播规则" align=center />

其中A、L、T、D是四个参数，分别表示归一化的：  
A：两血管相对方向  
L：连线和其中一个血管的相对方向  
T：平均粗细相似度，越接近T越靠近1  
D：两个点的距离反比

**评价指标**：
    没有针对拓扑结构设计指标，只比较了连接前后的 分割图、中心线（accuracy、F1）、中心线偏差>2px（accuracy、F1）


## 2、19-Dense Dilated Network with Probability Regularized Walk for Vessel Detection 
这篇文章既有分割也有连接，可以借鉴一些写法。
骨架修正的相关工作只列了三篇【57】【58】【59】

**目标**：视网膜血管分割

**方法**   
CNN分割得到概率图和二值图后，为了连接断裂，模拟随机游走的过程（不是使用随机游走算法）。  
1、先将二值图的连通域标号，最大的连通域记为i1。分别计算其他连通域与i1距离<L的点A、B。记AB中点M，以M为中心的方行区域为连接断裂的ROI（fracture region）。  
2、确定方向。以小连通域的点集合拟合二次曲线，与i1的交点为C，  
3、以ROI内的小连通域的点为初始种子点，计算当前种子点的八邻域的概率值（与C距离的反比、当前位置分割概率值 二者加权求和（权值为0.2和0.8效果最好）），取最大概率值的邻域像素点为next step。  
4、直到8邻域内的点的概率值都很小，终止。  
<img src="./source/2-6.png" width = "600" alt="模拟随机游走算法" align=center />

**指标**：  
1、比较了连接前后的分割指标 Sensitivity (Sen) and Accuracy (Acc)、time consumption (Time) ，这些指标也用于选择距离阈值L。  
2、比较不同的连接方法（conventional walk (CRW)）：连接后的 1-precision（越小越好）。选择超参α也是通过这个指标。  

[注]：敏感度（Sensitivity）：true positive rate =TP/ (TP+ FN)，描述识别出的所有正例占所有正例的比例  
特异度（specificity）：true negative rate= TN / (FP + TN)，描述识别出的负例占所有负例的比例


**表达**：
连通性：connectivity；
血管碎片：fracture of the vessels；
新生血管疾病：neovascular diseases；
<img src="./source/2-3.png" width = "800" alt="连接评价指标ERR+超参选择" align=center />
<img src="./source/2-4.png" width = "800" alt="超参选择+ablation study" align=center />
<img src="./source/2-5.png" width = "300" alt="结果比较" align=center />


## 3、13-TRANS An Automatic Graph-Based Approach for Artery/Vein Classification in Retinal Images 
方法较简单，定义了多种错误类别。见阅读笔记1.  
**评价指标**：  
连接前后的 分割图、中心线、断裂ROI的分割图、断裂ROI的中心线，  
还有连接后不同粗细的血管的四个指标比较。表格上看越粗的血管指标越好，没有详细分析。

## 4、21-Topology-Aware Retinal Artery–Vein Classification via Deep Vascular Connectivity Prediction
**任务**：血管分割 和 动静脉判别

**基于的假设/观察：**  
1、细血管的形状比外观更明显；  
2、【14】中提到交叉点处的粗细、方向变化是可以学习到的；  
3、分叉时同一根血管的粗细、方向不会突变  

相关文献中和拓扑有关的：【9，11，10】，这些方法都定义了人工的特征。  
而本文提出的方法不需要自己定义。

**方法**：  
网络b：分割血管+动静脉分类，用分割图refine分类结果得AV-result1；  
网络c：三个模块：预测+方向+连接分类（把血管对的连接看成分类，输出分类概率值）。  
其中表示粗细和方向的特征图用来作为连接分类器的监督。  

1、先给AV-result1划分segments，把junction point挖出来以分开segments。例如有四个segments连到一起，则可以有C42个连接对。  
2、然后每个连接对的特征送去分类器（图c右侧），逐像素判断是否连接，逐像素概率和的均值作为血管对的连接概率。  
3、最后从细血管开始trace，遇到正连接对则置label一致。  

<img src="./source/4-1.png" width = "500" alt="方法流程" align=center /><img src="./source/4-2.png" width = "500" alt="网络" align=center />


**如何修正的？**  连接对的 粗细、方向特征拿去分类。

**数据**：  
粗细GT：对分割GT做距离变换，则粗细为中心线上的值×2。设置为4个值[1.5, 3, 5, 7]  
角度GT：中心线上每五个点作为一个segment，其两端连线和x轴所成角度称为这个片段的角度。[−p/2, p/2)均分为6个值，加上0，作为七个方向标签  
连接GT：专家标注。contribution里说会放出来，但是GitHub是空的。  

**指标**：  
没有做修正的ablation study

