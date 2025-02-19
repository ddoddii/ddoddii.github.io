+++
author = "Soeun"
title = "BERT Practical deployment on NVIDIA GPU"
date = "2024-08-01"
summary = "Deep into Triton Inference Server"
categories = [
    "CS"
]
tags = [
    "Triton",
]
slug = "bert-deployment-with-triton"
series = ["Triton Inference Server"]
series_order = 4
+++

{{< alert icon="edit">}}
This posting is a summary of [GTC 2020 presentation](https://www.youtube.com/watch?v=cKf-KxJVlzE&t=51s), with personal thougts added.
{{< /alert >}}

## Deep Learning 


![img](https://github.com/user-attachments/assets/728c776a-47c0-4777-b84f-18e439d7f89f 'Deep Learning')

At a very high level, deep learning is divided into steps. First step is training. You have a very deep network, you have a huge amount of data and you train the network to find the values for the weights of the networkk. For instance, the classical example is age classifiction. You train with images, and then once you computed the weights, you can do inference. During inference you use the weights used in the previous phase. So we will focus on the inference side, 


![img](https://github.com/user-attachments/assets/b63646ef-2566-458d-bbf6-e9c83361276a 'Why we use GPUs for inference')

In the inference part, you need to have programmability, low latency, accuracy. You also have to deal with high network, and want high throughtput, efficiency rate of learning. So all this is delivered by using GPUs.  

### Challenge during inference

Usually, in the traditional inference system you basically have one GPU dedicated to run inference on a particular model so you have one GPU for speech recognition you have one GPU for recommendation system and another GPU for language processing. The problem is that sometimes you have spikes in one application so do you get you get your one GPU vary at 100% utilization and the others are left unused or underutilized. 


## Case Study