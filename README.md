# Alibaba-Cloud-German-AI-Challenge-2018

这是天池大数据的比赛<br/>
https://tianchi.aliyun.com/competition/introduction.htm?spm=5176.100067.5678.1.3e7731f5WP7NmY&raceId=231683<br/>



1.数据集描述：<br/>
数据集分为训练集和验证集，图像包括sen1,sen2两部分，其中sen1是sar图像,分为实部和虚部,有两个通道是滤波处理的。sen2是高光谱图像<br/>

特点：<br/>
a.由两个卫星的图层叠加而成，可以视为32,32,18的图片<br/>
b.测试集和验证集均出现类别分布不平衡情况，可以视为多标签不平衡分类问题<br/>
c.sen1图层的像素波动很大（从负数到上千），sen2图层像素多为0-1之间的浮点数<br/>

2.训练模型（分为预处理，特征提取，模型训练，后处理4个模块，因为各个模块难度不同，从简单往难的做, []表示待执行的步骤）<br/>

模型训练进展：<br/>
网络采用了L_Resnet_E_IR, 损失函数基于softmax entropy<br/>
[2]<br/>

预处理模块：<br/>
1.对于数据分布不平衡的问题，采用重复过采样，保证训练时的数据分布平衡<br/>
[4]<br/>

特征提取模块：<br/>
对sen1,sen2做特征处理，参考blog http://blog.sina.com.cn/s/articlelist_1984634525_4_1.html<br/>

后处理模块：<br/>
[5]<br/>

todo list:<br/>
[2]尝试不同激活函数和损失函数<br/>
[4]图像归一化处理，镜像堆对称，随机裁剪，提取中心和四角的子图片x5<br/>
[5]先利用神经网络学习特征，然后获取神经网络最后一层的向量，利用传统分类器，如GBDT,LightGBM来分类<br/>

1.利用训练集，迭代70000步，在训练集上达到过拟合（拟合度100%），在验证集上面准确率在60%左右<br/>
2.训练期间，实现early stopping，34000步时达到最优，此时对于训练集拟合度在90%，验证集准确率61%<br/>
3.结合1，2，可以看出训练集过不过拟合对于验证集的分类没有太大影响<br/>
4.提交在验证集上性能最优的模型（61%准确率），线上测试集效果不到60%<br/>
5.从测试集和验证集准确率近似，提出猜想，测试集和验证集的分布近似，所以决定利用验证集来辅助模型训练<br/>
6.将训练集和验证集融合起来进行训练，达到过拟合（100%）后提交结果，线上效果达到75%<br/>
7.训练集为train+val, 针对数据的不平衡情况，采用了权重采样处理，保证每个batch的迭代中各个类别的分布比例平衡，采用early stopping，线上达到77.9%<br/>
---------------------------------------------------------------------------------------12.6日进展，线上排名前5%<br/>
8.构造数据集：从val训练集里面拆分3000个样本出来构造val21000和val3000，保证类别分布一致
9.train+val24000做训练集，val3000做验证集，将18个通道每个通道训练一个神经网络，利用投票记数法，将18个网络模型进行集成，线上72%<br/>
10.train+val24000做训练集，val3000做测试集，将18个通道每个通道训练一个神经网络，根据在每个模型在各个类别上的performance构建权重矩阵(18* 17)，对18个网络模型进行集成，线上74%<br/>
11.5层神经网络，val24000做训练集，只用sen2, val3000上达到92%，线上78.1%<br/>
12.L-Resnet-E-IR网络，val24000做训练集，只用sen2, val3000上达到94%，线上79.4%<br/>
13.利用线上77%，78%，79%的结果进行加权投票，线上81.7%<br/>
---------------------------------------------------------------------------------------12.28日进展，线上排名前113/1300<br/>
