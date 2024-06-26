---
layout: post
title:  Transformer in PyTorch
date:   2023-07-23 17:35:17
description: study notes of transformer in pytorch. references see below.
tags: programming
categories: mix-posts
---

# Table of Content
- [Table of Content](#table-of-content)
- [Overview](#overview)
- [Torch model summary](#torch-model-summary)



# Overview
PyTorch official example [here](https://pytorch.org/tutorials/beginner/transformer_tutorial.html).

Transformer [Doc](https://pytorch.org/docs/stable/generated/torch.nn.Transformer.html), 
TransformerEncoder [Doc](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoder.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/transformer.html#TransformerEncoder);
TransformerEncoderLayer [Doc](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoderLayer.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/transformer.html#TransformerEncoderLayer);
Multi-Head Attention [Doc](https://pytorch.org/docs/stable/generated/torch.nn.MultiheadAttention.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/activation.html#MultiheadAttention);
Dropout [Doc](https://pytorch.org/docs/stable/generated/torch.nn.Dropout.html). 



# Torch model summary
[pytorch-summary](https://github.com/sksq96/pytorch-summary)
    
```python
from torchvision import models
from torchsummary import summary

vgg = models.vgg16()
summary(vgg, (3, 224, 224))
```

```shell
----------------------------------------------------------------
        Layer (type)               Output Shape         Param #
================================================================
            Conv2d-1         [-1, 64, 224, 224]           1,792
              ReLU-2         [-1, 64, 224, 224]               0
            Conv2d-3         [-1, 64, 224, 224]          36,928
              ReLU-4         [-1, 64, 224, 224]               0
         MaxPool2d-5         [-1, 64, 112, 112]               0
            Conv2d-6        [-1, 128, 112, 112]          73,856
              ReLU-7        [-1, 128, 112, 112]               0
            Conv2d-8        [-1, 128, 112, 112]         147,584
              ReLU-9        [-1, 128, 112, 112]               0
        MaxPool2d-10          [-1, 128, 56, 56]               0
           Conv2d-11          [-1, 256, 56, 56]         295,168
             ReLU-12          [-1, 256, 56, 56]               0
           Conv2d-13          [-1, 256, 56, 56]         590,080
             ReLU-14          [-1, 256, 56, 56]               0
           Conv2d-15          [-1, 256, 56, 56]         590,080
             ReLU-16          [-1, 256, 56, 56]               0
        MaxPool2d-17          [-1, 256, 28, 28]               0
           Conv2d-18          [-1, 512, 28, 28]       1,180,160
             ReLU-19          [-1, 512, 28, 28]               0
           Conv2d-20          [-1, 512, 28, 28]       2,359,808
             ReLU-21          [-1, 512, 28, 28]               0
           Conv2d-22          [-1, 512, 28, 28]       2,359,808
             ReLU-23          [-1, 512, 28, 28]               0
        MaxPool2d-24          [-1, 512, 14, 14]               0
           Conv2d-25          [-1, 512, 14, 14]       2,359,808
             ReLU-26          [-1, 512, 14, 14]               0
           Conv2d-27          [-1, 512, 14, 14]       2,359,808
             ReLU-28          [-1, 512, 14, 14]               0
           Conv2d-29          [-1, 512, 14, 14]       2,359,808
             ReLU-30          [-1, 512, 14, 14]               0
        MaxPool2d-31            [-1, 512, 7, 7]               0
           Linear-32                 [-1, 4096]     102,764,544
             ReLU-33                 [-1, 4096]               0
          Dropout-34                 [-1, 4096]               0
           Linear-35                 [-1, 4096]      16,781,312
             ReLU-36                 [-1, 4096]               0
          Dropout-37                 [-1, 4096]               0
           Linear-38                 [-1, 1000]       4,097,000
================================================================
Total params: 138,357,544
Trainable params: 138,357,544
Non-trainable params: 0
----------------------------------------------------------------
Input size (MB): 0.57
Forward/backward pass size (MB): 218.59
Params size (MB): 527.79
Estimated Total Size (MB): 746.96
----------------------------------------------------------------
```