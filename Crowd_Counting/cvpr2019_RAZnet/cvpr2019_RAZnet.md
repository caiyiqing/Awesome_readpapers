# Recurrent Attentive Zooming for Joint Crowd Counting and Precise Localization
[[论文连接]](https://paperswithcode.com/paper/recurrent-attentive-zooming-for-joint-crowd)用于联合人群计数和精确定位的循环周期缩放
## 简单介绍
提出一个新奇的框架，能够同时解决两个内在联系的问题，人群计数和定位

## 动机
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/1.jpg">
</center>
传统上计数一直是当前研究的中心焦点，很少考虑对每个人进行精确定位。如图l(b)和(d)所示，虽然计数提供了关于每个人位置的微弱信息，但是通过锐化密度图来精确推断出每个人的位置通常仍然是不可行的，可用于人群轨迹分析、行人重识别等。

## 贡献点
1. 提出了一个新的定位深度网络。并认为传统的MSE不能给予网络稀疏激励的Loss，引入二元交叉熵损失的归一化变量，并设计了一个学习注意力模型的网络分支，它提示最可能需要放大的图像区域，可重复放大区域的子区域。
2. 模型的定位和计数相互促进，模型是multi-branched，每个branch分别count或locate，最后做自适应加权和。
3. 定义了新的评估指标。

## 相关工作
初看计数和定位联合的文章，介绍一下相关工作：
* Detection based methods:低密度是有很好的效果：DecideNet用detector来修正低密度区域（通常高估）的计数
* Density based methods: 最近的发展考虑了空间局部性[22]跨尺度聚集[3]和尺度自适应[50]，多分支网路：包括Switch-Net、Divide-and-Grow Net
* Localization in congested scenes: 最近的发展是预测人群图像中的斑点、UCF-QNRF dataset

## 方法
1. 整体的网络结构
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/2.jpg">
</center>

* counting branch
计数分支网络和CSRnet一模一样，VGG16的前13层+一些dilated conv layes（膨胀率为2），输出大小是原分辨率的1/8
loss有一些差别

* localization branch
输入定位分支的图像是原输入的1/8，因此先通过3个反卷积将其上采样到原来的分辨率，每个反卷积提高一倍分辨率，然后最终输出是和原图分辨率相同
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/3.jpg">
</center>
用K=[0 1 0；1 1 1 ；0 1 0],得到ground truth map,loss 用BEL loss ，计算如下式
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/4.jpg">
</center>
为了提高定位准确度，先用3X3 stride=1 的平均卷积用，来提高尖点压缩噪声。然后，用NMS避免检测点过近。也可以3X3的maxpooling。

* Two-Stream Fusion for Counting:
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/5.jpg">
</center>
鉴于随着人群密度增大，两个branch的准确率都减低了，但location branch 相对更差。换句话说，对于高密度区域，基于密度的估计更可靠，应该主导计数融合。
每个图片partition成4X4的subimages，并采用公式4 这种策略来计算总数。

* Zooming Region Proposal Branch
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/6.jpg">
</center>
cat 以上两个branch的预测结果，用了穷举搜索方案，进行高密度区域的定位。方法与Faster R-CNN类似，先随机生成几百个注意力点，然后在点上生成几个不同维度的archor，，并对每个子区域进行评分，降序排列，先删掉分数低于某个值的区域，剩下的区域通过NMS模型，最终只留下k个archor。在实践中,我们使用(m = 200, 0 z = 0.01和k = 10),锚有三个高宽比(1:1、1:1、2:1)和两个空间尺度(其中一个比另一个大两倍)。在较小的尺度下，1:1比例的锚点大小是图像较短边缘大小的1/3。
将选出来的区域喂到下面的RAZ-Net。

* Recurrent Attentive Zooming Net:
在注意力分支中被选出来的图像分辨率将被放大2倍放入RAZ中，
<br>这个Branch 不包含counting branch。
<br>这个Net的网络设置和权重与Main-Net一样，为了防止过拟合。
<br>此外RAZ-Net 可以不断的训练和使用。

### 实施细节
训练过程：counting branch -+ localization branch -+ zooming region proposal branch.
<br>使用30个 Titan X？
<br>最初将学习率设置为10-4，当验证性能在多次迭代中停滞不前时，学习率减半

### Evaluation Protocol
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/7.jpg">
</center>
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/8.jpg">
</center>
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/9.jpg">
</center>
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/10.jpg">
</center>
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/cvpr2019_RAZnet/11.jpg">
</center>

### 
