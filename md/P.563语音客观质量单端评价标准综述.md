# P.563标准综述

​        p563是一种应用于窄带通信的话音客观质量单端评价标准

### 1.介绍

​        p.563算法无需另外一段参考信号即可对话音进行客观评估，因此被广泛用于无参考音的话音质量评估如未知信号源的电话通信远端。
​        相较于双端语音评估标准p862，仅基于对比计算出的有局限性的一套参数如无声段的能级或者噪声来计算话音质量，p563算法是一种无参考话音质量衡量标准。其着眼于公用电话网络中全方位的失真情况来预估话音的质量，根据ITU-T的p.800.1标准得到基于感知刻度的mos-lqo(Mean Opinion Score – Listening Quality Objective)值。
​        这套算法不受限于端到端的限制，可被用于传输信道的任意位置，但仅适用于3.1khz的窄带电话通信应用。

### 2.应用场景

​		在很多情况下，主观语音评估的方法无法实施，所以使用p563方法通过计算客观话音听觉质量MOS-LQO来预测MOS-LQS，通过映射得到的mos主观评分区间为1~5。

主要应用场景有如下三种：

- Live network monitoring using digital or analogue connection to the network

  使用数字或模拟接口连接到网络的实时网络监控

- Live network end-to-end testing using digital or analogue connection to the network

  使用数字或模拟接口连接到网络的实时网络端到端测试

- Live network end-to-end testing with unknown speech sources at the far end side

  在远端对未知语音源进行实时网络端到端测试

### 3.被证明可准确衡量的因素

经过试验论证，P.563算法计算出的客观语音质量分可准确衡量下列情况，并被建议使用

1. Characteristics of the acoustical environment声环境特性
2. Environmental noise at the sending side发送端环境噪声
3. Characteristics of the acoustical interface of the sending terminal发送端的声接口特性
4. Remaining electrical and encoding characteristics of the sending terminal发送端的其余的电气和编码特性
5. Speech input levels to a codec编解码器的语音输入电平
6. Transmission channel errors信道错误
7. Packet loss and packet loss concealment with CELP codecs基于CELP编解码器的丢包与丢包隐藏
8. Bit rates if a codec has more than one bit-rate mode一个编解码器有一个以上的比特率模式情况下比特率的影响
9. Effect of varying delay on listening quality in ACR tests 绝对等级评价（ACR）中不同延迟对听力质量的影响
10. Short-term or long-term time warping of speech signal语音信号短时或长时扭曲变形
11. Transmission systems including echo cancellers and noise reduction systems under single talk conditions and as they will be scored on an ACR scale传输系统包括回声消除和降噪系统在单一通话条件下的影响将会在ACR测试中衡量

### 4.被证明不可准确衡量的因素

1. Listening levels, Loudness loss响度损失
2. Sidetone侧音，通常指在终端设备（例如电话机）中，发端信号经处理后，其中一部分回馈到自身接收电话的那部分信号
3. Effect of delay in conversational tests会话测试中延迟的影响
4. Talker echo讲话人回声
5. Music or network tones as input signal音乐或网络铃声作为输入信号

### 5.被测话音信号需要满足的条件

​        p563方法被设计仅用于被电子设备（如麦克风）录制的人类话音，不能被用于音乐，噪声，人工制造语音或者其他无话音语音信号的评估，其他使用场景即限制，详见2、3、4节。

​		其他条件见表1：

| Requirements                  | threshold value     |
| ----------------------------- | ------------------- |
| Sampling frequency            | 8000Hz              |
| Amplitude resolution          | 16 bit linear PCM   |
| Minimum active speech in file | 3.0s                |
| Maximum signal length         | 20.0s               |
| Minimum speech activity ratio | 25%                 |
| Maximum speech activity ratio | 75%                 |
| Range of active speech level  | –36.0 to –16.0 dBov |


<center>表1：Requirements on speech signals to be assessed</center>

### 6.参考文献

ITU-T Recommendation P.48
ITU-T Recommendation P.800
ITU-T Recommendation P.810
ITU-T Recommendation P.830 
ITU-T Recommendation P.862
ITU-T Pseries Recommendations
ITU-T Recommendation P.563

#### 附录：

相关专业名词：

ACR	  Absolute Category Rating
CELP	Code-Excited Linear Prediction
dBov	dB to overload point
DCME   Digital Circuit Multiplication Equipment
ERP	Ear Reference Point
HATS	Head and Torso Simulator
IRS	 Intermediate Reference System
LPC	Linear Prediction Coefficient
MOS	Mean Opinion Score
MOS-LQO	Mean Opinion Score – Listening Quality Objective
MOS-LQS	 Mean Opinion Score – Listening Quality Subjective
PCM	Pulse Code Modulation
SNR	 Signal-to-Noise Ratio
SPL	  Sound Pressure Level





PS：下一个博客将介绍p563相关模块原理及其算法实现

