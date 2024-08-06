---
title: " CogVideoX-2B: China's open source sora "
date: 2024-08-06
draft: false
thumbnail: /CogVideoX-2B.png
---

With the continuous development of large-scale model technology, video generation technology is gradually maturing. Technologies represented by closed-source video generative models such as Sora and Gen-3 are redefining the future pattern of the industry. However, as of now, there is still no open-source video generative model that can meet the requirements of commercial-grade applications.

Zhipu AI adheres to the concept of "serving global developers with advanced technology" and announces the open source of CogVideoX, a video generative model with the same origin as "Qingying", in order to allow every developer and every enterprise to freely develop their own video generative model, thereby promoting the rapid iteration and innovative development of the entire industry.

CogVideoX open source model contains a number of different sizes of models , we will open source CogVideoX-2B , it inference in FP-16 precision only 18GB memory, fine-tuning only requires 40GB memory, which means that a single 4090 graphics card can be inferred, and a single A6000 graphics card can complete fine-tuning.

CogVideoX -2B has a maximum of 226 tokens for prompt words, a video length of 6 seconds, a frame rate of 8 frames per second, and a video resolution of 720 * 480. We have reserved a broad space for improving video quality and look forward to developers contributing to open source efforts in prompt word optimization, video length, frame rate, resolution, scene fine-tuning, and various feature development around videos.

Models with stronger performance and larger parameters are on the way , please pay attention and look forward to it.1

Code repository: https://github.com/THUDM/CogVideo

Model download: https://huggingface.co/THUDM/CogVideoX-2b

Technical Report: https://github.com/THUDM/CogVideo/blob/main/resources/CogVideoX.pdf



## 

![alt text](/img_v3_02dg_dad99112-a43a-4e4e-82cc-c28243c1095g.png)

