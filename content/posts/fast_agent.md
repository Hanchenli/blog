---
title: "CES and Groq Acqui-hire Reflection: NVidia's Potential to Build Real Time Agents"
date: 2025-01-27
description: "A deep dive into NVidia's recent CES announcements, exploring their strategy to enhance LLM inference speed and efficiency through innovative hardware and software solutions."
author: "Hanchen Li, and Collaborators"
tags: ["LLM", "Inference", "Agent", "KV Cache","LMCache", "Memory", "SRAM"]
categories: ["general"]
cover:
    image: images/agent_future/main.png
    alt: "Agent Efficiency"
    relative: false
ShowToc: true
TocOpen: false

---

In this blog post, we will discuss NVidia's recent announcements at CES and their acquisition of Groq, focusing on their strategy to enhance LLM agent inference.
We will explore three main aspects: the importance of KV cache hits, the role of SRAM in improving decoding speed, and a proposed hardware-software architecture that potentially speeds up agent inference from task-based to real-time.

## CES Announcements and Groq Acquisition
In CES 2026, NVIDIA introduced the Rubin architecture. As described by this [announcement](https://nvidianews.nvidia.com/news/rubin-platform-ai-supercomputer) This new GPU architecture is designed to significantly boost the performance of large language model (LLM) inference by 10x. It incorporates NVIDIA Vera CPU, Rubin GPU, NVLink 6 Switch, ConnectX-9 SuperNIC, BlueField-4 DPU and Spectrum-6 Ethernet Switch. It natively supports the Inference Context Memory Storage Platform [ICMS](https://developer.nvidia.com/blog/introducing-nvidia-bluefield-4-powered-inference-context-memory-storage-platform-for-the-next-frontier-of-ai/), which stores KV cache for contexts that allow future reuse.

Moreover, NVIDIA announced the acqui-hire of Groq, a company known for its high-performance AI accelerators. Utilizing SRAM extensively, Groq's LPU architecture is highly optimized for low-latency inference. making it a perfect fit for real-time applications. This acquisition is expected to enhance NVIDIA's capabilities in delivering fast and efficient LLM inference.


## Immediate Comments on the News

From the authors' point of view, the new Rubin architecture is basically adding a KV cache layer to the existing GPU platform. The ideas are similar to the previous post about LMCache, but NVidia have also added some additional hardware components such as the DPUs (Data Processing Unit). 

![KV architecture](../../images/agent_future/storage.png)

On the Groq side, this acqui-hire may be a defensive move for NVidia to reduce competitors. But regardless, ultra-low latency inference is crucial for real-time agents. Groq's architecture is well-suited for this purpose and potentially can be integrated into future NVIDIA platforms.

These movements demonstrate NVidia's investments into inference, which it has been talking about since 2025. This aligns with the problem faced by NVidia's biggest end customer: Frontier Labs including OpenAI, Anthropic, ...  One of the main doubts for these frontier labs before they raise more capital is their profitability. While people are racing for the best capability on benchmarks and providing training services like [Tinker](https://thinkingmachines.ai/tinker/), so far inference is the only cash cow for them. Gross margin of Anthropic and OpenAI are rumored to be around 40% according to [The Information](https://www.theinformation.com/articles/anthropic-lowers-profit-margin-projection-revenue-skyrockets), leaving plenty of room for improvement.


## Importance of KV Cache Hits
In our previous blog [post](https://hanchenli.github.io/blog/posts/kv_agent/), we disussed the significance of KV cache hits in enhancing the efficiency of LLM agents. By storing previously computed key-value pairs, agents can avoid redundant computations, leading to faster response times and reduced computational load.

The danger of a cache non-hit for any agent LLM trace is that it has to do the prefill for all the previous tokens again. For example, if an agent misses the previous KV cache for a 100K token context, it has to redo the prefill. On xx GPU with balba, prefill takes xx seconds. Since prefill is not batched, this will also block other tasks on the same GPU from being scheduled. Given that each agent can easily go over 20 turns, a few times of KV cache re-prefill can easily inflate the total latency by multiple times.

However, if we are able to save and reuse the KV cache for long contexts, we can significantly reduce the full prefill time. For each round of the agents, we only need to do incremental prefill for the newly added user and agent messages. This can lead to substantial speedups by reduced computation.

## Why Decoding Speed is the True Killer
What we did not discuss in the previous post is the improvement of decoding in agent inference.

Depending on the task, agents often contain long decoding phases. We give a brief breakdown of the time spent in prefill and decoding for a SWE-agent task assuming this is the only task on the set of GPUs. 

<!-- Put a graph here on decode and prefill -->






## Why NVidia has the Potential to be the First to achieve real-time agents
Let's now assume we are the CTO of NVidia. Given the previous calculations, you will suddenly realize that if we just put the two pieces together: ICMS for KV cache hits and Groq's SRAM-based architecture for decoding speed, we can potentially achieve real-time agents. Here is a rough sketch of the proposed architecture:

![Proposed Architecture](../../images/agent_future/architecture.png)



Inference Context Memory Storage Platform can be used to store KV cache for long contexts to optimize for prefill speed. Groq's SRAM-based architecture can be used to optimize decoding speed. What is the limit given these two building blocks now and what will be the limit in the future?


## Conclusion




