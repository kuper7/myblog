### 语音情感识别领域-论文阅读笔记2

## 个性化的注意力机制：一种使用个性化注意力档案（profile）的多模态情感识别网络

## Attentive to Individual：A Multimodal Emotion Recognition Network with Personalized Attention Profile

本文是对interspeech会议论文“Attentive to Individual：A Multimodal Emotion Recognition Network with Personalized Attention Profile”的阅读笔记，论文作者结合了多模输入，使用Attention机制优化不同属性说话人（例如老人、儿童等情感表现方式不同的人群）的Emotion识别效果。

下载地址

### 1.摘要（Abstract）	

越来越多的以人为本的应用得益于情感识别技术的不断进步。许多情绪识别算法已经被设计用于多模态行为线索建模，以达到更好性能。但是，其中绝大多数算法并没有考虑到个体的个人属性在其表达行为中的调控因素。在这项工作中，作者提出了一种个性化属性感知注意模型（Personalized Attributes-Aware Attention Network，PAaAN）。该模型引入一种新颖的个性化注意力机制来利用话音和语言线索进行情感识别。其中，注意力档案是从个人简介、声音和词汇行为数据的嵌入（embeddings这里的嵌入可以理解为用一个向量来描述这些个人数据）学习得到。这些档案嵌入是使用目标说话人对于大量电影剧本的LIWC（ linguistics inquiry and word count）推导出的。作者的方法在IEMOCAP数据库的四类情感识别任务中达到了70.3%的非加权准确率。进一步分析发现，在语料库中，每个说话人对情感相关语义类别的重视程度不同，这说明我们的注意机制对个性化的作用是有效的。

### 索引词（Index Terms）

personal attribute个人属性；multimodal emotion recoginition多模态情感识别；attention注意力；psycholinguistic norm心理语言学规范

### 一些专业名词

- profile：个人理解为用户数据/档案/配置。
- Personalized Attributes-Aware Attention Network，PAaAN：个性化属性感知注意力网络
- linguistics inquiry and word count（LIWC）：摘要中提到了这个名词，LIWC是一款软件，他实现的功能是LIWC朗读一段给定的文本，然后计算出反映不同情感、思维风格、社会关注等单词的百分比。作者使用LIWC的大致用意是针对说话人对反映不同情感单词的敏感度推导个性化的注意力档案，来提高情感识别模型对特定说话人的性能。



### 2.研究方法（Methodology）

#### 2.1数据库（Databases）

##### 2.1.1IEMOCAP情感数据库（The IEMOCAP Emotion Database）

在本论文研究中，作者使用了一个多模态的二元交互语料库IEMOCAP作为主要的情感识别评价数据库。它包括12小时的视听记录，有文字对齐并提供手写文本。总共有10位不同的演讲者配对表演，或是在假定场景照本宣读，或是即兴表演。总共有10039句话，每句话至少有三个评注。我们使用数据库的一个子集来与最近的多模态情感识别工作（对照组论文为：J. Cho, R. Pappagari, P. Kulkarni, J. Villalba, Y. Carmiel, and N. Dehak,“Deep neural networks for emotion recognition combining audio and transcripts,” Proc. Interspeech 2018, pp. 247–251, 2018.）进行比较。该子集包含了四个情绪类别的5531个样本(其中愤怒:1103，快乐:1636，悲伤:1084，中性:1708)。

##### 2.1.2. 背景影片剧本数据库（The Background Movie Script Databases）

我们的PAaAN将个性化的个人信息集成到识别框架中。IEMOCAP数据库只包含10位发言者，为了有效地提取每个发言者的个人档案，我们的框架利用了其他大型发言者语料库中的文字记录。在这项工作中，我们收集了两个额外的背景电影剧本数据库，每个都有大量的说话人。其中一个是康奈尔电影对白语料库（Cornell movie dialogs corpus），包括5531个不同角色的电影剧本。另一个是EmotionLines语料库，包括656位演讲者的台词，这些台词来自电视剧《老友记》和Facebook messenger的私人短信。两个语料库都包含与IEMOCAP类似的对话内容。

#### 2.2听觉与文字表征（Acoustic and Textual Representations）

##### 2.2.1听觉特征（Acoustic Features）

我们使用openSMILE工具箱提取了45维低水平特征描述符(llds)，包括12维梅尔频率倒谱系数(MFCCs)，基频(F0)，响度（loudness），人声概率（voice probability），过零率以及MFCCs和响度的一阶和二阶差分。这些llds提取使用60ms帧长度和10ms步长；应用zscore标准化。在我们的BLSTM中，每个时间步长（time step）对应4帧(40ms)，每个时间步长的输入是这四帧llds的平均值。

##### 2.2.2词嵌入（Word Embeddings）

实验中使用的文本是通过预先训练的GloVe word2vec模型来编码的，这个模型在420亿标记和190万个词汇上训练。将N个单词组成的句段（utterance）表示为一组词嵌入向量，$U=\lbrace\omega_1,\omega_2,...,\omega_n\rbrace$，其中$\omega$是词嵌入。在BLSTM模型的每个时间步中，每个单词都被编码为一个300维的向量。

#### 2.3个性化属性感知注意力网络（Personalized Attribute-Aware Attention Network）

完整的PAaAN体系结构如图1所示。它包括一个双模态BLSTM和深度神经网络（DNN）作为情感识别网络，同时使用声学和文本特征。每个BLSTM都与一个新的个性化档案注意力机制一起学习。我们将首先描述在BLSTM识别网络中计算个性化注意力所需的个人档案（profile）嵌入的来源。