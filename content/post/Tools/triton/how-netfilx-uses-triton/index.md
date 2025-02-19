---
title: "How Netflix uses Triton for model scoring service"
date: 2024-07-27
draft: false
summary : "Learings and Painpoints of using Triton"
tags: ["Triton"]
categories : ["Tools"]
series : ["Triton Inference Server"]
series_order : 3
slug : "how-neflix-uses-triton"
toc : true
katex : true
markup: 'mmark'
---

## Challenges with AI inferencing

There are many challenges when it comes to AI inferencing. 

1. **Fast** :  It needs to meet use case and user experience latency constraints. 
2. **Interoperable** : It needs to support multiple development frameworks and deployments.
3. **Integration** : It should integrate well with enterprise landscape.
4. **Scalable** : It should be able to easily add new models and support spikes in user demand.
5. **Efficient** : It needs to highly utilize underlying HW accelerators.
6. **Cost** : It needs to meet budget constraints and delivers max ROI.


But as we start to peeling the onion and look a bit deeper underneath these aspects, you start seeing some very intricate challenges that are particularly unique to AI inferencing. For example, when you talk about latency and speed, a lot of enterprises today are deploying not just a single model, but hundreds of models with each serving unique applications, that can have different service level agreements. For instance, applications that really meet the customer has to have very low latency, but applications that are in the back office might have lower priorities. So The ability to have inferencing service platforms, to identify these models and serve them based on their specific service level agreement(sla) becomes an interesting challenge to solve. 

Second is **interoperability**. A lot of models these days have different back ends, which could be pytorch, onnx, tensorflow, but also some of the models in productions can have associated pre and post processing requirements that might not necessarily need to run on accelerated hardware and can potentially run on CPU. So being able to have an inferencing platform serving capability that serves all these backends, but also all these different hardware accelerators(CPUs, GPUs) becomes an interesting challenge. 

On the **integration** side, this is particularly important when we think about AI as a lot of enterprises today are carving out significant portions of their budget to invest in AI applications, we're starting to see requirements around making sure that these investments can be easily ported across different deployment models. So some organizations might be standardized on AWS, and these organizations might be specifically opt to hire specialized in AWS. But the ability to quickly port these models and inference serving platforms to different clouds is important.

## Architecture of Triton Inference Server

Inference serving is a tool to help you take your machine learning models and deploy them at scale, putting them behind some kind of service. 

![ML-6284-image001](https://github.com/user-attachments/assets/d42bc44e-6092-4a78-8f90-e83abe9a9d3f)

In the model repository, you need to place models. Models could be in your local repository, S3, cloud storages like GCP. Triton takes that model, loads it. When a client sends a request, or even a curl from the command line, triton has all sorts of advanced scheduling  features per model queues to optimize the scheduling the request, scaling the number of instances per model based on popularity. If a certain model is more popular, you can load more instances of that, which is the ability to load models dynamically. Taking all these together, you get a http or grpc endpoint that you can query behind your service. 

## Netflix's Model Scoring Service with Triton

### Model Scoring Service overview

The model that Netflix is using is built internally, and it is gvm based. 

Typical model serving flow at Netflix is like this.

![image](https://github.com/user-attachments/assets/ebcd0216-9427-4b4c-b972-152993d3f4a2 'Typical model serving flow at Netflix')

The model here is not just the core machine learning model. It contains 4 stages of machine learnig, including data fetching, feature extraction, inference and ranking. 

With seperating the inference, the service looks like below.

![img](https://github.com/user-attachments/assets/df52ab72-06d6-4a35-88ad-5b849995b2c8 'Model Scoring Service')



### Triton Backend with Model Scoring Service

With Triton on the model scoring service, it is integrated into Model Scoring Service as a backend. This is a default choice for GPU inference. It supports different type of models, and has good cost efficiency for Netflix's main workload. Various levels that can be tuned for better performance, dynamic batching, quantization, etc. Tools are also available for debugging and perf analysis.  


### LLM serving with Triton

Netflix's benchmark results on December,2023 show that TensorRT-LLM has the best serving throughput for internal use cases. 


![img](https://github.com/user-attachments/assets/7ba07054-0a14-45d6-b536-951c2aa558b2 'LLM Serving with Triton')


After LLM is fine-tuned, it needs to be compiled into TensorRT-LLM engine, and then a yaml file will be generated during the completion. Than the **Model Lifecycle Management Service** sends deployment request which includes parameters like instance type, instance count, and yaml file location. The yaml file includes metadata such as engine repository location, tokenizer repository location. Then the **Model Deployment Service** deploys Model Scoring Service. This has GPU docker, and the Model Loader will first parse the yaml file and get the metadata. The engine files, token files, configuration files are downloaded so that Triton Backend can load them. 

Usually there are 4 models involved in LLM Model Serving. The **Preprocessing Model** does cleaning, tokenization, normalization, and prepares input to main language model. The **LLM Model** is core model for understanding input and generating natural language text. Than the **Postprocessing Model** will refine the output to meet specific requirements and improve outpu t quality. This could involve tasks like filtering out irrelevant information, rephrasing sentences for clarity, and adding formatting. Than the **Ensemble Model** connects these 3 models together. The Application can talk to Model Scoring Service with HTTP or gRPC. The proxy will make gRPC requests to Triton Backend.


### Learnings and Painpoints from using TensorRT-LLM backend

Although Triton is now being tested in the production environment because it is ongoing work, Netflix's benchmark results show that TensorRT-LLM has great thorughput. Triton provides multi-framework support(Tensorflow, PyTorch, TensorRT-LLM) that allows unified deployment and serving.

There are also painpoints. First one is that Triton image has the TensorRT-LLM and required dependencies installed, however the Netflix could not use the Triton image directly, because Netflix has own base image. So Netflix had to compile TensorRT-LLM, maintaing all cuda and relevant dependencies. This can work as a maintainance burden. Second is that there are too many params to configure at compile time. It is challenging to strike a balance between ease-of-use and optimal performance. Users can be left in the dark, about what parameter will effect in the runtime performance.  Lastly, TensorRT-LLM version has to be the same at compile time and runtime. It would be good to have backward compatility. 


## Reference 

- [NVIDIA Triton Inference Server and its use in Netflix's Model Scoring Service](https://www.youtube.com/watch?v=NR_iUl2Ooc0)