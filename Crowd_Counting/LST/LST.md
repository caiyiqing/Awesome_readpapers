### 网络的整体结构
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/LST/1.jpg">
</center>

#### 建立了目前最大的视频数据集[FDST](https://github.com/sweetyy83/Lstn_fdst_dataset)
#### 亮点：
<br>通过分块来建立前后两帧的关系
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/LST/2.jpg">
</center>
<center>
    <img src="https://github.com/caiyiqing/Awesome_readpapers/blob/master/Crowd_Counting/LST/3.png">
</center>
<br>前后两帧的关系由损失函数来限定。S是相似性，越相似权重越小表明变化的越小。

