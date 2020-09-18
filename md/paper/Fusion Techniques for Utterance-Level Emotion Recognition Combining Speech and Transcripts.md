### 语音情感识别领域-论文阅读笔记1

## Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts

## 融合语音和文字的句段级别情感识别技术

本文是对interspeech会议论文“Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts”的阅读笔记，该文章是多模态情感识别领域的好文章，使用了语音和文本两种模态数据，深度学习网络为LSTM和CNN。

下载地址https://www.isca-speech.org/archive/Interspeech_2019/pdfs/3201.pdf

### 1.摘要（Abstract）

​		在人类认知和理解过程中，大量种类不同、彼此互不的不同模态线索被接受。人类交流中的各种情绪状态反映出不同模态线索的多样性。多模态情绪识别最近的进展是利用基于不同种类特征如文本、音频、视频图像的深度学习技术来实现卓越的表现。这篇论文着眼于将跨模态融合技术运用于情感识别的深度学习网络，使用的模态数据为说话人话音和相应的文本记录。

​		作者研究了长短时记忆循环神经网络（LSTM）使用预训练的词向量（pre-trained word embedding）来进行基于文本的情感识别以及卷积神经网络（CNN）使用话语级别(Utterance-Level)特征描述符来进行基于话音的情感识别。各种融合策略实施在这两种模型上来为每一种情绪类别给出一个综合评分。每一种情绪的模态内部（intra-modality）动态信息被捕获在为特定模态设计的神经网络中。融合技术被设计用于获取跨模态（inter-modality）动态信息。对于这类模态内部联系和跨模态联系的理解可参考论文“*Dynamic Fusion with Intra- and Inter-modality Attention Flow for Visual Question Answering*“并参考下图。注：下图并非本论文的图，只是用来帮助理解inter-和intra-modalitt。

<center><img src="https://pangcong1117.github.io/myblog/images/Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts图1.jpg"/>


​		作者在IEMOCAP多模态情感识别数据库上进行说话人与独立会话实验（ Speaker and session-independent experiments）来论证（show）本文提出方法的有效性

关键词：emotion recognition，multi-model，fusion techniques，deep learning

一些名词：

frame-level：帧级别

Utterance-Level：话语级别（段级别），也可以理解成句子级别或者段级别，是一个人说的一段话，比frame级别高。

dialog-level：对话级别，是两个人的对话，情感识别利用自身和两个说话人之间的关系来识别情感。

​		本文方法的流程图见下图1：

<center><img src="https://pangcong1117.github.io/myblog/images/Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts图2.png"/>

### 2.文本特征提取（Emotion Recognition from Text）

这一部分主要将特征提取和提出的基于文本的情感识别神经网络框架。特征提取模块为每个语句（utterance）提供了一种表示，其上下文依赖关系在LSTM的神经网络中建模。

#### 2.1特征提取（Feature Extraction）

采用CNN卷积神经网络从话语的转录文本（utterance transcriptions）中提取特征。基于特征提取的神经网络如CNN会学习出输入句子的抽象表达，这些语句中包含有基于单词和单词概率的语义。使用一个带有卷积层和最大池化层的简单CNN网络作为特征提取器。

CNN的输入形式是300维的词向量。这些300维的词向量是基于Fast-Text词嵌入提取的。（简单来说，每个英文单词都会用一个300维词向量表示）。卷积层包含三个卷积核，尺寸分别为f1，f2，f3，同样的，有三个输出通道。我们使用这些卷积核执行一维卷积，然后对其输出执行最大池化（maxpooling）。池化后的特征最终被投影到维度为D~T~的稠密层上，其经过激活函数后的向量被用作文本表示T∈$R^{D_T}$。

#### 2.2LSTM循环神经网络（LSTM RNN framework）

架构包含一个LSTM层和三个全连接层。循环网络中的连接获取了上下文信息来对话语给出的文本进行分类。这有利于情感标签分类，因为连续的单词为情感分类提供了额外的线索。每个话语（utterance）D~T~维的特征数据被喂到time step=N~1~的LSTM层。全连通层的隐含层节点数N2、N3和N4的数量是递减的，最后一个（N~4~）的是情感标签类别的数量。

### 3.话音特征提取（Emotion Recognition from Speech）

在特征提取阶段对每个话语进行声学特征提取，这些声学特征将被用于构建CNN进行情感识别。这个CNN模型叫做联合CNN模型，因为模型的输入是初步融合（early fusion）后的数据，这些数据具有更好的性能。

#### 3.1特征提取（Feature Extraction）

话音信号的特征提取使用到了openSMILE工具箱。interspeech13年的挑战赛上提供了包含6373维静态特征的特征集，称为ComParE特征集，可以通过openSmile开源包来获得。数据集包括LLDs和HSDs。特征集的详细描述可参见这篇描述“https://www.cnblogs.com/liaohuiqiang/archive/2018/12/22/10161033.html”

- LLDs（low level descriptors）LLDs指的是手工设计的一些低水平特征，一般是在一帧语音上进行的计算，是用来表示一帧语音的特征。
- HSDs（high level statistics descriptors）是在LLDs的基础上做一些统计而得到的特征，比如均值，最大值等等。HSDs是对utterance上的多帧语音做统计，所以是用来表示一个utterance的特征。

在这些特征中，我们进行*min-max*标准化 和基于L2范数的特征选择，将特征维数降低到D~s~ 。这样低维度的基于话音的特征S∈$R^{D_{s}}$被用于后续输入。

#### 3.2CNN网络框架（CNN Framework）

用于获取语音情感分类的神经网络由两个ReLU激活的卷积层组成，每个层之后是一个最大池化层。然后是三个全连接层。每个卷积层都有N~f~数量的卷积核（filters），每个卷积核的宽度为N~w~。卷积层进行单位步长（unit stride）的卷积操作，用于学习情感类别。将第二卷积层的输出扁平化（flattened）后，将其送入两个大小分别为N~c2~和N~c3~的全连接层。最后输出层的尺寸为情感类别标签的数量。

- Flatten层：用来将输入“压平”，即把多维的输入一维化，常用在从卷积层到全连接层的过渡。Flatten不影响batch的大小

第2节中介绍的基于lstm的模型也可以用于语音。我们观察到，它也提供类似的性能。尽管如此，本工作还是坚持使用CNN进行语音，因为它的融合性能比基于LSTM的系统要好，可能是因为采用了完全不同的建模方法。

### 4特征融合技术（Fusion Techniques）

该方法结合各模型输出特征用于后期融合（Late Fusion），并将一开始的文本和声学特征连接用于早期融合（Earlt Fusion）。

#### 4.1早期融合（Early Fusion）

早期融合是一种很常见的融合技术。在特征级融合中，我们将通过文本和语音的特征提取阶段得到的特征信息结合起来。话语（一段语句utterance）的最终输入表示是
$$
U_D=tanh((W^f[T;S]+b^f))
$$
第三部分提到的CNN网络的卷积层输入就是尺寸为$U_D=(T;S)$的特征向量，这个特征向量就是从早期融合中获得的。这有助于捕获在同一模式下文本和语音之间的模态间动态。我们称这个CNN网络为joint-CNN。

#### 4.2后期融合（Late Fusion）

​		作者在文本单模态lstm情感识别网络和联合cnn情感识别网络的输出中考虑了三种类型的决策级融合（decision-level fusion）。我们考虑了联合模型与特定模态模型的后期融合。除了在联合cnn模型中捕获到的跨模态动态联系（inter-modality dynamics），后期融合还能捕获到更多有用的跨模态动态信息。

##### 4.2.1后期融合I（Late Fusion-I）

这种决策级的融合是通过结合多个系统的得分来实现的。如果在相似的特征空间上建立不同的模型，则使用求和组合规则，如集成方法。这样给定一段语句的平均融合输出分数为：
$$
Score=\frac{Score(T)+Score(S,T)}{2}
$$
后期融合①对来自基于lstm的文本情感识别网络的输出和联合cnn模型的输出具有相同的偏好。（简单的决策级得分融合，也可调整权重）

##### 4.2.2后期融合II（Late Fusion-II）

在该方法中，我们使用基于lstm的文本情感识别模型的输出类别概率与联合cnn模型的输出类别概率根据不同权重值进行后期融合。对给定话语的输出得分加权平均融合后为：
$$
Score=\omega_{1}* Score(T)+\omega_{2}*Score(S,T)
$$
其中，$\omega_1,\omega_2\leq1$，且$\omega_1+\omega_2=1$。根据验证数据的性能，采用试错法确定权重。

##### 4.2.2后期融合III（Late Fusion-III）

输出概率也可以使用乘积规则进行组合：
$$
Score_j=\frac{Score_j(T)*Score_j(S,T)}{\sum_{j'}^{}(T)*Score_{j'}(S,T)}
$$
其中，$j'\neq j$且j代表情感标签分类的下标。基本假设是，特征空间是不同的，并且分类条件独立。这是非常有用的，因为我们正在构建一个特定领域的模型，并将其与使用不同特征表示的jointCNN相结合。采用后期融合技术会进一步改善了联合cnn模型表现出的跨模态动态特性。

### 5实验评估（Experimental Evaluation）

#### 5.1数据库（Dataset）

多模态情感识别任务在IEMOCAP数据库上进行。它是一个数据库，包括音频、视频、文本转录和动作捕捉数据，这些数据来自10对演讲者之间的互动。这是多模态情绪识别任务中最常用的数据集之一。受试者参与情感互动，共分为五个阶段，每个阶段有一对受试者。这些视频被分割成带有情绪标签的话语（句段），这些注释包括9个分类和3维度标签。IEMOCAP数据集有大约12个小时的口语内容，是该任务最大的公开可用数据集之一。我们将情绪分为六大类(愤怒、快乐、悲伤、中性、兴奋和沮丧)。每段录音至少有3个标注者进行标注。实验使用的是大多数人对情绪标签有一致看法的录音。

#### 5.2实验步骤（Experimental Procedure）

我们考虑六个情绪类别，以保持与最近的文献中的多模态情绪识别一致。现有文献大多是基于语音的情感识别，只考虑其中的四个类别(不包括兴奋和沮丧)。然而，由于情绪类之间的混淆程度较大，以及情绪类的分布变化，6分类模型的性能会下降。一些多模态系统只考虑四到五类。所提出的系统和基线在相似的评估(四类分类任务)下优于同类系统，因此本文不将它们包括在内进行比较。

训练和验证数据是使用由8个发言者在120个视频(5810个段话语)组成的前4个会话创建的。第五个会话，包括31个视频(1623个段)用于测试。通过这种方法，我们可以确保测试的演讲者和会话没有被训练好的模型训练过。验证集被选为训练数据的20%。

基于文本和语音的情感识别的超参数细节和所提方法的其他参数如表所示。对于单模态和双模态架构，Adam优化器的初始学习率为损失为0.001，loss函数用的是交叉熵损失（cross entropy loss）。训练过程中CNN进行了20个epoch，LSTM进行了40个epoch，用的是早停法（Early Stopping）。

- 早停法（Early Stopping）是一种被广泛使用的方法，在很多案例上都比正则化的方法要好。其基本含义是在训练中计算模型在验证集上的表现，当模型在验证集上的表现开始下降的时候，停止训练，这样就能避免继续训练导致过拟合的问题。

验证损失（泛化损失 The validation loss）的监测耐心系数（可以理解为一个阈值）为6。LSTM层的dropout参数设置为0.2用于进行正则化。文本特征提取使用单层CNN，其超参数如表所示。LSTMs和CNNs使用Keras工具包实现。

通过实验来和先进的多模态情感识别系统进行比较，观察在具有相似的输入特征表征，相同的分类标签的数目和类型，以及相似的评估方案时的实验结果。在文献中，多模态情感识别通常采用加权正确率(WA)、非加权正确率(UA)或F1评分(F1)进行评价。我们将所有这些措施考虑在内与其他文献方法进行了公平的比较。

<center><img src="https://pangcong1117.github.io/myblog/images/Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts图3.png"/>

#### 5.3基线系统（ Baseline Systems）

我们比较了所提出的方法与最先进的语言水平系统的性能。此外，还考虑了对话级系统，以分析除了句段级特征之外，所提供的上下文信息对该系统性能的影响。

1. Tensor fusion network张量融合网络(TFN)：这是一种基于融合的方法，明确地模拟模态内和跨模态间动态联系。单模态、双模态和三模态交互被聚合在一个特别设计的融合层和推断层中。
2. Memory fusion network记忆融合网络(MFN)：该方法利用德尔塔记忆注意网络（delta-memory attention network）融合机制来实现多模态序列学习（the multi-view sequential learning）。该模型为语音句段水平多模态情绪识别系统提供了最先进的的研究结果。作者考虑将TFN和MFN作为我们的基线系统，尽管它们还使用了语音和文本之外的视觉特征。
3. Bi-directional contextual LSTM双向上下文长短时记忆神经网络 (cLSTM)：这是一个对话级情感识别模型，它通过使用独立的LSTM对上下文的单模态和多模态特征进行分层建模来对句段进行分类。最终决策也受到邻近句段的影响。
4. Interactive conversational memory network交互式对话存储器网络（ICON）：这种方法使用全局记忆对自我和说话者之间的影响（相互作用与关系）进行分层建模。它由自影响模块、动态全局影响模块和多跳内存组成。这个对话级模型提供了最先进的对话级情感识别。然而，像ICON这样的对话级系统需要句段的历史记录，这在实时人机交互中是不容易得到的。

#### 5.4实验结果（Result）

表2显示了不同融合技术下所提方法的性能，其参数如表1所示。通过加权精度(WA)和F1评分来评价平均分类性能。在IEMOCAP数据库的中，有384句是“中性”情绪，而只有144句是“快乐”情绪。非加权精度(UA)计算特定情绪类别召回的平均值，对IEMOCAP等不平衡数据集有利。

早期融合是对单模态系统的改进措施。单模态系统的后期融合比早期融合捕获更多的模态间动态联系。早期和后期融合的组合进一步提高了性能。joint-CNN和LSTM系统的加权和组合提供了最佳的非加权精度，而这些系统的输出组合提供了最佳的总体精度。在所有系统中，支持度（support）最小的“happy”类别的类级性能最差，而支持度最大的“sad”类别的类级性能最优。在“高兴”和“兴奋”这两个类别中，混淆率最高为33%。

本文提出的方法与表3中的基线系统进行比较。这两种提出的方法在所有性能度量上都优于TFN。最后的Fusion-III方法表现优于最先进的基线系统(MFN)的所有评估指标。值得注意的是，所提出的方法只使用文本和语音模态，而基线也使用视觉特征进行分类。然而，提出的融合方法在基线系统上更有提高。

我们将对话级的情感识别模型纳入比较，即ICON和cLSTM，以了解我们的句段级系统在表现上的差异。这也如表3所示，该融合方法的性能优于同样使用视觉特征的cLSTM融合方法。

基于输出规则的融合结果以极小差距位于ICON模型下方。这表明，最先进的句段水平的表现对于对话水平情感识别模型是有竞争力的。实验评估验证了通过选择决策级的输出组合模型，捕获跨模态间的动态信息，可以获得最先进的句段级情感识别效果。

<center><img src="https://pangcong1117.github.io/myblog/images/Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts图4.png"/>

<center><img src="https://pangcong1117.github.io/myblog/images/Fusion Techniques for Utterance-Level Emotion Recognition Combining Speech and Transcripts图5.png"/>

### 6总结（Conclusions）

作者提出了一种新的深度学习模型融合技术，以改进多模态场景中的情绪识别。基于文本和语音模型的早期和后期融合技术的组合利用了跨文本和话音内容的互补信息。当在标准的基准标记数据集上评估时，它们可以达到最先进的段级识别性能。该模型在决策过程中充分利用了说话人自身和说话人其他因素的影响，其性能接近于现有的最好的多模态情感识别模型。这表明适当的建模和融合方法是多模态情感识别的一个很好的方向。未来的研究将集中于结合视频片段特有的视觉特征和模型，通过融合来帮助决策。

