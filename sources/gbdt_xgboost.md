# Machine-Learing：GBDT与XGBoost

Oct 14, 2019[machine learning](/categories/machine%20learning)

最近在Kaggle上看到了GBDT和XGBoost模型的使用，之前只知道这是与决策树有关的集成学习算法，之后又在知乎看到关于这个的回答，感觉这个在机器学习里面还是很重要的方法，因此还是很有必要花时间去弄懂的。

## 1.集成学习[](2019/10/14/gbdt_xgboost#1-集成学习)

西瓜书里面提到：根据个体生成器的生成方式，目前的集成学习方法可以分为两大类：  
1.个体学习器之间存在强依赖关系、必须串行生成的序列化方法，如：Boosting；  
2.个体学习器之间不存在强依赖关系、可以同时生成的并行化方法，如：Bagging和随机森林；  
其实除了Boosting和Bagging外，集成学习还包括一个Stacking的方法，Stacking是通过一个元分类器或元回归器来整合多个分类模型或回归模型的集成学习技术；  
在Boosting中，代表算法是Adaboost、**GBDT** （Grdient Boosting Decision Tree）、**XGBoost**
；而Bagging的代表性算法就是Random Forest。

## 2.提升树Boosting Decision Tree[](2019/10/14/gbdt_xgboost#2-提升树Boosting-
Decision-Tree)

提升树是以决策树为基学习器的提升方法，可以是分类树也可以是回归树，因此通常使用CART树作为基学习器；  
当基学习器为分类树，输出值是不连续的，可以把损失函数换为指数损失函数，这就与Adaboost很像，还可以用类似于逻辑回归的对数似然损失函数；在以回归树作为基学习器的提升树算法中，回归树将输入空间划分为M个区域：
R_1 , R_2 ,…, R_M ，每个区域都有一个输出值 c_m ，其最优值 \hat{c}_ m 为该区域上所有输出 y_i 的均值，即：

\hat{c}_ m=ave(y_i|x_i \in R_m)

因此，回归树模型可以表示为：

T(x)=\sum_{m=1}^M{\hat{c}_ mI(x \in R_m)}

使用回归树作为基学习器的提升树模型可以表示为决策树的加法模型：

f_M(x)=\sum_{m=1}^{M}{T(x\ ;\Theta_m)}

其中， T(x;\Theta_m) 表示一颗决策树， f_M(x) 表示所有决策树的组合， M 为树的数量；  
提升树的算法采用的是前向分步算法，第m步中的提升树模型为：

f_m(x)=f_{m-1}(x)+T(x_i\ ;\Theta_m)

即当前的提升树模型是由上一步的提升树模型加上当前的回归树得到的，而当前的回归树是通过最小化损失函数 L(\cdot) 得到的：

\hat{\Theta}_
m=arg\,\min_{\Theta_m}\sum_{i=1}^{N}L(y_i,f_{m-1}(x_i)+T(x_i,\Theta_m))

一般在回归问题里使用的损失函数为平方误差损失函数，分类问题里使用指数损失函数。  
当在回归问题里使用平方损失函数：

L(y,f(x))=(y-f(x))^2

则对于提升树模型可以得到：

L(y,f_{m-1}(x)+T(x;\Theta_m))=[y-f_{m-1}(x)-T(x;\Theta_m)]^2=[r-T(x;\Theta_m)]^2

其中， r=y-f_{m-1}(x) 表示残差，因此，每一回归树都是学习上一颗树的结果和残差，最终，提升树即是整个迭代过程生成的回归树的累加；

**提升树的算法描述为：**  
输入：训练数据集 T=((x_1,y_1),(x_2,y_2),\dots,(x_N,y_N)),\ x_i\in X\in R^n,\ y_i \in Y
\in R ;  
输出：提升树 f_M(x) ;  
(1) 初始化 f_0(x)=0 ;  
(2) 对 m=1,2,...,M :  
(a) 计算残差： r_{mi}=y_i-f_{m-1}(x_i),\ i=1,2,...,N  
(b) 拟合残差 r_{mi} 学习一个回归树，得到 T(x\ ;\Theta_m)  
(c) 更新 f_m(x)=f_{m-1}+T(x\ ;\Theta_m)  
(3) 得到回归问题的提升树： f_M(x)=\sum_{m=1}^{M}{T(x\ ;\Theta_m)}

> 《统计学习方法》第149页有一个关于如何构建提升树的很好的例题

在一个简单的回归问题中使用提升树算法，先确定一个损失函数，比如取平方损失函数，然后优化以下问题：

m(s)=\min_s[\min_{c_1}\sum_{x_i\in R_1}(y_i-c_1)^2+\min_{c_2}\sum_{x_i\in
R_2}(y_i-c_2)^2]

(损失函数并不等同于目标函数，最小化损失是为了提高精度，而目标函数里可能会考虑到损失和模型的复杂度)  
上式表示求得一个划分点 s 将输入空间划分为 R_1 和 R_2 ，一般取 c_1 和 c_2 为两个区域内的平均值:

c_1=\dfrac{1}{N_1}\sum_{x_i\in R_1}y_i\ ,\quad c_2=\dfrac{1}{N_2}\sum_{x_i\in
R_2}y_i

其中 N_1,\ N_2 分别表示 R_1,\ R_2 中的样本数量；要使得 R_1, R_2 区域内的每个值分别与 c_1,c_2
的平方差之和最小，这样的一个划分点就是当前的最优划分点，然后求得在该划分点下的训练数据的残差，用平方差损失函数求当前的损失（也就是残差的平方和），接下来每一步都是对上一步的残差进行学习，因此损失（残差平方和）是不断下降的。

## 3.梯度提升决策树GBDT[](2019/10/14/gbdt_xgboost#3-梯度提升决策树GBDT)

一般是认为是使用平方误差函数或指数误差函数时，提升树算法中每一步的优化是比较简单的，但是对于一般的损失函数就没那么容易，我的理解是，在使用平方误差损失函数时，误差函数的展开为当前的残差值与对残差学习的决策树的结果差值的平方和，即其中包含残差，因此每一步都是对残差进行学习，如果换成另外一个损失函数，如果每次再对残差进行学习就行不通了，因为求损失函数值的时候就不会再用到残差，因此，再梯度提升方法中，将残差换为损失函数的负梯度：

-\left[\dfrac{\partial L(y,f(x_i))}{\partial f(x_i)}\right]_ {f(x)=f_{m-1}(x)}

（有个问题还需要再看看，为什么可以用负梯度代替残差，梯度下降再看看）  
可以发现如果误差函数取为平方差损失，那么它的负梯度就是残差，也就是说，提升树就是损失函数取平方损失的GBDT，但是使用平方损失的缺点在于对异常值很敏感，可以用huber损失函数来替换：

L(y,f(x))=\begin{cases} \tfrac{1}{2}(y-f(x))^2 \quad\quad\ \ |y-f(x)|\le\delta
\\\ \delta(|y-f(x)|-\tfrac{\delta}{2}) \ |y-f(x)|>\delta \end{cases}

**梯度提升树的算法描述为：**  
输入：训练数据集 T=\\{(x_1,y_1),(x_2y_2),...,(x_N,y_N)\\}, \ x_i\in X \subseteq R^n,\
y_i \in y\subseteq R ;损失函数 L(y,f(x)) ;  
输出：梯度提升树 \hat{f}_ x ;  
(1) 初始化

f_0(x)=arg \min_{c}\sum_{i=1}^{N}L(y_i,c)

_（初始化的c应该是0）_  
(2) 对m=1,2,…,M  
(a) 对i=1,2,…,N，计算

r_{mi}=-\left[\dfrac{\partial L(y_i,f(x_i))}{\partial f(x_i)}\right]_
{f(x)=f_{m-1}(x)}

_（在提升树里是求残差）_  
(b) 对 r_{mi} 拟合一棵回归树，得到第m棵树的区域划分；  
(c) 对 j=1,2,...,J ，计算每个区域的输出值：

c_{mj}=arg \min_{c} \sum_{x_i \in R_{mj}}L(y_i,f_{m-1}(x_i)+c)

_（在提升树里是求每个区域值的平均，这里是替换了损失函数）_  
(d) 更新梯度提升树： f_m(x)=f_{m-1}+\sum_{j=1}^{J}c_{mj}I(x\in R_{mj})  
(3) 得到梯度提升树：

\hat{f}(x)=f_M(x)=\sum_{m=1}^{M}\sum_{j=1}^{J}c_{mj}I(x\in R_{mj})

以上的算法流程是：先使用上一步得到的提升树求负梯度，然后根据负梯度学习一颗树，学习的过程是先找到最优的值空间的区域划分，然后求得每个区域的输出值。

## 4.XGBoost[](2019/10/14/gbdt_xgboost#4-XGBoost)

与提升树相同，XGBoost可以表示为决策树的加法模型：

f_M(x)=\sum_{m=1}^{M}{T(x\ ;\Theta_m)}

定义模型的目标函数为：

Obj=\sum_{i}^{n}L(y_i,\hat{y}_ i)+\sum_{k=1}^{K}\Omega(f_k)

目标函数前面部分表示损失，后部分表示决策树的复杂度，是一个正则化项，目标函数的优化也和前面的GBDT一样采用前向分布算法，即当前的模型由上一步的模型和当前的决策树构成：

\hat{y}_ i^t=\sum_{k=1}^{t}f_k(x_i)=\hat{y}_ i^{t-1}+f_t(x_i)

由此可以将目标函数写为：

Obj^{(t)}=\sum_{i=1}^n{L(y_i,\hat{y}_
i^t)}+\sum_{i=1}^{t}{\Omega(f_t)}=\sum_{i=1}^{n}L\left(y_i,\hat{y}_
i^{t-1}+f_t(x_i)\right)+\Omega(f_t)+constant

上式中，constant表示前面t-1棵树的复杂度，在当前为一个常数值；  
泰勒公式是在局部用一个多项式函数近似代替一个复杂的函数，其表达式为：

f(x)=\sum_{i=0}^{n}{\dfrac{f^{(i)}(x_0)}{i!}(x-x_0)^i}=f(x_0)+f \prime
(x_0)(x-x_0)+\dfrac{f\prime\prime(x_0)}{2!}(x-x_0)^2+...

用损失函数 L(y_i,\hat{y}_ i^{(t)}) 的二阶泰勒展开来近似损失函数，可以将目标函数表示为：

Obj^{(t)}=\sum_{i=1}^{n}\left[L(y_i,\hat{y}_
i^{(t-1)})+g_if_t(x_i)+\dfrac{1}{2}h_if_t^2(x_i)\right]+\Omega(f_t)+constant

其中， L(y_i,\hat{y}_ i^{(t-1)}) 对应泰勒展开式中的 f(x_0) ，因此泰勒展开式中的 x-x_0 有：

x-x_0=\hat{y}_ i^{t}-\hat{y}_ {i}^{(t-1)}=f_t(x_i)

目标函数 Obj 中：

g_i=\partial_{\hat{y}_ i^{(t-1)}}L(y_i,\hat{y}_ i^{(t-1)})  
MATHJAX-SSR-113

表示泰勒展开式中的 f\prime(x_0) 和 f\prime\prime(x_0) ， \hat{y}_ i^{(t-1)}
是上一步的提升树模型的最优输出；  
优化的目的是使目标函数最小化，因此去除常数项，得到目标函数如下：

Obj^{(t)}=\sum_{i=1}^{n}[g_if_i(x_i)+\dfrac{1}{2}h_if_t^2(x_i)]+\Omega(f_i)

接下来定义一棵包含 T 个节点的决策树（CART树）的模型：

f_t(x)=\omega_{q(x)},\quad \omega\in R^T,q:R^d\rightarrow\\{1,2,...,T\\}

其中， q(x) 是将输入映射到一个叶子节点的函数， \omega 是叶子节点值组成的向量，由此可以定义上面的正则化项 \Omega 为：

\Omega(f)=\gamma T+\dfrac{1}{2}\lambda\sum_{j=1}^{T}\omega_j^2

其中， \gamma 和 \lambda 是正则化参数，这个正则化项是为了得到结构简单的树  
由此可以将目标函数改写为：

Obj^{(t)}\approx
\sum_{i=1}^{n}[g_i\omega_{q(x_i)}+\dfrac{1}{2}h_i\omega_{(q(x_i))}^2]+\gamma T
+ \dfrac{1}{2}\lambda \sum_{j=1}^T \omega_j^2=\sum_{j=1}^T[(\sum_{i\in
I_j}g_i)\omega_j+\dfrac{1}{2}(\sum_{i\in I_j}h_i+\lambda)\omega_j^2]+\gamma T

其中， I_j 表示叶子节点 j 中的样本集合，这个转换有点难理解，其实就是求和顺序的变换，再定义：

G_j=\sum_{i\in I_j}g_i\ ,\quad H_j=\sum_{i\in I_j}h_j

由此可以将目标函数简化为：

Obj^{(t)}=\sum_{j=1}^{T}[G_j\omega_j+\dfrac{1}{2}(H_j+\lambda)\omega_j^2]+\gamma
T

求解可以得到叶子节点的最优值和此时的目标函数值：

\omega_j^* =-\dfrac{G_j}{H_j+\lambda}

Obj^* =-\dfrac{1}{2}\sum_{j=1}^{T} \dfrac{G_j^2}{H_j+\lambda}+\gamma T

**单颗决策树的生成过程可以描述为** ：  
1.枚举所有的树结构；  
2.计算每个树结构的对应的目标函数 Obj 的值；  
3.找到目标函数值最小的子节点，生成新的分支，并为每个子节点计算输出值；

实际上树的结构非常多，因此难以穷举，通常情况下采用贪心策略来生成决策树的每个节点：  
1.从深度为0的树开始，对每个叶子节点枚举所有特征；  
2.对于每个特征，线性扫描得到特征的最佳分裂点，记录最大收益；  
3.选择收益最大的特征作为分类特征，该特征的最佳分裂点作为分裂位置

每次分裂的收益怎么计算呢？假设当前的节点为C，分类的左孩子节点为L，右孩子节点为R，则可以定义分裂的收益为：

Gain=Obj(C)-Obj(L)-Obj(R)=\dfrac{1}{2}[\dfrac{G_L^2}{H_L+\lambda}+\dfrac{G_R^2}{H_R+\lambda}-\dfrac{(G_L+G_R)^2}{H_L+H_R+\lambda}]-\gamma

由此可以找到最优的树结构，然后确定每个叶子节点的输出值，就可以确定当前的决策树；  
另外，普通的决策树在生成之后需要剪枝来控制模型的复杂度，但是XGBoost在生成的时候就通过正则化参数来控制了模型的复杂度，因此不需要单独进行剪枝

[GBDT](/tags/GBDT)[XGBoost](/tags/XGBoost)