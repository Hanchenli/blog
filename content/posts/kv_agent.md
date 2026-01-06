---
title: "Why Agents are Efficiency Nightmares and How to Fix them?"
date: 2025-11-11
description: "Blog Post Demonstrating the Benefits of KV Cache Sharing in Agent Serving"
author: "Hanchen Li, Xiaokun Chen, Jingzhuo Hu, and Collaborators"
tags: ["LLM", "Inference", "Agent", "KV Cache","LMCache"]
categories: ["general"]
ShowToc: true
TocOpen: false
---

Agents are EXPENSIVE. Claude Code takes 1 USD to handle a single issue in a mid-sized public repository when using API. 
In the same time, it charges 20 USD per month for a subscription license. 
Why do we still get to use them? 
Maybe the price war, maybe the crazy debts some companies upstream are carrying, maybe the implicit labeling you are carrying out for the providers.

But regardless of the reason behind the pricing created by the giants, we want to use agents so we can sit back and enjoy a sip of the coffee.



## Wonderful Agents Marching Towards Automation. 
[Claude Code](https://www.claude.com/product/claude-code) has quietly become one of the most interesting & widely-adopted real-world agentic systems available to normal developers.



## Why the LLM inference were not designed for Agents?
<!-- talk about kv cache sharing problem -->

## How to make it better?
<!-- talk about how KV cache sharing can make it better -->

## The Inefficiency dilemma of Agents
<!-- We will talk more about improvement beyond KV Cache Offloading. -->