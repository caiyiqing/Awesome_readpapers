# Counting with Focus for Free

### 简单介绍
文章认为点注释比仅仅构建密度图更能提供更多的监督，因此重复利用原有的点注释，建立人头分割分支和全局密度分支，协助原有基础网络生成高质量的密度图。通知改进解码器-编码器网络，让其更适合于人群计数。

### 本文贡献：
在传统以密度图为监督的方法中，我们的贡献为
1. 我们利用提供注释的点，分割出可用于监督的关注点，用于引导计数网络感兴趣的区域。
2. 新建立整体密度估计分支，对网络进行正则化，让网络能学习到匹配整体密度的估计
3. 提出一个新的核密度估计器用于非均匀点分布的点注释。
4. 对于深度网络设计一种改进的编码器-解码器网络来处理不同尺度的对象。为（了避免过拟合将第四级之后的网络参数减少至98、并加入蒸馏器层整合4、5、7、8级网络的特征）。

### 整体网络结构
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/1.jpg">
</center>
顶部分支：用于分割出点注释
底部分支：用于估计全局密度

### 一、	Focus from segmentation
1、分割地图
重新利用注释点创建二元分割图，并利用该图通过独立的损失功能提供全面的监督。
二进制分割图的确立方法。训练图像i中每个像素p的二进制值被确定为：
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/2.jpg">
</center>
如果至少一个点P在由核估计器指定的方差范围σP内，则像素p获得值1。

2、焦点网络的损失
（1）点损失：改进的交叉熵损失，实验表明当R=2时效果最好(2017ICCV)，a用来平衡重要性，
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/3.jpg">
</center>
在本文中引用为：
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/4.jpg">
</center>
前景分支的细节过程是：
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/5.jpg">
</center>
本文以沿第一个维度的第二个像素的值表示为前景一部分的概率

### 二、Focus from global density
1、全局密度图
<br>在全局密度中：文章使用双线性池化层来捕获全局上下文中的特征统计，已知这种特征统计特别适用于纹理和细粒度识别，对于训练图像i中的图像块，其全局密度表示为：
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/6.jpg">
</center>
<br>Pij表示图像i的图像块j中点注释的数量，L表示全局密度步长
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/7.jpg">
</center>
<br>Zi和Zji表示图像i和图像块j中的像素，密度步长L：表示全局图像块中最大的密度，M表示使用多少的全局密度水平。
2、全局密度图的损失：
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/8.jpg">
</center>
3、分支网络细节：
<br>双线性模型，是在图像的每个位置使用外积，再池化获得图像描述符，该架构可以以平移不变的方式对局部成对特征相互作用建模，这对于细粒度分类尤其有用
<br>引用的文章模型：
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/9.jpg">
</center>
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/10.jpg">
</center>
<br>该文中的外积：是自己和自己的转置外积，再通过平均值池化生成双线性向量。先外积、然后平方根归一化、L2正则化、求平均值。
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/11.jpg">
</center>

### 三、Focus from global density非均匀核估计
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/12.jpg">
</center>
<br>其中w和h是确定点注释P-中心局部区域R的范围的超参数，并且我们将它们的值设置为我们的实验中的图像大小的八分之一。 a表示位于R中的任意点注释。| R（w，h）|是指p的数量。 d¯p表示注释点p与其k个邻近邻居之间的平均距离，β是用户定义的超参数。 通过局部估计点注释的方差，我们不再需要假设点在整个图像上均匀分布。

### 四、架构与优化
#### 优化的解码器编码器网络
  文章采用改进的解码器-编码器架，加入了蒸馏器模块，该模块的目的是通过提取最重要的计数信息来聚合来自编码器的多级信息，解决透视失真下对象尺度的广泛变化问题。改进网络如图所示：
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/13.jpg">
</center>
改动：
* 将级别4之后的特征映射的通道从256/512更改为96，为了避免过拟合、 并方便融合
* 蒸馏器模块通过使用跳过连接和串联操作来融合编码器模块中的4,5,7和8级特征
* 不融合6级特征的原因是6级包含具有大扩张率的卷积层，这很容易引起网格化艺术
* 为了避免常规反卷积操作引起的棋盘格伪影问题，在每个去卷积层之后添加两个卷积层
* 网络的输入和输出大小相同。

#### 多级损失：联合L1和L2泛化损失
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/14.jpg">
</center>

### 训练细节

* 预处理:通过将所有值除以255来标准化输入RGB图像
* 在训练期间，我们通过随机裁剪128×128个图像块来增强图像
* 设备：GTX 1080 Ti GPU、使用TensorFlow、mini-batch=16的小批量
* β1设置为0.9，将β2设置为0.999并将初始学习率设置为0.0001。
* 训练在最多1000个纪元后终止。
* 即ShanghaiTech A部分和TRANCOS，我们使用我们提出的内核，β= 0.3和k = 5
* 对ShanghaiTech B部分和DCC，我们将高斯核方差设置为σ= 5和σ= 10
* 对ShanghaiTech A部分使用M = 8密度级别，对其他数据集使用4密度级别。

### 总的实验效果
<center>
  <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/CFF_Focus_for_Free/15.jpg">
</center>
