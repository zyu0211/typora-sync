# 论文总结

------

## Frequency-aware Discriminative Feature Learning Supervised by Single-Center Loss for Face Forgery Detection

### 论文信息

作者：Jiaming Li *et al.*
单位：中国科学技术大学、快手科技
会议：CVPR 2021
论文链接：[https://arxiv.org/abs/2103.09096](https://arxiv.org/abs/2103.09096)

### 介绍

#### 背景

- 现有方法存在的问题：
  - learned features supervised by softmax loss are separable but not discriminative enough, since softmax loss does not explicitly encourage intra-class compactness and interclass separability.
    
    >softmax损失监督的学习特征是可分离的，但没有足够的区分性；因为softmax损失并没有明确鼓励**类内紧性**和**类间可分离性**。
  - fixed filter banks and hand-crafted features are insufficient to capture forgery patterns of frequency from diverse inputs.
    
    >固定滤波器组和手工制作的功能不足以从各种输入中捕获伪造频率模式。
#### 创新
- 为了弥补上述局限性，本文提出了一种频率感知鉴别特征学习框架:Frequency-aware Discriminative Feature Learning frame-work (**FDFL**)。
- **FDFL：**
  - 解决问题：
    - how to adopt metric learning to learn more discriminative features for face forgery detection.
      
      >如何采用度量学习来学习更多的鉴别特征用于人脸伪造检测。
    - how to adaptively extract frequency-related features.
      
      >如何自适应提取与频率相关的特征。
  - 组成：
    - 单中心损失：single-center loss (**SCL**)
      - 单中心丢失旨在减少自然人脸的类内变化，同时增加嵌入空间中的类间差异。SCL将真脸到中心点的距离最小化，同时SCL鼓励假脸到中心点的距离至少比真脸到中心点的距离大一个margin。但SCL不限制被操纵人脸的类内紧度，更符合被操纵人脸的特征分布特征。
    - 自适应频率特征生成模块：adaptive frequency feature generation module (**AFFGM**)
      - 数据预处理：使图像块在空间域的位置关系与在频率域的位置关系保持一致。
      - 自适应频率信息挖掘块(AFIMB)：以数据驱动的方式自适应地挖掘频率线索，避免使用太多不全面的先验知识。
### 方法
#### 总体框架
![](PaperNotes.assets\2022-06-14-15-27-36.png)
- 此框架同时从RGB域和频域提取特征，并在整个框架的早期阶段将其合并。经过特征嵌入后，得到了高层次的表示。框架的最后是一个分类器，用于输出样本的预测结果。

#### 自适应频率特征生成模块AFFGM
- 数据预处理
  ![](PaperNotes.assets\2022-06-14-16-06-24.png)
  - 步骤：
    - 首先，将输入的RGB图像转换为YCbCr颜色空间；
    - 接下来，对图像的每个8×8块应用2D-DCT变换；
    - 然后，将所有8×8块中来自相同频带的DCT变换系数分组到一个通道中，并保留其原始位置关系；
    - 最后，将所有频率通道串联在一起，形成一个张量。
    - 图像尺寸变化：$W×H×3 \to W/8×H/8×192 ，192=8×8×3$
-  AFIMB
  -  网络结构：
    <img src="PaperNotes.assets\2022-06-14-16-27-58.png" style="zoom:75%;" />
    * 首先，通过一层特殊的3×3卷积块(with three groups)，分别处理来自Y、Cb、Cr不同通道的数据；
    * 然后，依次经过一个普通的3×3卷积块和一个最大池层，来自Y、Cb、Cr不同通道的信息相互作用；
    * 然后，为了增强特征，采用上述最大池层和两个线性层组成的通道注意块；
    * 最后，使用普通的1×1卷积层进一步提取频率相关特征。
#### 单中心损失SCL
* 在SCL中，我们只设置自然人脸的中心点C，基于mini-batch在每次迭代时更新中心点C。
* SCL计算公式：
  $$L_{sc}=M_{nat}+max(M_{nat}-M_{man}+m\sqrt{D}，0) \tag{1}$$  
  
  $$M_{nat}=\frac{1}{|\Omega_{nat}|}\sum_{i\in{\Omega_{nat}}}||f_i-C||_2 \tag{2}$$ 
  
  $$M_{man}=\frac{1}{|\Omega_{man}|}\sum_{i\in{\Omega_{man}}}||f_i-C||_2 \tag{3}$$
  
  >* $M_{nat}$：自然面特征表示与mini-batch中心点C之间的平均欧氏距离；
  >* $M_{man}$：操作面特征表示与mini-batch中心点C之间的平均欧氏距离；
  >* $f_i$：表示第$i$个样本$x_i$经过网络$f(·)$后的特征向量表示；
  >* D：表示输出特征向量的维度。
* 总损失：$$L_{total}=L_{softmax}+\lambda L_{sc} \tag{6}$$ 

### 实验

#### 实验设置
* 数据集：FaceForensics++
* 评价指标：AUC、pAUC、Accuracy
* 实现细节：
  * 使用PyTorch框架实现；
  * 使用在imagenet上预先训练的XceptionNet作为主干网络；
  * 融合模块插入到XceptionNet的 entry flow 和 middle flow 之间；
  * 分类器是一个具有两个节点的简单FC层。
#### 消融实验
* 在FF++的c40版本上，不同损失函数的性能：
  ![](PaperNotes.assets\2022-06-15-13-34-58.png)  
* 在FF++的c40版本上，不同FDFL模块的性能：
  ![](PaperNotes.assets\2022-06-15-13-40-26.png) 
#### 与现有方法的比较
![](PaperNotes.assets\2022-06-15-13-44-58.png)
#### 总结
* 局限性
  * 缺乏对未知操作方法的泛化能力；
  * 忽略了帧间信息。

------


## Face Forensics in the Wild
### 论文信息
作者：Tianfei Zhou *et al.*
单位：苏黎世联邦理工学院、北京理工大学
会议：CVPR 2021
论文链接：[https://arxiv.org/abs/2103.16076](https://arxiv.org/abs/2103.16076)
代码链接：[https://github.com/tfzhou/FFIW](https://github.com/tfzhou/FFIW)
### 介绍
#### 背景
* 在多人视频中，通常在一个场景中会包含多个人，而只有一小部分人被操纵。
#### 创新
* 大规模数据集：$FFIW_{10K}$
* 人脸伪造检测框架：提出了一种**区分注意模型**(Discriminative Attention Model)，用于多人场景下的人脸伪造分类和定位。
  * 多时间尺度实例特征聚合模块(multi-temporal-scale instance feature aggregation module)
    
    >该模块总结每个人脸轨迹的短期、长期和全局特征，以获得鲁棒和可区分的表示。
  * 基于注意力的包特征聚合模块(attention-based bag feature aggregation module)
    
    >该模块将所有人脸轨迹的表示自适应聚合为视频级表示。
  * 稀疏正则化损失(a sparse regularization loss)
    
    >加强人脸选择的稀疏性，从而实现真/假鉴别。
### 方法
#### $FFIW_{10K}$数据集
* 组成：真假实视频序列：10000-10000。视频每帧有1–15个人脸，平均3.15个。
* 伪造方法：DeepFaceLab、FSGAN、FaceSwap。
* 标签：同时提供面部标签和视频级别标签。
* 数据集划分：16000个训练、500个验证、3500个测试。
#### Domain-Adversarial Quality Control
* 领域对抗性质量评估网络Q-Net(基于VGG16)：自动评估每个交换人脸的质量，用来滤掉低保真度的伪造人脸，方便数据集构建。通过Q-Net的伪造人脸只有得分大于0.6才会被保留下来。
  * 动机：对于大多数GAN模型，随着训练的继续，其合成图像的质量逐渐提高。
  * 过程：收集生成模型(StyleGAN、StyleGAN2、PGGAN)在不同迭代轮次中生成的人脸图像，并使用相应的迭代数作为质量分数。具体而言，对于每个生成的面$I_i$，分数为$s_i=0.9×n/ N$，n和N分别表示当前迭代次数和最大迭代次数。同时为每个训练样本分配域标签，即$d_i\in \{StyleGAN, StyleGAN2,PGGAN\}$。因此、训练集定义为$\{I_i, s_i, d_i\}$。
#### Face Forgery Detection Framework
* 总体框架
  ![](PaperNotes.assets\2022-06-16-12-47-44.png)
  
  * 每个视频$V$对应一个包，其包类别标签为$l_V\in \{fake,real\}$，一个包由$K$个未知标签的实例组成。
  * 每个实例表示为一个人物的面轨迹，即$\Gamma=\{\tau_1,...,\tau_T \}$， $\tau_i$ 为一张人脸。
  * 实例经过 ***ResNet-50*** 网络，提取特征：$X=[x_1,...,x_T] \in \mathbb{R}^{D \times T}$，$x_i$ 为对应的人脸图片 $\tau_i$ 的特征表示。
  * $X$经过 ***多时间尺度实例特征聚合模块***，得到紧凑表示$Y\in \mathbb{R}^D$，此时整个视频$V$可以表示为：$\{Y_k \}_{k=1}^K$。
  * $\{Y_k \}_{k=1}^K$通过 ***基于注意力的包特征聚合模块***，将所有实例特征自适应聚合为全局包级表示：$O_V\in\mathbb{R}^D$，用作最后分类器的输入。
* 多时间尺度实例特征聚合模块
  * 多时间尺度实例特征聚合模块$F()$包含三部分：
    * 短期聚合：$ S = F^s(X) \in \mathbb{R}^{D \times T}$
      * $F^s()$结构如上图绿色块所示，其中$F_l^{atr\_conv}$ 结构为$bn\to relu\to conv(1 × 1)\to bn\to relu\to conv(3 × 3, rl)\to bn\to conv(1 × 1)$
    * 长期聚合：$ L = F^l(S) \in \mathbb{R}^{D \times T}$
      * $F^l()$结构如上图红色块所示，其中
        $$A = softmax(S^\top S)\in \mathbb{R}^{T \times T}$$
        $$L = F^l(S) = SA + S \in\mathbb{R}^{D \times T}$$
    * 全局聚合：$ Y = F^g(L) \in \mathbb{R}^{D}$
      * $F^g()$为最大池化操作。
* 基于注意力的包特征聚合模块
  * 全局包级表示：
  	$$O_V=\sum_{k=1}^K a_kY_K \in \mathbb{R}^{D}$$ 

  	$$a_V = (a_1,...,a_K)\in [0, 1]^K$$ 

  	$$a_k=\frac{exp\{w^\top tanh(W^\top Y_k)\}}{\sum_{\acute k=1}^K exp\{ w^\top tanh(W^\top Y_{\acute k})\}} $$ 

  	此处$w$和$W$是可学习的参数。根据$a_V$可以定位被操纵的人脸，如果$a_k>0.75$，就认为第$k$个实例是伪造的。
* 损失函数
  * 损失函数定义为：
    $$L = L_{CE}(\hat l_V, l_V) + \beta L_{Sparsity}(a_V)$$
    其中$L_{CE}$表示二元交叉熵损失，$ L_{Sparsity}(a_V)=||a_V||_1$表示为$a_V$的$l_1$范数。
### 实验
#### 实验设置
* 数据集：FF++、DFDC Preview、Celeb-DF、 $FFIW_{10K}$
* 超参数设置：batch_size 32、Adam optimizer、learning_rate 1e-4、$\beta= 0.001$
#### 模型评估
* 在$FFIW_{10K}$的比较
  ![](PaperNotes.assets\2022-06-16-14-40-18.png)
* 跨数据集比较
  ![](PaperNotes.assets\2022-06-16-14-41-00.png)
* 消融实验
  ![](PaperNotes.assets\2022-06-16-14-41-34.png)
  
------


## Improving the Efficiency and Robustness of Deepfakes Detection through Precise Geometric Features
### 论文信息
作者：Zekun Sun *et al.*
单位：上海交通大学
会议：CVPR 2021
论文链接：[https://arxiv.org/abs/2104.04480](https://arxiv.org/abs/2104.04480)
代码链接：[https://github.com/frederickszk/LRNet](https://github.com/frederickszk/LRNet)

### 介绍
#### 背景
* 以前的Deepfakes视频检测工作主要集中在外观特征上，这些特征有被复杂的操作绕过的风险，也会导致模型的高度复杂性和对噪声的敏感性。
* 如何挖掘和利用被操纵视频的时间特征仍然是一个悬而未决的问题。
#### 创新
* 提出了一个高效而健壮的**LRNet**框架(Landmark Recurrent Network)，通过对精确几何特征的时间建模来检测深度伪造视频。
  * 几何特征选取为*Facial landmarks*。
  * 双流RNN网络：充分利用时间特征，提取landmark序列的深层时间特征。
  * 校准模块(calibration module)：提高几何特征的精度，通过减少抖动来增强几何特征的识别能力。
### 方法
#### 总体框架
  ![](PaperNotes.assets\2022-06-16-18-09-14.png)
#### 人脸预处理
* 人脸预处理模块从人脸图像中提取几何信息；它包括人脸检测、人脸标志点检测和标志点对齐。
* 步骤：
  * 首先，在视频的每一帧上执行人脸检测，并保留人脸的感兴趣区域(ROI)。
  * 然后，裁剪出面部图像，在其上检测到68个面部标志，这些标志勾勒出面部的标志性轮廓。
  * 最后，我们将地标点对齐到通过仿射变换实现的预设位置。
#### Landmark校准
* 我们使用**Lucas-Kanade光流算法**，在连续帧上，预测地标的下一个位置；并将有效预测与定制的**卡尔曼滤波器**的检测结果合并，以去除噪声并获得更高精度的校准地标。
* 过程：
  * Pyramidal LK operation：通过第i帧的地标点预测的i+1帧的地标点。
    ![](PaperNotes.assets\2022-06-16-19-18-17.png)
  * 定制卡尔曼滤波器：整合检测和预测的地标点信息。
  $$X_{i+1}^{opt}=X_{i+1}^{pred}+K_{i+1}(X_{i+1}^{det}-X_{i+1}^{pred})$$
#### 分类预测
* 将提取的地标序列嵌入到两类特征向量序列中，然后输入到双流RNN中进行伪视频分类。
* 特征嵌入
  * 第i帧的地标序列表示：$L_i = [X^1_i,... , X^{68}_i ]^{\top}$，$X_i^a=[x_i^a,y_i^a]^{\top}$
  * 第一类特征向量$\alpha_i$表示为：$\alpha_i = [x_i^1,y_i^1,...,x_i^{68},y_i^{68}]$
  * 第二类特征向量$\beta_i$表示为：$\beta_i = \alpha_{i+1}-\alpha_i=[x_{i+1}^1-x_i^1,...,y_{i+1}^{68}-y_i^{68}]$
* RNN网络
  * 通过嵌入，得到两个特征向量序列：$A = [α_1, ..., α_n]^{\top}$，$B = [β_1, ..., β_{n−1}]^\top$。
  * 使用两个RNN网络分别处理A、B，再分别连接全连接层预测各自结果，最终整个视频的预测结果是上述两个结果的平均。
### 实验
#### 实验设置
* 数据集：UADFV 、FaceForensic++、Celeb-DF、DeeperForensics-1.0
* 实现细节：
  *  landmark detection使用：Dlib；
  *  RNN网络：双向RNN，由GRU组成，输出单元数为64；
  *  全连接网络做为分类器，共两层，神经元数为：64、2；
  *  RNN网络之前，插入了一层dropout，另有三层dropout插入剩下的层中；
  *  Adam优化器，Lr=0.001，batch size=1024，epochs=500。
#### 模型评估
* 一般性评估(AUC)
![](PaperNotes.assets\2022-06-16-20-27-04.png)
* 鲁棒性评估
![压缩](PaperNotes.assets\2022-06-16-20-29-40.png)
![噪声](PaperNotes.assets\2022-06-16-20-30-28.png)
* 消融实验
![校准模块](PaperNotes.assets\2022-06-16-20-37-22.png)
![双流RNN](PaperNotes.assets\2022-06-16-20-37-42.png)
* 局限性：很难解释模型捕捉到的时间特征，也很难将真假人脸之间的运动模式差异可视化。

------


## Representative Forgery Mining for Fake Face Detection

### 论文信息
作者：Chengrui Wang *et al.*
单位：北京邮电大学
会议：CVPR 2021
论文链接：[https://arxiv.org/abs/2104.06609](https://arxiv.org/abs/2104.06609)
代码链接：[https://github.com/crywang/RFM](https://github.com/crywang/RFM)
### 介绍
#### 背景
* 基于CNN的检测器倾向于从有限的面部区域检测伪造，这表明检测器对伪造缺乏了解。
* 不同的伪造方法会在假脸的不同区域留下伪造痕迹，检测器应该更多地关注能够显著代表相应操作技术的伪造，而不是过度拟合整个训练集上的伪造。
#### 创新
* 本文提出了一种基于注意力的数据增强方法(RFM)，通过在训练过程中细化训练数据来解决注意力有限的问题。
* RFM(Representative Forgery Mining)
  * FAM(Forgery Attention Map)示踪方法：利用检测器的梯度生成图像级伪造注意图，可以精确定位面部敏感区域，并将其作为数据增强的指导。
  * SFE(Suspicious Forgeries Erasing)数据增强方法：有意遮挡面部前N个敏感区域，允许检测器从之前被忽略的面部区域探索具有代表性的伪造。
### 方法
#### 总体框架RFM
  ![](PaperNotes.assets\2022-06-16-22-02-38.png)
* 在训练期间，使用RFM的每个迭代需要前向传播两次、后向传播两次。

#### 伪造注意图FAM
* 在前向传播中，检测器接收人脸图像 I 作为输入，并输出两个logit $O_{real}$和$O_{fake}$来测量 I 是否为真。
* 因为任何扰动都会影响两个logit，所以检测结果应该由两个logit的相对大小来确定；通过利用$\nabla_IO_{real}$和$\nabla_IO_{fake}$单独表示 I 中的扰动如何影响logits输出。
* FAM 被视为$\nabla_IO_{real}$和$\nabla_IO_{fake}$的最大绝对差值：$$Map_I = max(abs(\nabla_IO_{fake} −\nabla_IO_{real}))= max(\nabla_I (abs(O_{fake} −O_{real}))$$
* FAM 中的每个值都精确地表示探测器对图像中相应像素的灵敏度。
#### 可疑伪造擦除SFE
* 首先，在mini_batch中为每个图像生成FAM；FAM和输入图像的大小都设为H×W。
* 然后，对于每个图像，根据相应FAM中的值按降序对坐标进行排序。
* 根据上面计算的顺序，将每个像素视为anchor。
* 对于每个anchor，使用随机整数形成一个矩形块$H_e\times W_e(H_e<H、W_e<W)$，对为遮挡的anchar进行遮挡，直到每张图像由N个块被遮挡。
### 实验
#### 实验设置
* 数据集：DFFD、Celeb-DF
* 评估指标：AUC(ROC曲线下面积)、 TDR at FDR of 0.01%、TDR at FDR of 0.1%
* 参数设置：
  * XceptionNet作为探测器的backbone。
  * Adam优化器，学习率为0.0002，batch_size=16，交叉熵损失函数。
* 预处理：
  * 首先将对齐的人脸图像调整为256×256的固定大小。
  * 然后，我们将随机裁剪和中心裁剪应用到训练和测试过程中，分别将图像大小调整为224×224。
  * 此外，我们在训练期间以50%的概率水平翻转每个图像。
#### 模型评估
* 消融实验
  ![](PaperNotes.assets\2022-06-16-23-25-20.png)
* 与其他擦除方法的比较
  ![](PaperNotes.assets\2022-06-16-23-26-36.png)
* 总结：
  * 本文方法可以很容易地与各种CNN模型集成，而无需额外的结构修改和复杂的训练集。
  * 且能够使基于普通CNN的检测器在不修改结构的情况下实现最先进的性能。

------
