---
author: "zhpu AI"
title: "CogVideoX:A Cutting-Edge Video Generation Models"
date: 2024-07-26
draft: false
thumbnail: /cogvideox.png
---

As a highly complex system, human cognitive function relies on the collaborative work between various regions of the brain, which involves not only the processing of text and language, but also multiple aspects such as visual understanding and auditory processing.

We firmly believe that the integration and improvement of perception and understanding in Multi-modal Learning is closely related to the development of cognitive abilities.

As a company dedicated to achieving Artificial General Intelligence (AGI), ZhipuAI has always attached great importance to the development of MultiModal Machine Learning technology. Since 2021, the ZPAI technology team has been laying out MultiModal Machine Learning models including text-2-img, text-2-video, img-2-text, and video-2-text, and has successively developed and open-sourced multiple advanced models 
such as CogView, CogVideo, Relay Diffusion, CogVLM, and CogVLM-Video.

Here, we are pleased to announce that the video generative model has been upgraded and the new generation product - CogVideoX has been officially launched.

The core technological features of CogVideoX are as follows:

Addressing the issue of content coherence, Zhipu AI has independently developed an efficient 3D Variational Autoencoder (3D VAE) structure. This structure is capable of compressing raw video data to 2% of its original size, significantly reducing the training cost and difficulty for video diffusion generation models. Combined with the 3D RoPE position encoding module, this technology effectively enhances the capturing ability of frame relationships over time, thereby establishing long-term dependencies within videos.
In terms of controllability, Zhipu AI has created an end-to-end video understanding model that can generate precise and content-related descriptions for a large amount of video data. This innovation strengthens the model's comprehension of text and its ability to follow instructions, ensuring that the generated videos are more aligned with user input requirements and can handle overly long and complex prompt instructions.

Our model adopts a transformer architecture that integrates text, time, and space into a single three-dimensional fusion. This architecture abandons the traditional cross-attention module and innovatively designs an Expert Block to achieve alignment between the text and video modalities. It further optimizes the interaction effects between modalities through a Full Attention mechanism.

CogVideoX has been officially launched on Zhipu Qingyan's PC, mobile application, and mini-program platforms. All C-end users can experience the AI video generation feature "Ying"（清影） for free, which offers services for AI text-to-video and image-to-video generation through Zhipu Qingyan. (Link: https://chatglm.cn/video)

The main features are as follows:

1. **Quick Generate**: Generate a 6-second video in just 30 seconds. Efficient instruction following ability: Even complex prompts, Qingying can accurately understand and execute.
2. **Content coherence**: The generated video can better restore the motion process in the physical world.
3. **Picture scheduling flexibility**: For example, the lens can smoothly follow the three dogs in the picture, just like a professional photographer's follow.

In addition, we have also deployed "Ying" (清影) on the Zhipu Large Model Open Platform bigmodel.cn. Enterprises and developers can experience and use the text generation video and image generation video functions of "Ying"（清影） through API calls.

**DEMO**

**Text-to-Video**

**Example 1**

**Prompt**

Low-angle forward movement, slowly looking up, a dragon suddenly appears atop the iceberg, and then the dragon notices you and charges toward you. Hollywood movie style.

**Generated**

{{< youtube fVetKRMMT64 >}}


**Example 2**

**Prompt**

A mushroom transforms into a little bear.

**Generated**

{{< youtube yYhZpq3LoVE >}}

**Example 3**

**Prompt**

Little yellow duck toy floating on the water in the swimming pool, close-up

**Generated**

{{< youtube 9GxJaMSEKnA >}}


**Example 4**

**Prompt**

Falling snowflakes, a little bird playing on the branch.

**Generated**

{{< youtube XLBzjfm_cXk>}}

