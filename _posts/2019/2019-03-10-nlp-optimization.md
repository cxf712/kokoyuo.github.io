---
layout: post
title: 深度学习各优化算法介绍
category: nlp
tags: [nlp]
---
# 优化算法总结  
![image](https://note.youdao.com/yws/api/personal/file/44CA30AD0E124CDCBFC5F38F3437D45F?method=download&shareKey=f98c0264762a79a00f3769eb58a33810)

## 梯度下降算法  
### 批量梯度下降(batch gradient descent)
$$
\theta_t =\theta_{t-1} -\eta *\partial_{\theta_{t-1}} J(\theta_{t-1}) 
$$

每次更新时需要针对所有样本进行梯度更新计算,每次更新都会朝着正确的方向进行，所以对于凸函数可以收敛到全局最小值，对于非凸函数可以收敛到局部最小值。  

**缺点：(由计算所有样本引起的)**  
1.计算慢，对大数据不友好  
2.当数据集较大而存在相似样本时，在计算梯度时会出现冗余  
3.不容易投入新数据进行实时更新权重  

### SGD(Stochastic gradient descent)
$$
\theta_t =\theta_{t-1} -\eta *\partial_{\theta_{t-1}} J(\theta_{t-1};x^{i};y^{i})
$$

其中，$\partial_{\theta_{t-1}} J(\theta_t-1)$是目标函数在$\theta_{t-1}$处的梯度。SGD每次更新权重时只对一个样本进行梯度更新计算，计算速度快，没有冗余可以新增样本 

**缺点：**  
SGD更新比较频繁,每次更新可能并不会按照正确的方向进行，因此可以带来优化波动  

**总结：**  
1.BGD 可以收敛到局部极小值，当然 SGD 的震荡可能会跳到更好的局部极小值处。   
2.稍微减小 learning rate时，SGD 和 BGD 的收敛性是一样的。  

### Mini-batch gradient descent 
$$
\theta_t =\theta_{t-1} -\eta *\partial_{\theta_{t-1}} J(\theta_{t-1};x^{i:i+n};y^{i:i+n})
$$

MBGD(Mini-batch gradient descent) 每一次利用一小批样本，即 n 个样本进行计算，这样它可以降低参数更新时的方差，收敛更稳定，另一方面可以充分地利用深度学习库中高度优化的矩阵操作来进行更有效的梯度计算。  

和 SGD 的区别是每一次循环不是作用于每个样本，而是具有 n 个样本的批次与SGD相比降低了收敛波动性，降低了参数更新的方差，更新更加稳定。与BGD相比，提高了每次学习的速度，不用担心内存瓶颈，通过矩阵计算进行高效计算  

超参数设定值:n 一般取值在 50～256

**缺点:**  
1.Mini-batch gradient descent 不能保证很好的收敛性  
2.learning rate 如果选择的太小，收敛速度会很慢，如果太大，loss function 就会在极小值处不停地震荡甚至偏离。  有一种措施是先设定大一点的学习率，当两次迭代之间的变化低于某个阈值后，就减小 learning rate，不过这个阈值的设定需要提前写好，这样的话就不能够适应数据集的特点。此外，这种方法是对所有参数更新时应用同样的 learning rate，如果我们的数据是稀疏的，我们更希望对出现频率低的特征进行大一点的更新。  
3.另外，对于非凸函数，还要避免陷于局部极小值处，或者鞍点处，因为鞍点周围的error 是一样的，所有维度的梯度都接近于0，SGD 很容易被困在这里。  

## momentum  
SGD 在遇到沟壑时容易陷入震荡。为此，可以为其引入动量 Momentum，加速 SGD 在正确方向的下降并抑制震荡。  

momentum即动量模拟物理里动量的概念，积累之前的动量来替代真正的梯度。，它模拟的是物体运动时的惯性，即更新的时候在一定程度上保留之前更新的方向，同时利用当前batch的梯度微调最终的更新方向。这样一来，可以在一定程度上增加稳定性，从而学习地更快，并且还有一定摆脱局部最优的能力：

$$
m_t=\beta m_{t-1}+\partial_{\theta_{t-1}} J(\theta_{t-1})  

\theta_{t}=\theta_{t-1}- \eta m_t 
$$

其中，$\beta$为动量因子，表示要在多大程度上保留原来的更新方向，这个值在0-1之间，在训练开始时，由于梯度可能会很大，所以初始值一般选为0.5；当梯度不那么大时，改为0.9。$m_t$和$m_{t-1}$是现在和上一时刻的更新方向。  

下降初期时，使用上一次参数更新，下降方向一致，乘上较大的$\beta$能够进行很好的加速  ;下降中后期时，在局部最小值来回震荡的时候，$gradient\rightarrow0$，$\beta$使得更新幅度增大，跳出陷阱,在梯度改变方向的时候，$\beta$能够减少更新,总而言之，momentum项能够在相关方向加速SGD，抑制振荡，从而加快收敛   

### Nesterov accelerated gradient  
nesterov项在梯度更新时做一个校正，避免前进太快，同时提高灵敏度。  
由之前的momentum公式展开得到：  
$$
\theta_{t}=\theta_{t-1}- \eta \beta m_{t-1}+\eta \partial_{\theta_{t-1}} J(\theta_{t-1})  
$$

可以看出，$m_{t-1}$并没有直接改变当前梯度$m_{t-1}$，所以Nesterov的改进就是让之前的动量直接影响当前的动量。即:  
$$
g_t=\partial_{\theta_{t-1}}J(\theta_{t-1}-\eta \beta m_{t-1}) 

m_t=\beta*m_{t-1}+g_t  

\theta_t=\theta_{t-1}-\eta*m_t  
$$

所以，加上nesterov项后，梯度在大的跳跃后，进行计算对当前梯度进行校正。如下图：  
![image](https://note.youdao.com/yws/api/personal/file/E013243BCD2D42D59497BFA4F443F9E6?method=download&shareKey=ef60adcd07487eeef7b7f2cea35481c6)  
nesterov项首先在之前加速的梯度方向进行一个大的跳跃(棕色向量)，计算梯度然后进行校正(绿色梯向量)  
其实，momentum项和nesterov项都是为了使梯度更新更加灵活，对不同情况有针对性。  

##自适应算法
### adagrad  
它能够对每个参数自适应不同的学习速率，对稀疏特征，得到大的学习更新，对非稀疏特征，得到较小的学习更新，因此该优化算法适合处理稀疏特征数据   

$$
g_t=\partial_{\theta_{t-1}}J(\theta_{t-1}) 

\theta_t =\theta_{t-1} -\eta *g_t/\sqrt{\sum_{i=1}^tg_t^2+\epsilon}
$$

 其中,$ \epsilon $ 保证分母非0  

 与SGD的核心区别在于计算更新步长时，增加了分母：梯度平方累积和的平方根。此项能够累积各个参数的历史梯度平方。频繁更新的梯度，则累积的分母项逐渐偏大，那么更新的步长(stepsize)相对就会变小，而稀疏的梯度，则导致累积的分母项中对应值比较小，那么更新的步长则相对比较大。  

AdaGrad能够自动为不同参数适应不同的学习率（平方根的分母项相当于对学习率$\eta$进行了自动调整，然后再乘以本次梯度），大多数的框架实现采用默认学习率$\eta=0.01$即可完成比较好的收敛。  

**优势**：  
在数据分布稀疏的场景，能更好利用稀疏梯度的信息，比标准的SGD算法更有效地收敛。  

**缺点：**  
仍依赖于人工设置一个全局学习率，设置过大的话，会使$\frac{1}{\sqrt{\sum_{i=1}^tg_t^2+\epsilon}}$过于敏感，对梯度的调节太大，由于分母项对梯度平方不断累积，随之时间步地增加，分母项越来越大，最终导致学习率收缩到太小无法进行有效更新。并且如果初始梯度很大的话，会导致整个训练过程的学习率一直很小，从而导致学习时间变长。    

### adadelta  
Adadelta是对Adagrad的扩展，也是对学习率进行自适应约束，但是进行了计算上的简化,它会累加之前所有的梯度平方，而Adadelta只累加固定大小的项，并且也不直接存储这些项，仅仅是近似计算对应的平均值   

相对于adagrad，改进点如下：  
1.将累计梯度信息从全部历史梯度变为当前时间向前的一个窗口期内的累积。  

$$
v_t=\gamma*v_{t-1}+(1-\gamma)*g_t^2
$$

2.然后将上述$v_t$开方后，作为每次迭代更新后的学习率衰减系数：  

$$
\theta_t =\theta_{t-1} -\eta *g_t/\sqrt{v_t+\epsilon}  

RMS(g_t)=\sqrt{v_t+\epsilon}
$$

该RMSprop方法也为这种更新方法解决了对历史梯度一直累加而导致学习率一直下降的问题，当时还是需要自己选择初始的学习率。  

3.Hessian方法与正确的更新单元  
在SGD和动量法中：  

$$
\Delta{x}\propto g \propto \frac{\partial_f}{\partial_x} \propto \frac{1}{x}
$$

$\Delta{x}$可以正比到梯度g问题，再正比到一阶导数。而log一阶导又可正比于$\frac{1}{x}$。   

Hessian矩阵法：使用[Becker&LeCun 1988]的近似方法，让求逆矩阵近似于求对角阵的倒数  

$$
\Delta{x}\propto H^{-1}g \propto \frac{\frac{\partial_f}{\partial_x}}{\frac{\partial_f^2}{\partial_{x^2}}} \propto \frac{\frac{1}{x}}{\frac{1}{x}*\frac{1}{x}} \propto x
$$

$\Delta{x}$可以正比到Hessian逆矩阵$H^{-1}g$问题，再正比到二阶导数。而log二阶导又可正比于x。   

可以看到，一阶方法最终正比于$\frac{1}{x}$，即与参数逆相关：参数逐渐变大的时候，梯度反而成倍缩小。  

而二阶方法最终正比于x，即与参数正相关：参数逐渐变大的时候，梯度不受影响。  

因此，使用Hessian方法进行Correct Units(正确的更新单元)，计算如下：  

$$
\Delta \approx \frac{\frac{\partial_f}{\partial_x}}{\frac{\partial_f^2}{\partial_{x^2}}} \Rightarrow \frac{1}{\frac{\partial_f^2}{\partial_{x^2}}}=\frac{\Delta{x}}{\frac{\partial_f}{\partial_x}}
$$

收束变形一下, 然后用RMS来近似：  

$$
\frac{\Delta{x}}{\frac{\partial_f}{\partial_x}} \approx -\frac{RMS(\Delta{x_{t-1}})}{RMS(g_t)} \Rightarrow \Delta{x}=-\frac{RMS(\Delta{x_{t-1}})}{RMS(g_t)}*g_t
$$

其中，使用$RMS(\Delta{x_{t-1}})$而不是$RMS(\Delta{x_t})$，是因为此时$RMS(\Delta{x_t})$还没计算出来  
最后，$x_t+1=x_t+\Delta{x}$,可以看出Adadelta已经不用依赖于全局学习率了。    

训练初中期，加速效果不错，很快  
训练后期，反复在局部最小值附近抖动  

### RMSprop
结合梯度平方的指数移动平均数来调节学习率的变化。能够在不稳定（Non-Stationary）的目标函数情况下进行很好地收敛。RMSprop可以算作Adadelta的一个特例  

$$
g_t=\partial_{\theta_{t-1}}J(\theta_{t-1}) 
$$

计算梯度平方的指数移动平均数（Exponential Moving Average），$\gamma$是遗忘因子（或称为指数衰减率），依据经验，默认设置为0.9。    

$$
v_t=\gamma*v_{t-1} +(1-\gamma)*g_t^2

\theta_t =\theta_{t-1} -\eta *g_t/\sqrt{v_t+\epsilon}
$$
梯度更新时候，与AdaGrad类似，只是更新的梯度平方的期望（指数移动均值），其中$\epsilon=10^-8 $，避免除数为0。默认学习率$\eta=0.001$。   

其实RMSprop依然依赖于全局学习率,RMSprop算是Adagrad的一种发展，和Adadelta的变体，效果趋于二者之间,适合于处理非平稳目标 - 对于RNN效果很好  

**优势：**  
能够克服AdaGrad梯度急剧减小的问题，在很多应用中都展示出优秀的学习率自适应能力。尤其在不稳定(Non-Stationary)的目标函数下，比基本的SGD、Momentum、AdaGrad表现更良好。  

### Adaptive Moment Estimation (Adam)  
Adam优化器，结合AdaGrad和RMSProp两种优化算法的优点。对梯度的一阶矩估计（First Moment Estimation，即梯度的均值）和二阶矩估计（Second Moment Estimation，即梯度的未中心化的方差）进行综合考虑，计算出更新步长,Adam的优点主要在于经过偏置校正后，每一次迭代学习率都有个确定范围，使得参数比较平稳。  

$$
g_t=\partial_{\theta_{t-1}}J(\theta_{t-1}) 
$$

首先，计算梯度的指数移动平均数，$m_0$ 初始化为0。类似于Momentum算法，综合考虑之前时间步的梯度动量。$\beta_1$ 系数为指数衰减率，控制权重分配（动量与当前梯度），通常取接近于1的值。默认为0.9  

$$
m_t=\beta_1*m_{t-1} +(1-\beta_1)*g_t
$$

其次，计算梯度平方的指数移动平均数，$v_0$初始化为0。类似于RMSProp算法，对梯度平方进行加权均值。$\beta_2$系数为指数衰减率，控制之前的梯度平方的影响情况。默认为0.999  

$$
v_t=\beta_2*v_{t-1}+(1-\beta_2)*g_t^2
$$

由于$m_0$初始化为0，会导致$m_t$偏向于0，尤其在训练初期阶段衰减率非常小（即β接近于 1）。所以，需要对梯度均值$m_t$进行偏差纠正，降低偏差对训练初期的影响。  

$$
\hat{m_t}=m_t/(1-\beta_1^t)
$$

与$m_0$类似，因为$v_0$初始化为0导致训练初始阶段$v_t$偏向0，对其进行纠正。  

$$
\hat{v_t}=v_t/(1-\beta_2^t)
$$

更新参数，初始的学习率$\alpha$乘以梯度均值与梯度方差的平方根之比。    

$$
\theta_t=\theta_{t-1}-\eta*\hat{m_t}/\sqrt{\hat{v_t}+\epsilon}
$$

由表达式可以看出，对更新的步长计算，能够从梯度均值及梯度平方两个角度进行自适应地调节，而不是直接由当前梯度决定。  

算法的效率可以通过改变计算顺序而得到提升，将最后三个公式替代为以下两个：  

$$
\eta_t=\eta*\sqrt{1-\beta_2^t}/\sqrt{1-\beta_1^t}  

\theta_t=\theta_{t-1}-\eta_t*m_t/\sqrt{v_t+\hat{\epsilon}}
$$

**优势：**
1. 实现简单，计算高效，对内存需求少  
2. 参数的更新不受梯度的伸缩变换影响  
3. 超参数具有很好的解释性，且通常无需调整或仅需很少的微调  
4. 更新的步长能够被限制在大致的范围内（初始学习率）  
5. 能自然地实现步长退火过程（自动调整学习率）  
6. 很适合应用于大规模的数据及参数的场景   
7. 适用于不稳定目标函数  
8. 适用于梯度稀疏或梯度存在很大噪声的问题  

**总结：**  
如果你的数据特征是稀疏的，那么你较好使用自适应学习速率SGD优化方法(Adagrad、Adadelta、RMSprop与Adam)，因为你不需要在迭代过程中对学习速率进行人工调整。  

虽然 Adam 算法在实践中要比 RMSProp 更加优秀，不过在很多领域里（如计算机视觉的对象识别、NLP中的机器翻译、语法成分分析等）的最佳成果仍然是使用带动量（Momentum）的SGD来获取到的。通常推荐在深度学习模型中使用 Adam 算法或 SGD+Nesterov 动量法。

**参考资料**  
1.[https://www.cnblogs.com/qniguoym/p/8058186.html](https://www.cnblogs.com/qniguoym/p/8058186.html)  
2.[http://www.dataguru.cn/article-10174-1.html](http://www.dataguru.cn/article-10174-1.html)  
3.[https://zhuanlan.zhihu.com/p/22461594](https://zhuanlan.zhihu.com/p/22461594)  
4.[https://zhuanlan.zhihu.com/p/22252270](https://zhuanlan.zhihu.com/p/22252270)  
5.[https://blog.csdn.net/qq_35860352/article/details/80772142](https://blog.csdn.net/qq_35860352/article/details/80772142)  
6.[https://www.jianshu.com/p/aebcaf8af76e](https://www.jianshu.com/p/aebcaf8af76e)  
7.AdaDetlad论文:[https://arxiv.org/abs/1212.5701](https://arxiv.org/abs/1212.5701)  
8.Adam论文:[https://arxiv.org/abs/1412.6980v8](https://arxiv.org/abs/1412.6980v8)  