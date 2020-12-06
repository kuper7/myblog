#### 11.28周汇报

##### 本周阅读论文：

1.Single-channel online enhancement of speech corrupted by reverberation and noise

2.Unified approach for underdetermined BSS, VAD, dereverberation and DOA estimation with multichannel factorial HMM

3.https://blog.csdn.net/continueoo/article/details/77893587博客HMM超详细讲解+代码

##### 下周计划：

解决该方法实现代码的剩余内容，继续阅读相关文献，进行代码分析与实验。



## HMM方法去混响

###### 文献来源：

###### 1.Single-channel online enhancement of speech corrupted by reverberation and noise

###### 2.Unified approach for underdetermined BSS, VAD, dereverberation and DOA estimation with multichannel factorial HMM

###### 3.https://blog.csdn.net/continueoo/article/details/77893587博客HMM超详细讲解+代码

大致内容：噪声环境下单麦克风去混响的方法。提出了一种结合频谱增强和概率估计结合的方法来增强含混响和噪声的语音的在线方法（类似基于HMM 的谱增强算法，通过引入语音和噪声的先验统计信息，有效地提高了去混响后的主客观质量）。像频谱增强一样，通过将时频增益应用于含混含噪的语音复STFT系数来执行增强。计算该增益所需的特性的估计被公式化为贝叶斯滤波问题，在贝叶斯滤波问题中，将它们与声学通道的参数一起进行联合估计。声学通道的参数是使用非负一阶自回归滑动平均（ARMA）过程建模的，该过程由混响时间（ Ť60）和直达信号与混响信号能量比（DRR）确定。干净的语音信号是通过隐马尔可夫模型（HMM）建模的，其中每个状态都捕获了多元语音对数功率的可能先验分布的频谱特征。在每个时间帧里，通过大量类似非线性卡尔曼滤波器的更新来测试可能的纯净语音先验分布。该分布使得观测信号功率的最大似然存在，指向语音，混响和噪声平均功率的后验估计。通过六种不同的客观测量方法对模拟数据进行评估，并通过语音识别器的误码率（WER）对实时录音进行评估。进行了听力测试以评估主观混响的降低和整体质量的提高。

详细分析：

#### 1.信号模型

基础的信号模型如下
$$
\begin{equation}
y(n) = \sum \limits _{r=0}^{J-1} \rho (r)s(n-r) + \nu (n).\tag1
\end{equation}
$$
其中，$\rho$为$J-tap RIR$，v(n)是加性噪声。

然后计算观测信号的复STFT系数：
$$
\begin{equation}
Y^{\circ }(l,\tilde{k}) = \sum \limits _{n=0}^{\tilde{K}-1} y(n+lT)w(n)e^{-j\frac{2\pi }{\tilde{K}}n\tilde{k}}\tag2
\end{equation}
$$
其中l为时间帧下标，k为STFT频点, w(n)为时域窗，代码中是汉宁窗，T为帧在时间上的增量。采用梅尔滤波器组计算K个m间隔子带的对数能量。

引入时频增益$\breve{G}(l)$，归一化源信号频谱有下式：
$$
\begin{equation}
\breve{S}(l,k) = \frac{1}{\breve{G}(l)}\sum \limits _{\tilde{k}=0}^{\tilde{K}-1} b_{k,\tilde{k}}|S^{\circ
 }(l,\tilde{k})|^2.\tag3
\end{equation}
$$
这里的$b_{k,\tilde{k}}$其实就是梅尔滤波器组的系数，k为滤波器的编号（代码中有25个梅尔三角滤波器），$ \tilde{k}$为对应频点的幅值。

带入时频增益，观察基础信号的频域表示，有公式：
$$
\begin{equation}
\breve{Y}(l,k) = \sum \limits _{\tau = 0}^{L_{h}-1} \breve{H}(\tau,k)\breve{G}(l-\tau)\,\breve{S}(l-\tau,k) +
 \breve{N}(l,k)\tag4
\end{equation}
$$
这里涉及到：声学通道（RIR）的参数是使用非负一阶自回归滑动平均（ARMA）过程建模的，该过程由混响时间（ Ť60）和直达信号与混响信号能量比（DRR）确定。

具体公式为下面两个：
$$
\begin{equation}
\breve{H}(l,k) = \delta (l) + d_k\, \alpha _k^{l-1} u(l-1)\tag5
\end{equation}
$$

$$
\begin{equation}
\alpha _k^{\frac{T_{60,k}}{T}} = 10^{-6}\tag6
\end{equation}
$$

$$
\begin{equation}
d_k = \frac{1-\alpha _k}{\text{DRR}_k}\tag7
\end{equation}
$$

这里解释一下：

对于（5），这是Polack提出的一个由宽带混响时间T60参数化的指数衰减高斯白噪声的RIR的时域统计模型，来源【H. Kuttruff, Room Acoustics, London, U.K.:Spon Press, 2009.】。

对于（6），在mel频带k中，衰减常数$\alpha_k$与$T_{60,k}$有关。

对于（7），直接路径后的能量下降$d_k$与频率相关的$DRR_k$相关。$DRR_k$的计算是作者通过实验测出来的均值，在不同的频带内，直接信号-混响信号能量比不一样，是测出来的。



接下来对（4）进行分解，有：
$$
\begin{equation}
\breve{Y}(l,k) = \breve{G}(l)\,\breve{S}(l,k) + \breve{R}(l,k) + \breve{N}(l,k).\tag8
\end{equation}
$$
清晰的知道，后三者分别对应，我们需要的去噪去混响信号，混响信号，噪声信号的对数能量

其中，根据（5）（6）（7），有
$$
\begin{equation}
\breve{R}(l,k)=\sum \limits _{\tau =1}^{L_{h}}d_k \breve{G}(l-\tau)\breve{S}(l-\tau,k)\alpha _k^{\tau -1}.\tag9
\end{equation}
$$
为了方便逐帧更新，公式可改写为：
$$
\begin{align}
\breve{R}(l,k) = d_k\,\breve{G}(l-1)\breve{S}(l-1,k) + \alpha _k\,\breve{R}(l-1,k).\tag{10}
\end{align}
$$

#### 2.HMM建模

对于HMM建模，有下面五要素具体分析：

1.隐藏状态集合(论文中定义为{$c_l$})

2.定义观测状态集合{$\begin{equation}
\boldsymbol {x}_l =\left({G}_{l},{\boldsymbol {R}}_l,{\boldsymbol {N}}_l\right)^T,
\end{equation}$}

3.转移矩阵A描述$c_{l-1}-->c_l$的关系

4.由隐藏状态$c_l$到观测状态$x_l$的的发射矩阵B

5.隐藏状态$c_l$的分布$\pi$

对于1，对于每个时间帧l，我们得到状态向量xl的高斯后验密度和纯净语音对数能量Sl的条件为HMM路径即为cl、

对于2，这个2K+1尺寸的观测状态的逐帧更新如下：
$$
\begin{equation}
\left\lbrace \begin{array}{l}
G_{l} = {G}_{l-1} \\
\boldsymbol {R}_{l} = {\log \left(\boldsymbol {\alpha }_{l-1} \odot \exp \left(\boldsymbol {R}_{l-1}\right) +
 \boldsymbol {d}_{l-1} \odot \exp \left(G_{l-1} + \boldsymbol {S}_{l-1}\right)\right)} \\
\boldsymbol {N}_{l} = \boldsymbol {N}_{l-1}
\end{array}\right.\tag{11}
\end{equation}
$$
对于3，基于贝叶斯，先进行状态序列计算，首先我们有$p(\boldsymbol {y}_{1:l-1},\mathbf {c}_{l-1})$,$p(\boldsymbol {x}_{l-1}|\boldsymbol {y}_{l-1},\mathbf {c}_{l-1})$,$p(\boldsymbol {S}_{l-1}|\boldsymbol {y}_{l-1},\mathbf {c}_{l-1})$

可以计算：
$$
\begin{align}
p(& \boldsymbol {y}_{1:l}, \mathbf {c}_{l}) = \nonumber \\
& \qquad p(\boldsymbol {y}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l-1})P(c_l|c_{l-1})p(\boldsymbol
 {y}_{1:l-1},\mathbf {c}_{l-1})\tag{12}
\end{align}
$$
其中：
$$
\begin{align}
p(\boldsymbol {y}_{l}|c_l, \mathbf {c}_{l-1}, & \boldsymbol {y}_{l-1}) = \nonumber \\
& \int _{\boldsymbol {x}_l}p(\boldsymbol {y}_{l}|c_l,\boldsymbol {x}_{l})p(\boldsymbol {x}_{l}|\mathbf
 {c}_{l-1},\boldsymbol {y}_{l-1})d\boldsymbol {x}_{l}\tag{13}
\end{align}
$$

$$
\begin{align}
p(\boldsymbol {x}_{l}|\mathbf {c}_{l-1},\boldsymbol {y}_{l-1}) = \int _{\boldsymbol {x}_{l-1}}p(\boldsymbol
 {x}_{l-1},\boldsymbol {x}_{l}|\mathbf {c}_{l-1},\boldsymbol {y}_{l-1})d\boldsymbol {x}_{l-1}.\tag{14}
\end{align}
$$

隐藏状态转移基于公式：
$$
\begin{equation}
\forall c_l, \; \hat{\mathbf {c}}_l= \mathop{\arg\,\max}_{\lbrace \mathbf {c}_{l-1}, c_l\rbrace }\,p\left(\boldsymbol
 {y}_{1:l},\lbrace \mathbf {c}_{l-1}, c_l\rbrace \right).\tag{15}
\end{equation}
$$
基于最大可能性的路径，进行隐藏状态$c_{l-1}-->c_l$的转移

对于4，观测状态的更新，先计算相关后验公式：

首先定义方程
$$
\begin{align}
\boldsymbol {x}_l &= f(\boldsymbol {x}_{l-1},\boldsymbol {S}_{l-1}) + \boldsymbol {\epsilon }_l \\
\boldsymbol {y}_l &= h(\boldsymbol {x}_l,\boldsymbol {S}_l) + \boldsymbol {\nu }_l\tag{16}
\end{align}
$$


我们用$\boldsymbol {\mu }_{\boldsymbol {x}_l}和\boldsymbol {\Sigma }_{\boldsymbol {x}_l}$表示$\boldsymbol {x}_l$的概率密度函数的均值和协方差矩阵。 HMM在隐藏状态$c_l$状态时,我们可以从训练数据的均值$\boldsymbol {\mu }_{\boldsymbol {S}_{c_l}}$和协方差矩阵$\boldsymbol {\Sigma }_{\boldsymbol {S}_{c_l}}$得到先验分布$p(\boldsymbol {S}_l|c_l)$。

图2中的“预测”块计算了路径相关的高斯先验密度$p(\boldsymbol {x}_{l}| \mathbf {c}_{l-1},\boldsymbol {y}_{l-1})$。我们定义$\boldsymbol {F}_{l-1}$为预测方程(16)在$\boldsymbol {\mu }_{\boldsymbol {x}_{l-1}}$和$\boldsymbol {\mu }_{\boldsymbol {S}_{l-1}}$处取值的f的雅可比矩阵。它可以写成
$$
\begin{equation}
\boldsymbol {F}_{l-1} = \left({\begin{array}{lc}
\boldsymbol {F}_{\boldsymbol {x}_{l-1}} & \boldsymbol {F}_{{\boldsymbol {S}}_{l-1}}\end{array}}\right)\tag{17}
\end{equation}
$$
其中，$\boldsymbol {F}_{\boldsymbol {x}_{l-1}} = \left.\frac{\partial f}{\partial \boldsymbol {x}_{l-1}} \right|_{\boldsymbol {\mu }_{\boldsymbol {x}_{l-1}}}$,$\boldsymbol {F}_{{\boldsymbol {S}}_{l-1}} = \left.\frac{\partial f}{\partial {\boldsymbol {S}}_{l-1}} \right|_{\boldsymbol {\mu }_{{\boldsymbol {S}}_{l-1}}}$

定义增广状态$\begin{equation} \boldsymbol {x}^{\star }_{l-1} = (\boldsymbol {x}_{l-1},{\boldsymbol {S}}_{l-1})^T=\boldsymbol {\mu }_{\boldsymbol {x}^{\star }_{l-1}} + \delta \boldsymbol {x}^{\star }_{l-1} \end{equation}$

省去一些中间计算

有：
$$
\begin{equation}
p(\boldsymbol {x}_{l-1},\boldsymbol {x}_{l}|\mathbf {c}_{l-1},\boldsymbol {y}_{l-1}) \sim \mathcal {N}\left(\boldsymbol
 {m} \,,\, \boldsymbol {P}\right)\tag{18}
\end{equation}
$$

$$
\begin{align}
&\boldsymbol {m} = \left(\begin{array}{c}\boldsymbol {\mu }_{x_{l-1}} \\
f(\boldsymbol {\mu }_{x_{l-1}},\boldsymbol {\mu }_{{\boldsymbol {S}}_{l-1}})\end{array}\right) \\
&\boldsymbol {P} = \nonumber\\
&{\left(\begin{array}{cc}\boldsymbol {\Sigma }_{x_{l-1}}\; & \; \boldsymbol {\Sigma }_{x_{l-1}}\boldsymbol
 {F}_{x_{l-1}}^T \\
\boldsymbol {F}_{x_{l-1}}\boldsymbol {\Sigma }_{x_{l-1}}\; &\; \boldsymbol {F}_{x_{l-1}}\boldsymbol {\Sigma
 }_{x_{l-1}}\boldsymbol {F}_{x_{l-1}}^T + \boldsymbol {F}_{{\boldsymbol {S}}_{l-1}}\boldsymbol {\Sigma }_{{\boldsymbol
 {S}}_{l-1}}\boldsymbol {F}_{{\boldsymbol {S}}_{l-1}}^T + \boldsymbol {Q}_l \end{array}\right)}\tag{19}
\end{align}
$$

对于后验分布$p(\boldsymbol {x}_{l-1}|\mathbf {c}_{l-1},\boldsymbol {y}_{l-1})$和$p(\boldsymbol {S}_{l-1}|\mathbf {c}_{l-1},\boldsymbol {y}_{l-1})$，我们有了$\boldsymbol {x}_{l-1}$和${\boldsymbol {S}}_{l-1}$的均值和协方差矩阵，因此可以计算xl的边际概率密度为：
$$
\begin{equation}
p(\boldsymbol {x}_{l}| \mathbf {c}_{l-1},\boldsymbol {y}_{l-1}) \sim \mathcal {N}\left(\boldsymbol {\mu }_{\boldsymbol
 {x}_l|\mathbf {c}_{l-1}},\, \boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}}\right)\tag{20}
\end{equation}
$$
其中：
$$
\begin{align}
\boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} &= f(\boldsymbol {\mu }_{\boldsymbol
 {x}_{l-1}},\boldsymbol {\mu }_{{\boldsymbol {S}}_{l-1}}) \\
\boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} &= \boldsymbol {F}_{\boldsymbol {x}_{l-1}}\boldsymbol
 {\Sigma }_{\boldsymbol {x}_{l-1}}\boldsymbol {F}_{\boldsymbol {x}_{l-1}}^T + \boldsymbol {F}_{{\boldsymbol
 {S}}_{l-1}}\boldsymbol {\Sigma }_{{\boldsymbol {S}}_{l-1}}\boldsymbol {F}_{{\boldsymbol {S}}_{l-1}}^T + \boldsymbol
 {Q}_l\tag{21}
\end{align}
$$
接下来开始观察状态的更新：

有了观察状态对于$N^2$条路径$\lbrace \mathbf {c}_{l-1}, c_l\rbrace$的的极大似然：$p({\boldsymbol {y}}_{l}|c_l,\mathbf {c}_{l-1},{\boldsymbol {y}}_{l-1})$,以及隐藏状态和纯净信号能量的后验分布$p(\boldsymbol {x}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l},\boldsymbol {y}_{l-1})$和$p(\boldsymbol {S}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l},\boldsymbol {y}_{l-1})$ 。我们可以利用观测方程(16)中h的一阶泰勒级数近似来得到$yl$和$xl$的近似高斯联合分布的均值和协方差。我们定义$Hl$为$h(xl,Sl)$在$(\boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}},\boldsymbol {\mu }_{\boldsymbol {S}_{c_l}})$处取值的雅可比Jacobian矩阵，因此
$$
\begin{equation}
\boldsymbol {H}_{l} = \left(\begin{array}{ll}\boldsymbol {H}_{\boldsymbol {x}_{l}} & \boldsymbol {H}_{{\boldsymbol
 {S}}_{l}}\end{array}\right).\tag{22}
\end{equation}
$$


对于由{cl−1,cl}定义的路径有：
$$
\begin{equation}
p(\boldsymbol {x}_{l},\boldsymbol {y}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l-1}) \sim \mathcal
 {N}\left(\boldsymbol {m_{xy}} \,,\, \boldsymbol {C_{xy}}\right)\tag{23}
\end{equation}
$$
其中
$$
\begin{align}
&\boldsymbol {m_{xy}} = \left(\begin{array}{c}\boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} \\
h(\boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}},\boldsymbol {\mu }_{\boldsymbol
 {S}_{c_l}})\end{array}\right) \\
&\boldsymbol {C_{xy}} = \left(\begin{array}{cc}I_{(K,2K+1)} & O_{K} \\
\boldsymbol {H}_{\boldsymbol {x}_{l}} & \boldsymbol {H}_{{\boldsymbol {S}}_{l}} \end{array}\right) \times
 \left(\begin{array}{cc}
\boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} & 0 \\
0 & \boldsymbol {\Sigma }_{\boldsymbol {S}_{c_l}} \end{array}\right) \nonumber \\
& \quad \times \left(\begin{array}{cc}I_{(K,2K+1)} & O_{K} \\
\boldsymbol {H}_{\boldsymbol {x}_{l}} & \boldsymbol {H}_{{\boldsymbol {S}}_{l}} \end{array}\right)^T +
 \left(\begin{array}{cc}
0 & 0 \\
0 & \boldsymbol {M}_l \end{array}\right)\nonumber \\
& = {\left(\begin{array}{cc}
\boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} & \boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf
 {c}_{l-1}}\boldsymbol {H}_{\boldsymbol {x}_{l}}^T \\
\boldsymbol {H}_{x_{l}}\boldsymbol {\Sigma }_{x_l|c_{l-1}} & \boldsymbol {H}_{x_{l}}\boldsymbol {\Sigma
 }_{x_l|c_{l-1}}\boldsymbol {H}_{x_{l}}^T + \boldsymbol {H}_{{\boldsymbol {S}}_{l}}\boldsymbol {\Sigma }_{\boldsymbol
 {S}_{c_l}}\boldsymbol {H}_{{\boldsymbol {S}}_{l}}^T + \boldsymbol {M}_l \end{array}\right).}\tag{24}
\end{align}
$$
因此我们有观察结果的极大似然为：
$$
\begin{equation}
p({\boldsymbol {y}}_{l}|c_l,\mathbf {c}_{l-1},{\boldsymbol {y}}_{l-1}) \sim \mathcal {N}\left(\boldsymbol {\mu
 }_{{\boldsymbol {y}}_l},\boldsymbol {\Sigma }_{{\boldsymbol {y}}_{l}}\right)\tag{25}
\end{equation}
$$
其中：
$$
\begin{align}
\boldsymbol {\mu }_{{\boldsymbol {y}}_l} &= h(\boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}},\boldsymbol
 {\mu }_{\boldsymbol {S}_{c_l}}) \\
\boldsymbol {\Sigma }_{{\boldsymbol {y}}_{l}} &= \boldsymbol {H}_{\boldsymbol {x}_{l}}\boldsymbol {\Sigma
 }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}}\boldsymbol {H}_{\boldsymbol {x}_{l}}^T + \boldsymbol {H}_{{\boldsymbol
 {S}}_{l}}\boldsymbol {\Sigma }_{\boldsymbol {S}_{c_l}}\boldsymbol {H}_{{\boldsymbol {S}}_{l}}^T + \boldsymbol {M}_l\tag{26}
\end{align}
$$
且$\boldsymbol {x}_l$的后验pdf（概率密度函数）为：
$$
\begin{equation}
p(\boldsymbol {x}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l},\boldsymbol {y}_{l-1}) \sim \mathcal
 {N}\left(\boldsymbol {\mu }_{\boldsymbol {x}_{l}},\boldsymbol {\Sigma }_{\boldsymbol {x}_{l}}\right)\tag{27}
\end{equation}
$$

$$
\begin{align}
\boldsymbol {\mu }_{\boldsymbol {x}_{l}} &= \boldsymbol {\mu }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} + \boldsymbol
 {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}}\boldsymbol {H}_{\boldsymbol {x}_{l}}^T\boldsymbol {\Sigma
 }_{\boldsymbol {y}_{l}}^{-1}[\boldsymbol {y}_l - \boldsymbol {\mu }_{\boldsymbol {y}_l}] \\
\boldsymbol {\Sigma }_{\boldsymbol {x}_{l}} &= \boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}} -
 \boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf {c}_{l-1}}\boldsymbol {H}_{\boldsymbol {x}_{l}}^T\boldsymbol {\Sigma
 }_{\boldsymbol {y}_{l}}^{-1}\boldsymbol {H}_{\boldsymbol {x}_{l}}\boldsymbol {\Sigma }_{\boldsymbol {x}_l|\mathbf
 {c}_{l-1}}\tag{28}
\end{align}
$$

类似的纯净语音对数能量$S_l$的后验pdf为：
$$
\begin{equation}
p(\boldsymbol {S}_{l}|c_l,\mathbf {c}_{l-1},\boldsymbol {y}_{l},\boldsymbol {y}_{l-1}) \sim \mathcal
 {N}\left(\boldsymbol {\mu }_{{\boldsymbol {S}}_{l}},\boldsymbol {\Sigma }_{{\boldsymbol {S}}_{l}}\right)\tag{29}
\end{equation}
$$

$$
\begin{align}
\boldsymbol {\mu }_{{\boldsymbol {S}}_{l}} &= \boldsymbol {\mu }_{\boldsymbol {S}_{c_l}} + \boldsymbol {\Sigma
 }_{\boldsymbol {S}_{c_l}}\boldsymbol {H}_{\boldsymbol {S}_{l}}^T\boldsymbol {\Sigma }_{\boldsymbol
 {y}_{l}}^{-1}[\boldsymbol {y}_l - \boldsymbol {\mu }_{\boldsymbol {y}_l}] \\
\boldsymbol {\Sigma }_{{\boldsymbol {S}}_{l}} &= \boldsymbol {\Sigma }_{\boldsymbol {S}_{c_l}} - \boldsymbol
 {\Sigma }_{\boldsymbol {S}_{c_l}}\boldsymbol {H}_{\boldsymbol {S}_{l}}^T\boldsymbol {\Sigma }_{\boldsymbol
 {y}_{l}}^{-1}\boldsymbol {H}_{\boldsymbol {S}_{l}}\boldsymbol {\Sigma }_{\boldsymbol {S}_{c_l}}.\tag{30}
\end{align}
$$

#### 3.相关代码分析

代码未完待续

```
%-------------------------------------------------------------------------%
%Prepare the STFT
%算一帧的点数，如果csts.ri被设置为1则计算最接一帧实际点数的2的指数值
if Csts.ri
    ni=pow2(nextpow2(Csts.ti*fs*sqrt(0.5)));%用最接近的2的指数值来近似一帧的点数
else
    ni=round(Csts.ti*fs);    % frame increment in samples%一帧的点数
end

tinc=ni/fs;          % true frame increment in time计算一帧的时间
no=round(Csts.of);   % integer overlap factor重叠系数整数化（系数用于窗的覆盖帧数）
nf=ni*no;            % fft length (size of analysis window)分析用的窗的长度（fft长度）=重叠系数no*一帧实际长度ni
w_synth=hann(nf+1)'; w_synth(end)=[];  % hamming window for synthesis定义海宁窗hann，长度为nf+1
w=w_synth/sqrt(sum(w_synth(1:ni:nf).^2));%？？？这一步不太明白，元素除以（重叠系数no帧的每帧第一个元素的平方和的开根）
%在no=6，ni=64时，w_synth(1:ni:nf)为0    0.2500    0.7500    1.0000    0.7500    0.2500
%-------------------------------------------------------------------------%


%梅尔尺度变换矩阵
%Mel scale transformation matrices
[ThO_Tr,~]=filtbankm(25,nf,fs,[],[],'m'); %forward transformation matrix)前向变换矩阵，这里其实就是设置一个mel滤波器组
%filtbankm的usage实例：
%(1) Calcuate the Mel-frequency Cepstral Coefficients

%         f=v_rfft(s);                     % v_rfft() returns only 1+floor(n/2) coefficients
%         x=v_filtbankm(p,n,fs,0,fs/2,'m');% n is the fft length, p is the number of filters wanted，n为fft点数，p为需要的滤波器个数
%         z=log(x*abs(f).^2);              % multiply x by the power spectrum
%         c=dct(z);                        % take the DCT
%代码分别对应
%(1)语音信号的预处理，主要有预加重，分帧，加窗，对加窗后的各帧信号进行快速傅里叶变换得到各帧的频谱。并对语音信号的频谱取模平方得到语音信号的功率谱。
%(2)将能量谱通过一组Mel尺度的三角形滤波器组，Mel三角带通滤波器组频率响应定义为:。。。此处插入图
%(3)使用功率谱，计算每个滤波器组输出的对数能量：
%(4)经离散余弦变换（DCT）得到MFCC系数


reco_mat = interpofiltbankm(25,nf,fs); %inverse transformation matrix (interpolation of the gain from mel bands to full STFT spectrum - see function below)
%此为自定义函数，详细分析见函数定义处，貌似是恢复矩阵


%-------------------------------------------------------------------------%
[DFTc,~]=enframe(input_speech,w,ni,'z'); %zero-padding the end of the signal to match the number of frames
%对input_speech进行分帧，w是hann(framelength)的意思，，ni帧移，‘z’模式是对末位补零匹配，这一步完成了对信号的分帧加窗
C=rfft(DFTc,nf,2);%对数据进行FFT，fft点数为nf，2是维数
[nrows,ncols] = size(C);%频谱的行与列值，nrows是分段后总段数，ncols为有效fft点数=fft点数/2+1，因为实信号频谱共轭对称
gt_YP_full=(C.'.*conj(C.')./(ncols*sum(w.^2)));%C.'.*conj(C.')得到的是频谱元素模的平方。之后元素除以实际上（有效fft点数*一帧的点数）
gt_YP = 10.*log10(ThO_Tr*gt_YP_full);%计算每个滤波器组输出的对数能量
gt_YP = max(gt_YP,max(gt_YP(:))+Csts.ef); % clip to 60 dB range avoid negative infinities
%上面这一行代码的功能是将db范围限制在60db以内，即最大和最小相差不超过60db，其中Csts.ef=60
Energy = 10.^(gt_YP./10);%计算修正后每个滤波器组输出的对数能量



%-------------------------------------------------------------------------%
%various initialisations变量初始化
K = Csts.cl; %Number of states in our HMM speech model，HMM语音模型中状态的数量，默认为6
nfc = size(gt_YP,1);%其实就是滤波器数目，上面设为了25，这里就是25
nb_frames = size(gt_YP,2);%就是emframe分出来的总的段数


%这里默认为快速模式，暂时用不到慢速模式，所以略过
if Csts.mo ==0%这里Csts.mo=1为快速模式，=0为慢速模式
    %UDU decomposition of the covariance of each state distribution
    U_state = zeros(nfc,nfc,K);
    D_state = zeros(nfc,nfc,K);
    for i=1:K
        [tmpU,tmpD] = udu(covStates(:,:,i));
        U_state(:,:,i) = tmpU;
        D_state(:,:,i) = tmpD;
    end
end



%init clean speech posterior mean and covariance with priors
%初始化干净语音后验均值和先验协方差

M_speech = mStates;%这里的mStates是字典
if Csts.mo ==0%如果模式为慢速模式
    U_speech = U_state;
    D_speech = D_state;
else
    Cov_speech = covStates;
end


%probs of each path
probs = log((1/(K)).*ones(K,1)); %initialise the probabilities    HMM的初始概率分布pi
%our state space representation --> [gain;reverb_power_in_subbands;noise_power_in_subbands]在论文中为x_L=（G_L,R_L,N_L）,状态空间声明
X = zeros(2*nfc+1,K);%尺寸是2*MFCC频点数目（滤波器数）+1，k为状态数
X(1,:) = -12.*ones([1,K]);%初始化增益为-12
%对于混响能量和噪声能量都是用观察信号第一帧来初始化
X(2:nfc+1,:) = repmat((mean(gt_YP(:,1:12),2)),[1,K]); %initialise reverberation energy to the mean observed power of the first frames (same as noise)
X(nfc+2:2*nfc+1,:) = repmat(mean(gt_YP(:,1:12),2),[1,K]);%initialise noise level to the mean of the 1st few frames



%初始化混响参数
%alpha参数的定义对应公式7，对于不同的中心频率k的梅尔滤波器，有不同的T60值
alpha = 10.^((-6.*tinc)./[0.625814328889286,0.635814328889286,0.559669908460684,0.533597692539940,0.523597692539940,0.519657090507566,...
    0.498220803534608,0.488220803534608,0.470970193330601,0.460004032902958,0.445158336751369,0.436812074651546,...
    0.400699712568631,0.395830867067534,0.399005310050930,0.401204573100827,0.417815999144577,...
    0.417892069024374,0.419406546706839,0.422573783237992,0.417601620276530,0.408528394010926,...
    0.403943250672072,0.354372247720777,0.316876109784076])'; %average subbands T60 values from measurements on a variety of RIRs (inaccurate at low freqs)
%依照公式8，f是一个对应不同DRR的dk值表，从-2db到8db
f = (1-alpha)./(10.^(linspace(-0.2,0.8,25)')); %corresponds to DRR = [-2dB --> 8dB]
Rp = [10.*log10(alpha./(1-alpha))+1.5;10.*log10(f./(1-f))];%这步没弄明白


%初始化一些数组
X_rev = X(:,1);
Speech_rev = mStates(:,1);


%设置时间更新的协方差矩阵
Qx=[1/350000,zeros([1,2*nfc]);...
    zeros([nfc,1]),diag(repmat(1/1500,[nfc,1])),zeros(nfc);...
    zeros([nfc,nfc+1]),diag(repmat(1/7550,[nfc,1]))];
Qr=[diag(repmat(1/1700,[nfc,1])),zeros(nfc);...
    zeros(nfc),diag(repmat(1/700,[nfc,1]))];
Qx = 15.*Qx + 1e-5*eye(2*nfc+1)*trace(Qx);
Qr = 15.*Qr + 1e-5*eye(2*nfc)*trace(Qr);
Covx = Qx.*1.5;
Covr = Qr.*15;
%观察状态的噪声协方差的初始化
%Initialise the noise covariance matrix with the observation from the first few frames
% Covx(nfc+2:end,nfc+2:end) = cov(gt_YP(:,1:10).');
Covx(nfc+2:end,nfc+2:end) = Covx(nfc+2:end,nfc+2:end)+cov(gt_YP(:,1:10).'); % modified to prevent errors when initial frames are silent
if Csts.mo == 0 %UDU decomposition of the state space covariance matrix (SR-EKF implementation)
    Uq = eye(2*nfc+1);
    [tU,tD] = udu(Covx);
    Up = repmat(tU,[1,1,K]);
    Dp = repmat(tD,[1,1,K]);
    Um = zeros(2*nfc+1,2*nfc+1,K);
    Dm = Um;
else
    Cov = cell(K,1);
    for i=1:K
        Cov{i} = Covx;
    end
end
%
%矩阵定义，用于预测和更新阶段的计算 (vectorisation purposes)
pdm = [zeros(nfc,1),eye(nfc),zeros(nfc),eye(nfc),zeros(nfc,2*nfc);...
    zeros(nfc,2*nfc+1),eye(nfc),zeros(nfc,2*nfc);...
    ones(nfc,1),zeros(nfc,3*nfc),eye(nfc),eye(nfc);...
    zeros(nfc,3*nfc+1),eye(nfc),zeros(nfc)];
pdm2 = [zeros(nfc,1);ones(nfc,1);zeros(nfc,1);ones(nfc,1)];
pdm2_aug = repmat(pdm2,1,K);
pdm3 = [ones(nfc,1),zeros(nfc,2*nfc),eye(nfc);...
    zeros(nfc,1),eye(nfc),zeros(nfc,2*nfc);...
    zeros(nfc,nfc+1),eye(nfc),zeros(nfc)];
stat_mat = reshape(repmat(mStates,K,1),nfc,K^2);
%Declaration of Jacobians%雅可比矩阵。在向量微积分中，雅可比矩阵是一阶偏导数以一定方式排列成的矩阵，其行列式称为雅可比行列式。
Fx = eye(2*nfc+1);
Fu = zeros(2*nfc+1,nfc);
if Csts.mo ==0
    Hx = zeros(nfc,2*nfc+1);
else
    Hx = cell(K,K);
    Hu = cell(K,K);
end
%Initialise Gain
SpecGain = zeros(ncols,nb_frames+1); SpecGain(:,1) = 0.00001.*ones(ncols,1);
%%%%% 为计算干净语音的MMSE估计量做的准备 %%%%%
if Csts.sg == 3
    Xi = 0.00001.*ones(nfc,1);
    p0 = 0.5;
    pInf = 1;
    mu_mmse = 0.5; % shape parameter of the chi distribution for clean speech magnitude
    beta_mmse = 0.5; % compression factor
    gammaFactor = (gamma(mu_mmse+beta_mmse/2)./gamma(mu_mmse));
end


%主程序从这里开始
%-------------------------------------------------------------------------%
%Main loop, frame by frame processing
for idx=2:nb_frames+1
    
    %%%%% first do the prediction stage for each track %%%%%
    
    if Csts.mo ==0
        tmpX = X;
        for k = 1 : K %for each track对每条路径定义迭代
            tmpaug = [X(:,k);Rp;M_speech(:,k)]; %create the augmented state定义增广状态
            tmp = reshape(exp((pdm*tmpaug).*0.2302585093)+pdm2,[nfc,4]); %0.2302585093 = log(10)/10
            tmp_sum = tmp(:,1)./tmp(:,2) + tmp(:,3)./tmp(:,4);
            %计算输出状态的混响功率部分
            tmpX(2:nfc+1,k) = 10.*log10(tmp_sum);
            %计算雅克比矩阵
            Fx(2:nfc+1,1:nfc+1) = [(tmp(:,3)./tmp(:,4))./tmp_sum,diag((tmp(:,1)./tmp(:,2))./tmp_sum)];
            Fu(2:nfc+1,:) = diag(Fx(2:nfc+1,1));
            %计算协方差矩阵
            [Um(:,:,k),Dm(:,:,k)] = mwgs_factor([Uq,Fx*Up(:,:,k),Fu*U_speech(:,:,k)],diag([diag(Qx);diag(Dp(:,:,k));diag(D_speech(:,:,k))])); %Covariance matrix of the prediction stage in UDU form
        end
    else
        tmpaug = [X;repmat(Rp,1,K);M_speech];%create the augmented state定义增广状态
        tmp = exp((pdm*tmpaug).*0.2302585093)+pdm2_aug;
        tmp_sum1 = tmp(1:nfc,:)./tmp(nfc+1:2*nfc,:);
        tmp_sum2 = tmp(2*nfc+1:3*nfc,:)./tmp(3*nfc+1:end,:);
        tmp_sum = tmp_sum1 + tmp_sum2;
        %c计算输出状态的混响功率部分
        X(2:nfc+1,:) = 10.*log10(tmp_sum);
        %雅可比矩阵比较
        Ftemp = [tmp_sum2./tmp_sum;tmp_sum1./tmp_sum];
        for k = 1 : K %for each track
            %计算雅克比矩阵
            Fx(2:nfc+1,1:nfc+1) = [Ftemp(1:nfc,k),diag(Ftemp(nfc+1:end,k))];
            Fu(2:nfc+1,:) = diag(Ftemp(1:nfc,k));
            %计算协方差矩阵
            Cov{k} = Fu*Cov_speech(:,:,i)*Fu' + Fx*Cov{k}*Fx' + Qx;
        end
    end
    new_probs = repmat(probs,1,K) + trans_probs;
    
    % 现在更新阶段有K^2种可能 
    
    %我们需要存储的各种结果的初始化
    err = zeros(nfc,K,K);
    Sk = zeros(nfc,nfc,K,K);
    lkl = zeros(K,K);
    if Csts.mo ==0
        vx = zeros(2*nfc+1,nfc,K,K);
        vu = zeros(nfc,nfc,K,K);
        for k1 = 1:K
            for k2 = 1:K
                %得到增广状态的平均值
                m_tilde = [tmpX(:,k1);mStates(:,k2)];
                %计算雅克比
                tmp2 = reshape(exp((pdm3*m_tilde).*0.2302585093),[nfc,3]);
                tmp_sum2 = sum(tmp2,2);
                Hx(:,1) = tmp2(:,1)./tmp_sum2;
                Hx(:,2:nfc+1) = diag(tmp2(:,2)./tmp_sum2);
                Hx(:,nfc+2:2*nfc+1) = diag(tmp2(:,3)./tmp_sum2);
                Hu = diag(Hx(:,1));
                %计算预测的输出和误差
                zk = 10.*log10(tmp_sum2);
                err(:,k1,k2) = gt_YP(:,idx-1)-zk; %error between update stage and observed log-power
                R = diag(kappa_s+((10/log(10))^2).*log(1+2.*(tmp2(:,1).*tmp2(:,2)+tmp2(:,1).*tmp2(:,3)+tmp2(:,2).*tmp2(:,3))./tmp_sum2));
                %计算xl的边际概率密度
                vx(:,:,k1,k2) = (Um(:,:,k1)')*Hx';
                vu(:,:,k1,k2) = (U_state(:,:,k2)')*Hu';
                [Usk,Dsk] = mwgs_factor([eye(nfc),vx(:,:,k1,k2)',vu(:,:,k1,k2)'],diag([diag(R);diag(Dm(:,:,k1));diag(D_state(:,:,k2))])); %Covariance matrix of the prediction stage in UDU form
                Sk(:,:,k1,k2) = Usk*Dsk*Usk'; %covariance matrix of the observation
                %计算极大似然概率
                lkl(k1,k2) = gaussmixp(gt_YP(:,idx-1)',zk',Sk(:,:,k1,k2)); %gaussmixp returns log-probability
            end
        end
        
    else
        %得到增广状态的平均值
        m_tilde = [repmat(X,1,K);stat_mat];
        tmp2 = exp((pdm3*m_tilde).*0.2302585093);
        tmp2_sum2 = tmp2(1:nfc,:) + tmp2(nfc+1:2*nfc,:) + tmp2(2*nfc+1:end,:);
        %预测输出，对应图2
        zk = 10.*log10(tmp2_sum2);
        %观测噪声
        R = kappa+((10/log(10))^2).*2.*(tmp2(1:nfc,:).*tmp2(nfc+1:2*nfc,:)+tmp2(1:nfc,:).*tmp2(2*nfc+1:end,:)+tmp2(nfc+1:2*nfc,:).*tmp2(2*nfc+1:end,:))./tmp2_sum2;
        %Jacob comp
        Htemp = tmp2./repmat(tmp2_sum2,3,1);
        for k1 = 1:K
            for k2 = 1:K
                %计算雅克比
                Hx{k1,k2} = [Htemp(1:nfc,K*(k2-1)+k1),diag(Htemp(nfc+1:2*nfc,K*(k2-1)+k1)),diag(Htemp(2*nfc+1:end,K*(k2-1)+k1))];
                Hu{k1,k2} = diag(Htemp(1:nfc,K*(k2-1)+k1));
                %计算预测误差
                err(:,k1,k2) = gt_YP(:,idx-1)-zk(:,K*(k2-1)+k1); %error between update stage and observed log-power
                %计算xl的边际概率密度
                Sk(:,:,k1,k2) = Hx{k1,k2}*Cov{k1}*Hx{k1,k2}' + Hu{k1,k2}*covStates(:,:,k2)*Hu{k1,k2}' + diag(R(:,K*(k2-1)+k1));
                %计算极大似然概率
                try
                    lkl(k1,k2) = gaussmixp(gt_YP(:,idx-1)',zk(:,K*(k2-1)+k1)',Sk(:,:,k1,k2)); %gaussmixp returns log-probability
                catch
                    error('Covariance matrix not positive definite - To avoid this, try a higher energy floor (e.g. algo_params.ef=-50) or running the algorithm in slow mode using algo_params.mo = 0');
                end
            end
        end
    end
```

