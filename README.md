# RCNN-Finetune-tutorial
- 在colab环境下跑通
- 这份文档主要源码集中在dataset的设置，其中box的设计很有用
- 模型设计上，直接使用了torchvision的models，即pretrained的模型
- 模型训练上，也用了engine模块中的train和evaluate，可以借鉴
- 模型结果上，这里只用了二分类判断，没有进一步的分割

- 11.13 在服务器上跑通，torch1.5.0 + torchvision0.6.0 + cuda10.1