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



# Overview
PyTorch official example [here](https://pytorch.org/tutorials/beginner/transformer_tutorial.html).

Transformer [Doc](https://pytorch.org/docs/stable/generated/torch.nn.Transformer.html), 
TransformerEncoder [Doc](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoder.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/transformer.html#TransformerEncoder);
TransformerEncoderLayer [Doc](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoderLayer.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/transformer.html#TransformerEncoderLayer);
Multi-Head Attention [Doc](https://pytorch.org/docs/stable/generated/torch.nn.MultiheadAttention.html), [Source](https://pytorch.org/docs/stable/_modules/torch/nn/modules/activation.html#MultiheadAttention);
Dropout [Doc](https://pytorch.org/docs/stable/generated/torch.nn.Dropout.html). 