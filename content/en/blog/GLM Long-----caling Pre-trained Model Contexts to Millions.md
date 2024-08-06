---
title: "GLM Long: Scaling Pre-trained Model Contexts to Millions"
date: 2024-07-18
draft: false
thumbnail: /GLMLONG.png
---

>In early 2023, even the most advanced GPT-3.5 had a context length of only 2k. However, today, a context length of 1M has become one of the important indicators for measuring the technological advancement of models.

If LLM is compared to the operating system of the new era, the context window is its "memory". A modern operating system requires sufficient memory to complete various complex tasks. Similarly, an excellent LLM also requires sufficient context length to complete various complex tasks.

Based on this concept, the GLM technical team has undergone continuous technological iterations, from the initial ChatGLM-6B that only supports 2K context, to ChatGLM2-6B (32K), ChatGLM3-6B (128K), and now GLM4-9B (1M), always pursuing the most advanced context technology capabilities. Especially GLM4-9B-Chat-1M, which integrates a large amount of research results in the field of long text.

This article will take the GLM4-9B series model as an example to introduce in detail the relevant technology of the GLM team to expand the context of pre-trained models to millions.

## 1. Review

Let's first look at the performance of GLM4-9B in four evaluations.

### 1.1 LongBench-Chat review

This is a 128K evaluation set with manually annotated questions and answers, which is more practical. GLM4-9B-Chat-1M can achieve performance close to larger parameter models.

![alt text](/640.png)

### 1.2 InfiniteBench

This review is a review set mainly for 100K-200K length, including 12 types of tasks (Minference only used 10) . In this review, the performance of GLM4-9B-Chat-1M is second only to GPT-4.

![alt text](/640-1.png)
**Data comes from official InfiniteBench and MInference papers*

### 1.3 Ruler
This evaluation is mainly used to evaluate the true context length of the model. It can be seen that the GLM4-9B-Chat-1M model still performs well even at 128K.

![alt text](/1.PNG)

**The data is from Ruler official, and the effective length refers to the length when the score is greater than 85.*

### 1.4 Finding a needle in a haystack

The needle in a haystack experiment, as the most famous experiment to evaluate LLM's ability to process long text information, involves randomly inserting a sentence unrelated to the text content into the long text, and then observing whether the model can accurately extract this hidden sentence from the text. As shown in the figure below, the GLM-4-9B-Chat-1M model conducted the "needle in a haystack" experiment under a context length of 1M, achieving the ability of lossless information processing.

![alt text](/640.webp)

## 2. Training Process

1M context capability is not achieved overnight, it needs to go through multiple stages to gradually activate and maintain the model's long text capability .

![alt text](/0_iK3h1lhjROCy6v6u.webp)

Furthermore, in the continuation of the pre-training, SFT, and RLHF phases, we carefully mixed the training to maintain the general ability of the model on short texts .

### 2.1 Continue pre-training

After pre-training with a massive amount of tokens, the Base Model has excellent information capture and inference capabilities. In order to generalize this ability to long text, we need to continue pre-training the Base Model with a small amount of long text tokens.

This step is crucial for stimulating the model's ability to handle long texts.

![alt text](/0_D9_m8EgMYY1HQhAr.webp)

Specifically, for the GLM4-9B-Chat-1M model, considering the huge span of directly jumping from 8K context to 1M, we adopted a two-stage pre-training strategy.

First, in the first stage, we expanded the context length to 128K; then, in the second stage, we expanded the context length to 1M.

Of course, for the GLM4-9B-Chat model, only the first stage of training is required.


#### Data

In order to maintain the existing general processing capability of the model while activating its ability to handle long texts, we have carefully mixed sampling strategies for the data that continues to be pre-trained.

Specifically, for the training data expanded to 128K in the first stage, its composition mainly includes the following parts:

- The pre-training data of the original distribution contains approximately 4B tokens.

- Based on pre-training data, upsample data with a length of over 8K, which is approximately 3B tokens.

- Based on pre-training data, upsampling is performed on data longer than 32K, which also contains approximately 3B tokens.

When processing these two types of upsampled data, we try to ensure that the total number of tokens in each length interval is consistent. For example, the total number of tokens in the 28K to 36K interval and the 60K to 68K interval should be similar. Under this balance principle, the number of data items in the 60K to 68K interval is approximately half of the number of data items in the 28K to 36K interval.

When constructing the second stage training data that extends to 1M, we encountered a challenge: in the pre-training corpus, data with original text length exceeding 1M is very rare and narrowly distributed, mainly from book materials. If we directly follow the method of the first stage training data, it will have a negative impact on the model performance.

To address this issue, we have introduced a new type of data - artificial long text data. This type of data is generated by concatenating existing data, including data from pre-training corpus and publicly available document data. To ensure the rationality of the concatenated data, we have designed a two-stage clustering strategy.

- First, we sample document data and classify it using a language model.
- Then, under each classification, we use the Embedding model to obtain the vector representation of the data.

Through this method, we ensured the balance of the total amount of tokens in the length range of 8K to 1M, thereby generating the required artificial long text data.

The final training data distribution is as follows:
- The pre-training data of the original distribution contains approximately 3.5B tokens.
- Based on pre-training data, upsample data with a length of over 8K, which is approximately 1.5B tokens.
- Based on pre-training data, upsample data with a length of over 32K, which is approximately 1.5B tokens.
- Based on pre-training data, upsampling is performed on data with a length exceeding 128K, which is also approximately 1.5B tokens.
- Artificial long text data, approximately containing 2B tokens.

#### Training

In the continuing pre-training phase, we used the same method as pre-training, but adjusted and optimized for position encoding.

Since 2023, there have been many extrapolation studies on position coding, and these studies have shown significant differences in performance in application scenarios where direct extrapolation without training is possible. However, after sufficient pre-training, the performance differences between these methods become relatively small.

Based on this observation, we chose a more concise approach, which is to extend the Rope position encoding Base by adjusting it to improve the position encoding resolution when processing long text.

![alt text](/0_xFOZFIiPrC0HI7qH.webp)

In addition, similar to the pre-training stage of Llama 3, we implemented a packing training strategy with Attention separation. Taking the first stage 128K context window as an example, each training sample may consist of multiple independent texts of different lengths, which do not interfere with each other during the Attention calculation process (implemented using Flash Attention-based Varlen). This Attention separation is crucial for activating the model's ability to process long texts. Compared with directly applying 128K full attention, it effectively avoids establishing many invalid long-distance dependencies.

### 2.2 SFT

During the SFT stage, we also collected corresponding SFT data specifically for long texts, and appropriately mixed these data with general SFT data to achieve training, ensuring that the model maintains its universality while improving its processing ability for long texts.

![alt text](/0_JLSH2BmHz-Z-9fmE.png)

#### Data

The quality of long text SFT data is crucial for improving the model's ability to handle long text.

Compared to simple question-and-answer questions that only require extracting fragments of information from the document to answer, complex questions that require multiple facts from the document to answer can more effectively stimulate the model's long-text reasoning ability.

In response to this, we screened and distinguished the corresponding data sources and task types based on the actual application scenarios of long texts, and guided annotators to annotate as complex questions as possible and their corresponding answers, thus constructing our long text SFT dataset. Given the difficulty and cost of annotation, we only annotate data within 128K length.

For the GLM4-9B-Chat model, we can directly use these labeled long text SFT data within 128K. However, for the GLM4-9B-Chat-1M model, these data are obviously insufficient in length. Therefore, we propose three methods to construct longer SFT data using short window language models (SCM).

- **SCI ( Single chunk self-instruct, SCI)**: Randomly select a fragment from the given text, whose length matches the context window of the SCM model. Choose one from multiple long text task templates and let the SCM model generate a challenging question and its answer. The final SFT data is concatenated from the original text, question, and answer.

- **Multi- chunk self-instruct ( MCI)**: Randomly select multiple fragments from the given text. The total length of these fragments is equivalent to the context window of the SCM model, and if the text consists of multiple documents, the fragments should be evenly distributed among each document. Then, select one from the long text task template and let the SCM model generate a question and its answer that requires synthesizing information from multiple fragments. The final SFT data is also composed of the original text, question, and answer.

- **Multi-level summary**: In the given text, select one based on the summary task template, divide the text into multiple segments shorter than the SCM context window, and require the SCM model to generate summaries for each segment. Finally, summarize the summaries and generate answers based on the prompts in the task template. The final SFT data is composed of the original text, questions, and answers concatenated.

In order to verify the effectiveness of these methods, we trained a model with a 16K context window using labeled SFT data within 128K length as an SCM model, and automatically generated new 128K length SFT data. Experimental results on downstream evaluation sets such as LongBench-Chat show that the model trained with the new 128K length SFT data has similar performance to the model trained with labeled 128K length SFT data, proving that our method based on SCM model can almost losslessly construct longer SFT data.

In the final data construction process, we used GLM4-128K as the SCM model and generated SFT data with a maximum length of 1M and a minimum length of 128K. These data, together with the labeled SFT data within 128K length, constitute the long text SFT dataset of the GLM4-9B-Chat-1M model.

#### Training

During the training process, we mixed general SFT data with long text SFT data and adopted an innovative training method - Sorted Packing. As described in the LongAlign paper, there are mainly two efficient training methods for long and short text mixed SFT: Packing and Sorted Batching. Sorted Batching may introduce some prior knowledge, that is, the length of data in the same batch tends to be consistent, which may lead to poor training results. In contrast, the Packing strategy can be directly implemented using Flash Attention's Varlen, which is more efficient and has been adopted by mainstream training frameworks such as Megatron-LM.

![alt text](/0_1Qz1nJjYBVdRuN_T.webp)

However, in the long text SFT stage, the uneven complexity inside the Packing may lead to a large amount of GPU idle time, also known as "bubble time". Taking the following figure as an example, there are two 128K Pack sequences, one consisting of four 32K sequences, and the other consisting of one 110K and one 18K sequence, with significant differences in their computational complexity (the dark part in the figure). This leads to a significant difference in the computational time between the two Pack sequences, where the faster copy needs to wait for the slower copy to complete layer synchronization, resulting in significant GPU bubble time.


![alt text](/0_JIxBSuVmqExBrK4E.webp)

As can be seen from the figure below, if we directly use packing training, the training time will fluctuate greatly.

![alt text](/0_f2Mv_-YK9XywMGvj.webp)

This phenomenon is not obvious in the Packing SFT training of short texts, because the difference in computational complexity is small, and the proportion of Attention calculation in the total computational complexity is not high. However, this phenomenon is more obvious in the training of ultra-long texts. To solve this problem, we combine the advantages of Packing and Sorted Batching and propose the Sorted Packing training method. We construct packs within the same batch based on the computational complexity to ensure that the computational complexity of each pack in the same batch is similar, thereby reducing bubble time. In addition, we introduce layer accumulation technology to avoid bias caused by sorting.

Finally, as pointed out in the LongAlign paper, different packs contain different amounts of data, which may lead to uneven loss calculation. Therefore, we also adopt the Loss Reweighting strategy to rebalance the loss.


### 2.3 RLHF（DPO）

In the RLHF stage, we use DPO to minimize the challenges faced by training Infra. At this stage, the main problem we need to solve is how to construct long text DPO data.

We use the same Reward model as short texts to score answers to long texts. To prevent exceeding the context limitations of the Reward model, we only keep the questions and answers, and discard equally long inputs. We also tried using language models such as GLM4-128K as the Reward model for long texts to automatically generate the final answer ranking based on all inputs.

However, the experimental results show that when the long text language model is directly used as a Reward model, the results fluctuate greatly and do not produce the expected effect, so it is better to directly use the Reward model of short text.

We believe that training long text Reward models is crucial for long text RLHF. However, currently, the Data Annotation of long text Reward models is extremely challenging, and we still need to continue exploring to find a reasonable way to train long text Reward models.


## 3. Training of the Infra

In long text training of large models, the main challenge faced by Infra is the significant increase in memory usage of intermediate variable Activation. However, after analyzing the existing mainstream 3D parallel strategies, we found that they all have certain shortcomings in solving this problem.

- **Tensor Parallel TP**: It can reduce the memory usage of Activation, but due to the large communication volume, it is usually not suitable for cross-machine use, and the parallelism is generally not more than 8.

- **Parallel PP on the pipeline**: In order to maintain efficiency, it is usually necessary to increase the micro batch, but while ensuring efficiency, it has no significant effect on reducing the memory usage of Activation.

- **Data Parallelism DP**: cannot reduce the memory usage of a single data sample.
In addition, there are some methods that can directly reduce the memory usage of Activation, such as Activation Checkpointing. However, the help of this method is limited. To ensure efficiency, usually all activations are not released and recalculated. In addition, when the input sequence is too long, there may be insufficient memory in the Forward stage, which cannot be solved by Activation Checkpointing technology.

### 3.1 Sequence parallelism

In order to solve this problem, a new parallel method called Sequence Parallelism has been proposed. Its core motivation is that in the Transformer architecture, tokens only need to interact with each other when performing Attention calculation, while in other parts, tokens are independent of each other. Based on this observation, Sequence Parallelism only performs parallel processing in the Attention part, while in other modules, a long sequence is treated as multiple pieces of data, similar to Data Parallelism DP for processing.

Currently, there are two mainstream implementations of sequence parallelism, namely Ring Attention and DeepSpeed Ulysses. Among them, Ring Attention has been optimized in Megatron-LM and named Context Parallel. These two mainstream sequence parallelism methods each have their own advantages and disadvantages.

#### Ring Attention：

- **Advantages**: Good scalability of parallelism, no obvious limitations; when using GQA, communication is limited to kv within the Group, and the communication volume is relatively small.

- **Cons**: Requires good masking between computation and communication to achieve high efficiency; not friendly enough to changes to Sparse Attention and other Attention variants, it invasively modifies the implementation of Attention.

#### DeepSpeed Ulysses：

- **Advantages**: No invasive modification to the implementation of Attention, relatively friendly to various Sparse Attention and related changes; less communication frequency.

- **Cons**: All parallel replicas require complete model parameters, which is not very friendly to parallel slicing strategies outside of ZeRO; parallelism is relatively limited, generally not exceeding the number of GQA groups, otherwise additional communication is required.


### 3.2 Parallelism of variable-length sequences

In order to be compatible with the mainstream Megatron-LM training framework, GLM4-9B-Chat-1M uses Context Parallel, which is the sequence parallelism of Ring Attention, in training. However, as mentioned earlier, the model needs to use Packing's variable-length training in most stages, which is not natively supported in Attention Ring. To achieve variable-length training, we use the following three solution strategies .

#### Cycle variable length

For Pack data containing multiple sequences, we can split each sequence separately and cyclically apply Ring Attention to calculate the Attention result of each sequence. Finally, the output results can be concatenated in order. When the number of sequences in the Pack is small (for example, a 128K Pack contains four 32K sequences), this method is more efficient because there is no need to introduce additional calculations, the time cost of the loop is relatively small, and each sub-sequence can fully utilize the parallel computing capabilities of the GPU. However, when the number of sequences in the Pack is large (for example, a 128K Pack contains 128 1K sequences), the efficiency of this method will significantly decrease. Due to the short length of each sub-sequence, the parallel computing capability of the GPU cannot be fully utilized, and the overhead introduced by the loop cannot be ignored.

#### Native variable length

Native variable-length sequence parallelism refers to modifying Ring Attention to natively support Ring Attention computation of variable-length sequences. Compared with loop variable-length, this method has significantly improved efficiency, maintaining high computational efficiency regardless of whether there are many or few sub-sequences in the pack. However, when the context length is extremely long (such as 1M) and the length difference of sub-sequences within the pack is large, the memory usage of this method will increase significantly, significantly higher than the implementation of loop variable-length. Therefore, we cannot directly use native variable-length Ring Attention to train large language models with ultra-long contexts (such as GLM4-9B-Chat-1M).

#### Divide and conquer variable length

In order to overcome the challenge of ultra-long context training, we combine the above two variable-length strategies and propose a new divide-and-conquer variable-length sequence parallelism method. For a pack containing 128 subsequences, we divide it into several sub-packs, such as four sub-packs, each containing several sub-sequences. Here, we will try to evenly distribute the sub-sequences to each sub-pack. Then, we use native variable-length loops to calculate the output for each sub-pack, and finally concatenate the output of attention together. Divide-and-conquer variable-length sequence parallelism not only fully utilizes the parallel computing ability of GPU, but also reduces the peak usage of video memory, making variable-length training applicable to ultra-long context training.
