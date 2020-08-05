# p.563算法原理解析

## 1.概览

​		下图展示了，人工主观语音评估mos-lqs，双端和单端客观语音评估mos-lqo这三种方法的差异。

<center><img src="https://pangcong1117.github.io/myblog/images/语音评估模式图.png"/>


​		p563算法可以被想象成一个专家使用测试设备如传统的听筒侦听一个真实的电话。这个形象的表达解释了算法主要的应用并且允许使用者去对P563算法获得的分数进行评分。P563算法给出的质量分通过在测量点连接一个常规的听筒基于感知质量预测。

​		因此，侦听设备是p563算法的一部分，信号首先会被预处理。整个预处理过程由话音接收端模型开始。之后是通过语音活动检测算法（VAD）信号中包含话音的部分并计算话音电平。最终话音电平被调整到-26dbov（3300以下）。预处理过的语音数据将分别进行几项独立分析研究，像一层多个传感器，检测一组表征信号的参数。之后基于一组关键参数，开始分析主要的失真类型。

​		关键参数和分析出的主要失真类别等用于调整语音质量模型。尽管多个不同失真发生在同一个信号中，但某些失真要比其他失真作用更显著，这提供了一个基于感知的加权。P563的基础方框图下图给出:



<center>
    <img src="https://pangcong1117.github.io/myblog/images/P563算法方框图.png"/>

​	

​		P563算法的信号参数化可以被分为以下三个独立的函数部分，这与失真的主要类别相对应：

- **Vocal tract analysis and unnaturalness of speech声道分析和话音不自然度分析（音色音调及自然度角度）**
- **Analysis of strong additional noise强附加噪声的分析（噪声分析角度，分析与信噪比有关）**
- **Interruptions, mutes and time clipping语音间断，减弱，裁剪分析（语音元素完整角度）**

​		最终为了证明最终语音客观质量评分结果与语音质量主观评分mos-lqs关联，引入相关系数这一数学指标。P563和主观评分的关联程度可以用相关系数描述：
$$
r=\frac{\sum (x_{i}-\bar{x})(y_{i}-\bar{y})}{\sqrt{(x_{i}-\bar{x})^2(y_{i}-\bar{y})^2}}
$$
​		式子中，x~i~是第i种情景下主观mos得分，y~i~是对应情况下P.563算法计算出的客观质量评分，$\bar{x}$，$\bar{y}$是所有情况下的均值。

​		在已知24组遵循ITU基准的测试实验中，相关系数平均值为0.88。

## 2.详细算法原理

### 2.1语音前端相关描述符及预处理算法

#### 2.1.1  Voice activity detection端点检测

​		VAD算法基于一个自适应能量阈值，逐帧检测音频能量，超过阈值则为活跃语音信号，否则为无声段（背景噪声段），详细框图如下：

<center>
    <img src="https://pangcong1117.github.io/myblog/images/VAD算法框图.png"/>



​		自适应能量阈值th~est~首先设定为全局均值，之后自适应逼近环境噪声值，公式如下：
$$
\left\{\begin{array}{l}
m_{x}=\sum_{i=0}^{n-1}{\frac{x_{noise}(i)}{n}}       \\
\ th_{est}=m_{x}+2*\sqrt{\sum_{i=0}^{n-1}\frac{(x_{noise}(i)-m_{x})^2}{n}}
\end{array}\right.
$$
​		还有两点额外判定：

- 如果仅12ms及以下长度的语音能量超过阈值，标记为无声段
- 活跃语音段之间的间隔小于200ms的，连接在一起

#### 2.1.2  IRS filtering中间系统滤波

​		测试的信号经过中间系统的传递后，对某些参数会造成影响。基于此，在预处理阶段需要进行一下两项步骤：

1. 对250hz到3000hz的信号进行电平估测并归一化到-26dBov处
2. 在第二步中，使用FFT滤波器对整文件在频域应用滤波器。滤波器特性与ITU-T Rec. P.830中给出的中间参考系统接收特性相似

​		滤波后的信号即可应用于语音音量衰减、非人声、帧重复、非自然哔哔声、基本音质、非自然安静、局部背景噪声和全局背景噪声的测试中。

#### 2.1.3  Signal normalization and level calculation信号归一化及电平计算

​		为了归一化到-26 dBov，算法基于VAD的输出计算每个活动帧的均方根值来估计语音水平。信号电平被归一化后，使用四阶巴特沃思高通滤波器滤波，截止频率100hz。

​		除了计算语音静默的参数要求原始信号以外，所有不使用IRS滤波信号的参数都使用这个归一化信号。

#### 2.1.4  SpeechSectionsLevelVar描述符

​		SpeechSectionsLevelVar是一个描述句子之间层次变化的描述符，它计算信号中每个语音部分的电平以及最大值和最小值之间的差值。

#### 2.1.5  LocalLevelVar描述符

​		LocalLevelVar描述信号的能量变化。均方根值是逐帧计算的。然后计算均方根数组的一阶导数，计算功率均值。

#### 2.1.6Pitch synchronous extraction基音同步分析提取（很重要）

​		基音周期和基音频率数据在后续的P563算法中是需要的，基音周期及频率数据提取框图如下：



<center>
    <img src="https://pangcong1117.github.io/myblog/images/基音周期及频率数据提取框图.png"/>

​    



##### 2.1.6.1  Pitch period estimate and voice/unvoiced decision基音周期估计及浊音/清音判决

 		人在发音时，根据声带是否震动可以将语音信号分为清音跟浊音两种。**浊音又称有声语言，携带着语言中大部分的能量，浊音在时域上呈现出明显的周期性；而清音类似于白噪声，没有明显的周期性。**发浊音时，气流通过声门使声带产生张弛震荡式振动，产生准周期的激励脉冲串。**这种声带振动的频率称为基音频率，相应的周期就成为基音周期。**

​		基音周期估计是通过自相关方法实现的，自相关计算中每一帧64ms（512个点），帧重叠面积50%。信号首先进行汉宁窗加窗处理:
$$
y_{i}=0.5x_{i}[1-cos(w)]\quad for\ i=0..(n-1)
$$
​		其中:
$$
w=\frac{2\pi i}{n}\\
$$
​		x为输入信号，n为帧长度
$$
R_{y}(\tau)=y(\tau)\otimes y^{*}(-\tau)=fft^{-1}(fft(w)\times fft(w)^{*})
$$


自相函数表示一个信号和延迟$\tau$点后该信号本身的相似性。如果信号x(n)具有周期性，那么它的自相关函数也具有周期性，而且周期与信号x(n)的周期性相同。自相关函数提供了一种获取周期信号周期的方法。在周期信号周期的整数倍上，它的自相关函数可以达到最大值，因此可以不考虑起始时间，而从自相关函数的第一个最大值的位置估计出信号的基音周期，这使自相关函数成为信号基音周期估计的一种工具。

​		$R_{y}(\tau)$基于最大值$R_{y}(0)$归一化，使用幅值一半的汉宁窗过滤输出数组的32个之后的元素。滤波后的自相关系数N~i~由下式给出：

$$
N_{i}=\frac{R_i}{R_0}\ for\ i=0,1,2,...31\ andN_{i}=\frac{R_i}{2R_0}[1-cos(w)]\ for\ i=32,33,...,n-1
$$
​		其中
$$
w=\frac{2\pi i}{64}
$$
​		然后在第20个和第100个元素之间搜索最大值，其基音频率范围从80Hz(T=100\*0.125=12.5ms)到400Hz(T=20\*0.125=2.5ms)，（这是我的理解，与手册有些许出入）。如果这个最大值大于0.5，那么这帧被归类为浊音，保存候选基音周期最大值(Tmax)。

​		我们得到了每个浊音帧的基音周期，为了避免基音加倍（意思是我们找到的是谐音而不是基音），我们维护的候选基音周期最大值（$T_{max}$）需要与上一帧被判为浊音的基音周期$T_{old}$($T_{old}$相当于一个局部峰值）进行比较。

​		若满足如下公式：
$$
N_{T_{old}}>\frac{1+N_{T_{max}}}{3}
$$
​		则我们判定误拾谐音，之前找到的只是局部峰值，更新候选基音周期最大值$T_{max}$，而被$T_{old}$设置为当前帧的基音周期。



- ​		**图解基音周期提取原理**

​		为了更清晰的讲解基音周期的检测的原理，借用一张图片来说明，下图是某帧语音的自相关函数图，采样率16k：

<center><img src="https://pangcong1117.github.io/myblog/images/某帧的自相关函数图片.jpg"/>



​		该帧的自相关函数中，除去第一个最大值后（0处），最大值Kmax= 114，那么该帧对应的基频16kHz/114=140Hz。由于P563算法使用的采样率为8khz，所以与本图计算基频周期有较小的差别。更多语音基音检测的细节可参考这篇文章，可以加深对语音基频的理解，通俗易懂，博客链接https://blog.csdn.net/zouxy09/article/details/9141875。



##### 2.1.6.2  Pitch mark placement基音标注

​		精确地确定语音最大幅度的位置是语音信号基音同步分析的一个重要部分，基音周期被精确地估计出来之后，还需要进行基音标注，准确地标注基音脉冲的最大位置。

​		P563算法使用的方法是**加窗语音信号$x$与脉冲序列的互相关$y$（卷积）**来寻找基音标注位置。对基音标注的算法，标注的位置一般都是在基音脉冲的短时能量的尖峰时刻上，P563算法逐帧执行以找到最佳的基音标注位置。

​		互相关公式为：
$$
R_{xy}(\tau)=x(\tau)\otimes y(\tau)
$$

$$
x_{i}=0.5z_{i}[1-cos(w)]\quad for\ i=0...n
$$



​		其中$w=\frac{2\pi i}{n}$，z为输入语音原始信号，n为帧长，$\begin{cases}
y_{i}=1 & \text{if }i为基音周期l的倍数 \\ 
y_{i}=0  & \text{if } i为其他
\end{cases}$,$\tau$是偏移量，$l$是基音周期。

​		当上述算法计算结束，**$i_{max}$被标记为最大的$R_{xy}$的下标，基音就标记在这一帧偏差量为$i_{max}$的位置**。





##### 2.1.6.3	Resolution of inconsistencies  解决基音标注的不一致性

​		由于使用互相关的方法进行基音位置标注过程中时间对齐的误差，帧重叠的做法导致可能出现近距离上多个基音标记。如果两个基音标记之间的间隔小于10个点（1.25ms），则认为它们是由同一个基音派生而来的，相邻的基音间隔也被分析，那些长度超过给定阈值的基音间隔被标记为不一致。P563设计了一个局部调整的算法用于修改基音标记到最佳位置，并对基音标注不一致性进行记录。

​		在每个被判决为浊音的区段中，检测出**持续时间最长的基音**，这种稳定的基音会延伸到余下的部分。算法将基于此**加强一些基音标记位置，完全删除一些，并纠正其他位置**，基频一致性检测流程图如下图所示。

​		**算法具体流程为：在各浊音区段的一致性基音（持续时间最长最稳定的基音）检测到之后，启动之后的基音标记鉴别（流程图F2处）。在各浊音段本地检查，尝试将各段的音高标记更合理地放置在输入音频波形中并符合一致性阈值（流程图G2处）。如果不符合一致性检测，则删除当前不合理的基音标记，算法向后移动（流程图F3处），一区段一区段的进行检查。通过这种方式，实现所有浊音段的基音标记遵循一致性。**

<center><img src="https://pangcong1117.github.io/myblog/images/pitch-inconsistency handling.png"/>



##### 2.1.6.4  Boundary review浊音区段边界检查

​		由于处理的语音是**以帧为框架来划分**的，浊音区段有看似合理的基音标记，在边界处可能是不对的。**边界处的基音检测误差大大增加，甚至无法检测**。P563针对这个问题，采用了边界检查算法。下图为浊音段边界基音检查流程图（Voiced segment boundary pitch review），展示了用于评估和纠正浊音部分边界标记的算法。



<center><img src="https://pangcong1117.github.io/myblog/images/Voiced segment boundary pitch review.png"/>



​		在区段边界处，不断尝试可超过原定边界放置候选的更长基音周期（**try 周期更大，频率更小基音记为pitch-cycle**）进行评估(图6,C1)。将这个新定义的pitch-cycle与三个阈值进行比较(图6,D1, E2, F3)。**如果没通过任意一个测试，它将被pass掉，这个最后已知（成功通过三个阈值测试）的基音标记将被删除(图6,G1)**。这个基音标记被移除的位置被存储起来，用于后续的稳定性评估(图6,G2)。

​		在下一轮迭代中，对旧段边界内的基音标记进行详细检查。如果新定义的基音周期满足所有定义的阈值范围，则认定它为有效，并添加基音标记(图6,G3)。通过这种方式，分段可以缩小和扩展。根据下面定义的规则，若都满足，则将分段连接在一起。而当接受新的音高标记时，将与之前删除的标记进行比较(图6,H3)。当发现他们匹配时，则该段分界是稳定的，不需要进一步扩展或缩小。

- Length of cycle > Pre-determined threshold (Figure 6, D1);
- RMS energy of cycle > Pre-determined threshold (Figure 6, E2);
- Frequency content of new and current frame comparable (Figure 6, F3).



### 2.2	Description of the functional block 'Vocal tract analysis and Unnatural Voice'声道分析与非自然语音分析模块描述

​		P563算法基于一系列信号参数的集合来分类不自然语音以及进行语音质量评估的。基本上，这些信号参数可以细分为两组：

- Speech statistics语音统计学参数
- Vocal tract analysis声道分析参数

#### 2.2.1	Calculation of speech statistics for unnatural voice detection用于非自然语音检测的语音统计学参数计算

​		用于语音的统计学参数主要基于倒谱和LPC参数的高阶统计量分析这两种公认信号处理技术。**偏度(skewness)和峰度(kurtosis）**这两个高阶矩，特别适用于进一步分析信号的性质。这些是统计信号是**统计数据分布非高斯性**的经典数字特征。

​		下图是语音统计学参数计算流程图Calculation of speech statistics



<center><img src="https://pangcong1117.github.io/myblog/images/Calculation of speech statistics.png"/>


- 偏度（skewness），**是统计数据分布偏斜方向和程度的度量，是统计数据分布非对称程度的数字特征**。偏度(Skewness)亦称偏态、偏态系数。偏度表征概率密度曲线相对于平均值不对称程度的特征数。**直观看来就是密度函数曲线尾部的相对长度**。

  根据定义，偏度是样本的三阶标准化矩，定义式如下，其中$k_{3}$表示三阶中心矩：

$$
skew(X)=E[(\frac{X-\mu}{\sigma})^3]=\frac{k_{3}}{\sigma_{3}}
$$

​		以下是一些典型分布的偏态值:

​				拉普拉斯分布Laplace，正态分布normal，均匀分布normal：0

​				指数分布Exponential：2

​		下图为**不同数据分布的偏态特征图**：

<center><img src="https://pangcong1117.github.io/myblog/images/偏态.jpg"/>

- 峰度（kurtosis）又称峰态系数。表征概率密度曲线在平均值处峰值高低的特征数。直观看来，**峰度反映了峰部的尖度**。**样本的峰度是和正态分布相比较，如果峰度大于三，峰的形状比较尖，比正态分布峰要陡峭。反之亦然**。

  根据定义，峰度可以表示为四阶标准矩，其中$k^4$表示四阶中心距，$\sigma$是标准差：
  $$
  E[(\frac{X-\mu}{\sigma})^4]或\frac{k^4}{\sigma^4}
  $$
  P563算法中，峰度被定义四阶中心矩除以概率分布方差的平方再减去3：
  $$
  Kurt(X)=\frac{k^4}{\sigma^4}-3
  $$
  这也被称为超值峰度（excess kurtosis）。**“减3”是为了让正态分布的峰度为0（大多数软件对峰度的定义都包含“减3”）**。

  以下是一些典型分布的峰度值:

  ​		正态分布normal：0

  ​		拉普拉斯分布：3

  ​		指数分布Exponential：6

  其中正态分布峰度（超值峰度）为0的推理过程：
  $$
  Kurt(正态分布)=E[(\frac{X-\mu}{\sigma})^4]-3=E(Z^4)-3=\int_{-\infty}^{+\infty}\frac{x^4e^{-\frac{x^2}{2}}}{\sqrt{2\pi}}dx-3=0
  $$

  

  下图为**不同数据分布的峰度对比图**：

  <center><img src="https://pangcong1117.github.io/myblog/images/超值峰度.png" width = "500" />
  
  
  
##### 2.2.1.1	Calculation of LPCcurt and LPCskew参数LPCcurt和LPCskew的计算

​		线性预测编码（LPC）的峰度和偏度，以及LPC偏度的绝对值，特别适合于语音信号特性的估计。

​		线性预测分析的基本思想是：**由于语音样点之间存在相关性，所以可以用过去的样点值来预测现在或未来的样点值，即一个语音的抽样能够用过去若干个语音抽样或它们的线性组合来逼近。**预测公式如下：
$$
\hat{s}(n)=\sum_{i=1}^{p}a_{i}s(n-1)
$$
​		预测误差$\epsilon(n)$为：
$$
\epsilon(n)=s(n)-\hat{s}(n)=s(n)-\sum_{i=1}^{p}a_{i}s(n-i)
$$
​		通过在**某个准则下使预测误差$\epsilon(n)$达到最小值的方法**来决定唯一的一组预测系数$a_{i}(i=1,2,...,p)$。鉴于语音信号的时变特性，**LPC分析必须按帧进行，使预测误差在某个准则下最小，求预测系数的最佳估值$a_{i}$，这个准则通常采用最小均方误差准则**。

​		P563算法使用的LPC系数的数量为21。

- ​		每帧的**LPC标准差**计算公式如下，其中P为LPC系数数量：

$$
\sigma_{n}^{LPC}=\frac{1}{P}[\sum_{p=1}^{P}\alpha_{p}^2-\frac{1}{P}(\sum_{p=1}^{P}\alpha_{p})^2]
$$



​		通过对每个活跃语音帧的LPC向量求值，得到**LPC峰度和LPC偏度**：

- ​		LPC向量的峰度（kurtosis）计算公式如下，其中P为LPC系数数量
  $$
  K_{n}^{LPC}=\frac{1}{P}\sum_{p=1}^{P}(\frac{\alpha_{p}-\frac{1}{P}\sum_{p=1}^{P}\alpha_p}{\sigma_{n}^{LPC}})^4-3
  $$
  
- ​        LPC向量的偏度（skewness）计算公式如下，其中P为LPC系数数量

$$
S_{n}^{LPC}=\frac{1}{P}\sum_{p=1}^{P}(\frac{\alpha_{p}-\frac{1}{P}\sum_{p=1}^{P}\alpha_p}{\sigma_{n}^{LPC}})^3
$$

​		结果将在**所有活动语音帧上取平均值**：
$$
\overline{K}^{LPC}=\frac{1}{N}\sum_{n=1}^{N}K_{n}^{LPC} \\ \overline{S}^{LPC}=\frac{1}{N}\sum_{n=1}^{N}S_{n}^{LPC}
$$
​		式中，n=1...N，N为活跃语音帧的数量，活跃语音帧满足$E_n>E_{minSpeech}$。



##### 2.2.1.2	Calculation of CepADev, CepCurt and CepSkew 倒谱标准差，倒谱峰度，倒谱偏度计算

​		同理，对于倒谱系数的**标准差，峰度，偏度**的计算与之前内容类似，公式依次给出：
$$
\sigma_{n}^{cep}=\frac{1}{M}[\sum_{m=1}^{M}c_m-\frac{1}{M}(\sum_{m=1}^{M}c_m)^2]\\
K_{n}^{cep}=\frac{1}{M}\sum_{m=1}^{M}(\frac{c_m-\frac{1}{M}\sum_{m=1}^{M}c_m}{\sigma_{n}^{cep}})^4-3\\
S_{n}^{cep}=\frac{1}{M}\sum_{m=1}^{M}(\frac{c_m-\frac{1}{M}\sum_{m=1}^{M}c_m}{\sigma_{n}^{cep}})^3
$$
​		式中M为倒谱系数数量，m为倒谱系数下标

​		同理，结果将**在所有活动语音帧上取平均值**：
$$
\overline{K}^{cep}=\frac{1}{N}\sum_{n=1}^{N}K_{n}^{cep} \\
\overline{S}^{cep}=\frac{1}{N}\sum_{n=1}^{N}S_{n}^{cep}
$$
​		式中，$n=1...N$，$N$为活跃语音帧的数量，活跃语音帧满足$E_n>E_{minSpeech}$。

​		由此产生的倒频谱偏度$\overline{S}^{cep}$和峰度$\overline{K}^{cep}$可用于**描述语音信号的失真程度**。结果值较小，在0到1之间表示语音信号的高度退化（无序噪音的倒谱接近正态分布？未论证）。未失真信号的典型值在2到4之间。

#### 2.2.2	Calculation of vocal tract parameters for unnatural voice detection计算用于非自然声音检测的声道参数

在这个模块中，人类声道被建模成由多个等长的不同截面积的管子串联而成的系统（**声管模型**）。在短时间内，声道可表示为形状稳定的管道，并可以认为声波是沿管轴传播的平面波。根据语音信号，这些横截面面积被确定。然后对这些面积进行非自然变异分析。

##### 2.2.2.1	Vocal tract model extraction声道模型提取

​		带基音标记的语音信号流**允许在可变的时间坐标上从语音的浊音部分提取相关参数**。这比固定帧分析更能准确可靠地表示语音，这比固定帧长分析更能准确可靠地表示语音。这也意味着声道模型提取与分析模块是和语音波形产生同步进行的系统因为人类声道形变和语音波形同步变化。

​		声道参数提取步骤如图所示：

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 8-P.563 – Vocal Tract Parameters extraction overview.png"/>

###### 2.2.2.1.1	Speech frames语音帧

​		**LPCs不是在固定帧长和固定间隔上逐帧计算，而是在基音同步帧上执行**，重叠率为50%。这里定义一个可变语音帧长总是包括两个连续的基音标记即可变语音帧的长度等于两个连续的基音标记之间距离的两倍。变长语音帧在第一个基音标记之前半个基音周期处开始，并在第二个基音标记之后半个基音周期结束。

​		下图是一个关于可变帧划分示例，假设现在有三个连续的基音标记，分别为A,B,C

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 9-P.563 – Pitch synchronous LPC frames.png"/>

###### 2.2.2.1.2	Linear Prediction Coefficients (LPC) calculation 线性预测编码LPC计算

​		对之前定义的每一"帧"**提取八阶线性预测系数**。在计算自相关函数之前加汉明窗，采用**Schur递归法**（没具体了解过）得到预测系数。

###### 2.2.2.1.3	Tube section area管界面面积

​		8根声管的截面积Sm，用**反射系数$\mu$**计算，计算公式如下:
$$
S_{m}=\frac{1+\mu_{m}}{1-\mu_{m}}S_{m+1}
$$
​		计算每个基音周期的声管截面，得到的矩阵称为VTP，它代表从声门到嘴唇的声道管截面。

2.2.2.1.4	Cavity tracking腔跟踪

​		VTP信息被简化为3个参数，分别代表**前腔、中腔和后腔**。为了生成这些参数，VTP按以下方式分组:

| Cavity | Tubes |
| :----: | :---: |
|  Rear  | 1,2,3 |
| Middle | 4,5,6 |
| Front  |  7,8  |

​		每个基音标记处的结果存储在一个名为ART(articulators构音器官)的数组中。

##### 2.2.2.2	Basic VTP descriptors基本VTP描述符

###### 2.2.2.2.1	VTPMaxTubeSection

​		这个参数是整个输入信号上第一个VTP管的最大截面尺寸。

###### 2.2.2.2.2	FinalVtpAverage

​		该参数代表了最后一个VTP管的平均截面尺寸。

###### 2.2.2.2.3	ARTAverage

​		该参数是后腔的**平均截面面积**。

###### 2.2.2.2.4	ConsistentArtTracker

​		这个参数描述了**后腔和中腔之间的相互关系**。对于每个基音帧，计算后腔和中腔的截面面积差。然后计算生成的阵列中平滑截面的长度。当连续的元素值变化不超过0.25时，截面被认为是一致的。然后，将这个长度在找到的截面数以及基音数上求平均。

###### 2.2.2.2.5	VTPPeakTracker

​		这个参数**跟踪声道内的振幅变化**。对于每一帧数据，找到八管中最大的截面位置。通过计算该位置阵列的导数来确定其变化量，然后对这个变量数组取平均值，这个变量数组可描述声道内的振幅变化。

###### 2.2.2.2.6	VtpVadOverlap

​		该参数计算整个语音中，**浊音段在语音部分（活跃语音段包含浊音以及声带不振动的清音等）的比例**。

#### 2.2.3	Unnatural periodicity parameters不自然的周期性参数

##### 2.2.3.1	Pitch frames correlation基音帧相关性

​		这两个参数是基于浊音段的连续帧的，**而帧的构建类似于基音同步LPC帧**(参见2.2.2.1.1帧的构建).

- PitchCrossPower是通过计算连续两帧之间的互功率来确定的。计算每个互功率估值的**峰均值比**，然后对峰均值比数组取平均。
- PitchCorrelationOffset计算为连续帧的互相关。对于每对帧，计算**最大值位置到帧中心电的距离**，然后取平均距离。

##### 2.2.3.2	Robotization机器人音

​		含有太多周期性的的语音信号被称为**机器人音**，这些信号大多是由于GSM网络中使用的带宽限制造成的。

​		要将信号分类为机器人音，只有在2200hz和3300hz之间的信号成分才有用。利用信号在这个范围内的分量，通过32毫秒长度的相邻信号帧的互相关性计算周期性。在这个过程中，下一步将帧分类为非静默帧和静默帧以及周期性帧和非周期性帧：**若帧的功率超过$10^6$，则一个帧是非沉默的**；**若其各自的互相关超过0.84，且$L_1$范数至少为2.52，则呈现周期性**。

​		如果**非静默帧中周期帧的百分比大于3.4%**，则声明这至少3.4%的周期帧为机器人帧来计算机器人化模块输出值。

##### 2.2.3.3	Frame repeats帧重复

​		该模块被训练用于检测信号中的重复帧。该算法利用了重复帧通常具有的**高互相关特性**。算法的详细描述如下图所示：

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 10-P.563 – Evaluation of frame repeats.png"/>

​		模块的输出有两个值：

- FrameRepeats：该值表示当前信号中检测到的**帧重复次数**。
- FrameRepeatsTotEnergy：这是所有检测到的**重复帧的能量总和**。

##### 2.2.3.4	Unnatural beeps异常的哔哔声

​		当一个信号被发出时，它可以用一个复合音来表示。然而，一个自然发出的声音总是包含**一个持续很短时间的浊音部分**。在这个信号描述符中，发声开始时不包含浊音分的持续时间短的复杂音调被称为异常的哔哔声。

​		非自然哔哔声的检测器考虑的是持续时间为32毫秒、总长度为160毫秒、位移为16毫秒的时间信号帧的周期特性。周期性本身是由频率范围在250到1200赫兹之间的光谱不平直度推导出来的。如果**周期性在160毫秒内上下起伏**（不满足一致性特性），则判定该160ms语音为一种非自然的哔哔声，模块输出值计算如下：

- UnnaturalBeeps：该值表示检测到的非自然哔哔的数量乘以1000，再除以处理过的采样点数量。
- UnnaturalBeepsMean：被发现包含哔哔声的帧的平均能量和。
- UnnaturalBeepsAffectedSamples：包含哔哔声的帧的采样点的总和。

#### 2.2.4	Basic voice quality基本语音质量

​		基本语音质量模块在**心理声学模型**的帮助下**评估波形失真的影响**，下图显示了**语音增强器和基于感知的侵入式语音质量测量**两个功能块的相互作用原理。

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 11-P.563 – Scheme for calculation of basic voice quality.png"/>

##### 2.2.4.1	The speech enhancer语音增强器

​		语音增强器可以分为三个逻辑模块，如下图所示。

1)	**lpc分析**：使用Levinson-Durbin算法对时间信号进行分析，得到信号的残差和10个lpc系数。

2)	**声道模型**：对lpc系数进行修改，使之适合人类说话者的声道模型。

3)	**lpc合成**：将残差与修改后的lpc系数再次结合，重建语音增强过的时序信号。



<center><img src="https://pangcong1117.github.io/myblog/images/Figure 12-P.563 – Scheme of the Speech Enhancer.png"/>

##### 2.2.4.2	Intrusive perceptual speech quality measurement – Basic voice quality侵入式感知语音质量测量-基本语音质量

​		侵入式感知语音质量测量是一种复杂的测量方法。因此，这里没有提供完整的描述，读者可以参考C源代码获得详细的描述。下面两张框图给出了算法的概述。提出了感知模型的核心和基本语音质量的最终确定方法。

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 13-P.563 – Overview of the perceptual model.png"/>

<center><img src="https://pangcong1117.github.io/myblog/images/Figure 14-P.563 – Overview of the perceptual model – Integration part.png"/>
###### 2.2.4.2.1	FFT window size

​		选取长32ms (8 kHz采样率)的汉宁窗，使用短时傅里叶变换将时间信号映射到频域，帧重叠率为50%。

###### 2.2.4.2.2	Absolute hearing threshold

​		听觉的绝对阈限是指人的听觉系统感受到的最弱声音和痛觉声音的强度值，与声压有关。阈限以外，人耳感受性降低以致不能产生听觉。绝对听力阈限$P_{0}(f)$通过插值得到，并以此计算出Bark（巴克）域上24个临界频带的中心频率值，这些值存储在一个数组中，用于Zwicker响度公式。

###### 2.2.4.2.3	The power scaling factor功率比例因子

###### 2.2.4.2.4	The loudness scaling factor响度缩放因子

###### 2.2.4.2.5	Computation of the active speech time interval活跃语音间隔计算

​		语音文件如果开头和结尾处有较长时间的静默间距，这可能会影响文件中某些平均失真值的计算。因此，在这些文件的开头和结尾对静默部分进行了估计。从伪增强语音文件的开始和结束，五个连续的绝对采样值的总和必须超过500，以便将该位置视为活动语音间隔的开始或结束。开始和结束之间的时间间隔被定义为活动语音时间间隔。为了节省计算时间或存储空间，一些计算可以限制在活动时间间隔内。

###### 2.2.4.2.6	Short-term Fast Fourier Transform短时快速傅里叶变换

###### 2.2.4.2.7	Calculation of the pitch power densities计算基音功率密度

Bark域尺度反映出，在低频时，人类听觉系统的频率分辨率比在高频时要高。这是通过对FFT带进行归一化，并将FFT的相应功率值相加来实现的，所得到的信号称为基音功率密度PPXWIRSS(f)n和PPYWIRSS(f)n。

###### 2.2.4.2.8	Partial compensation of the pseudo-enhanced pitch power density for transfer function equalization根据传输函数均衡性，对伪增强的基音功率密度进行局部补偿

###### 2.2.4.2.9	Partial compensation of the distorted pitch power density for time-varying gain variations between distorted and pseudo-enhanced signal根据失真和伪增强信号之间的时变增益变化，对失真的基音功率密度进行局部补偿

###### 2.2.4.2.10	 Calculation of the loudness densities 计算响度密度

​		在对滤波和短期增益变化进行部分补偿后，利用兹威克定律将伪增强型和劣化的基音功率密度转换到Sone响度尺度：
$$
LX(f)_{n}=S_{i}*(\frac{P_{0}(f)}{0.5}))^{\gamma}*[(0.5+0.5*\frac{PPX^{'}_{WIRSS}(f)_{n}}{P_0(f)})^\gamma-1]
$$
​		式中，$P_0(f)$是绝对听觉阈限，$S_l$是响度收缩因子（2.2.4.2.4）

在Bark域4以上，兹威克幂是0.23。4以下，兹威克幂略有增加，以解释所谓的补偿效果。由此产生的二维阵列$LX(f)_n$和$LY(f)_n$称为响度密度。

###### 2.2.4.2.11  Calculation of the disturbance density计算扰动密度

​		计算失真和伪增强响度密度之间的差值。当这个差值为正时，说明一些成分如噪声就被加入了。当这个差值为负时，说明伪增强信号中的部分分量被忽略，这种差分阵列称为原始干扰密度。

​		计算每个时频单元伪增强和伪退化的响度密度的最小值，并用最小值乘以0.25。相应的二维数组称为掩模数组。以下规则适用于每个时频单元：

- 如果原始干扰密度为正且大于掩模值，则从原始干扰中减去掩模值。
- 如果原始干扰密度介于正负掩模值之间，则干扰密度设置为零。
- 如果原始干扰密度大于掩模值与-1的乘积，则掩模值加到原始干扰密度上。

###### 2.2.4.2.12	  Cell-wise multiplication with an asymmetry factor

###### 2.2.4.2.13	 Aggregation of the disturbance densities over frequency and emphasis on soft  parts of the pseudo-enhanced signal

###### 2.2.4.2.14	 Aggregation of the disturbance within split second interval

###### 2.2.4.2.15	  Aggregation of the disturbance over the duration of the speech signal, including a recency factor

###### 2.2.4.2.16  Computation of the Basic Voice Quality基本语音质量的计算

​		最终的输出值(BasicVoiceQuality)是平均干扰值(BasicVoiceQualitySym)，不对称扰动值(BasicVoiceQualityAsym)和频率范围在20到170Hz内的单位为db的平均功率谱密度值比较值(FractionLow)的一个线性组合。基本音质的范围是1到11。

该模型的输出值为：

- basicvoicequality：该值表示对声音干扰的估计。
- basicvoicequalityasym：该值等于非对称帧扰动的积分值。

### 2.3	Description of the functional block 'Additive Noise'功能块“加性噪音”的描述