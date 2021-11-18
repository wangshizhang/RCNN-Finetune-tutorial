# RCNN-Finetune-tutorial
- 在colab环境下跑通
- 这份文档主要源码集中在dataset的设置，其中box的设计很有用
- 模型设计上，直接使用了torchvision的models，即pretrained的模型
- 模型训练上，也用了engine模块中的train和evaluate，可以借鉴
- 模型结果上，这里只用了二分类判断，没有进一步的分割

- 11.13 在服务器上跑通，torch1.5.0 + torchvision0.6.0 + cuda10.1

## 网络个人补充理解
### Faster RCNN
- 参考https://zhuanlan.zhihu.com/p/31426458
- 源码参考https://zhuanlan.zhihu.com/p/145842317
- RPN
    - 单纯生成Proposal的，但选取proposal的时候是参照anchor的score，并没有结合shift。
    - Proposal的选取是所有box的排序
- RoI Pooling
    - 解决box尺寸不固定的问题
    - 十分简单，对原始结构做pool就行，相当于downsample，不过是w,h的非等效downsample
        - 不过如果box的尺寸小于Classifier的尺寸咋办？
- Classifier
    - 除去分类外，又做了一次bbox的拟合->这是进一步用到了ImageNet预训练的信息
- 训练
    - 先训练RPN
        - RPN训练中，$p_i^*$是对应每个点的每个anchor和GT中最大的box去做IoU。
        - loss分两部分，一个是softmax probability的训练，一个是box的训练->针对的是RPN的最后输出
        - 相当于，feature map进来，每一点的anchor都有一个score，这个score对应一个GT box。那么，这个score是要训练，用来做判定这个anchor是否有用。最后生成的pred box是和anchor对应的（因为是通过score排序的），那这个pred box要和GT box形成一个loss做训练。loss分开写的原因，是为了把pred box所对应的GT box更好的找出来，并且更好的训练score。->相当于把找对应GT Box的任务放在anchor上了，简化一点。
    - 再训练后面分类+bbox
    - 多次迭代，最后联合训练
### Mask RCNN
- 参考 https://zhuanlan.zhihu.com/p/37998710
- ResNet + FPN
    - 一个多阶识别的结构。低阶（清晰的）会融合高阶（模糊的）信息
    - 输入到RPN的结构将是多层分辨率的情况，根据不同层级去切不同的ROI->ROI大，则低分辨率feature map。
- ROI Align->也就是Mask的体现
    - Faster RCNN的问题，ROI Pooling是非均匀的downsample，去整过程中会不对齐
    - 因此，相比较直接downsample，ROI Align是直接在原始浮点型框框中，切分完后，进行浮点特征的双线性插值。也就是根据坐标选取周边的点进行线性插值。那么downsample会更精确。
