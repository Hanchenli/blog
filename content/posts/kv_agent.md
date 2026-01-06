---
title: "Why Agents are Efficiency Nightmares and How to Fix them?"
date: 2025-1-07
description: "Blog Post Demonstrating the Benefits of KV Cache Sharing in Agent Serving"
author: "Hanchen Li, Xiaokun Chen, Jingzhuo Hu, and Collaborators"
tags: ["LLM", "Inference", "Agent", "KV Cache","LMCache"]
categories: ["general"]
cover:
    image: images/agent_kv/front.png
    alt: "Agent Efficiency"
    relative: false
ShowToc: true
TocOpen: false
---

Agents are EXPENSIVE. Claude Code takes 1 USD to handle a single issue in a mid-sized public repository when using API. 
In the same time, it charges 20 USD per month for a subscription license. 
Why do we still get to use them? 
Maybe the price war, maybe the crazy debts some companies upstream are carrying, maybe the implicit labeling you are carrying out for the providers.

But regardless of the reason behind the pricing, we want to use more these powerful agents.
Current models and applications are already capable of handling many complex tasks with minimal human intervention.
However, the efficiency of these agents has just rised to our attention. 
In this blog post, we will discuss why agents are efficiency nightmares, how we can make them (somewhat) better with KV cache management, 
and where we could be going for these tasks.


## Wonderful Agents; But Draining Bank Account.
Everyone said 2025 was the year of agents. With continuous improment in base model and emerging techniques like reinforcement learning, coding agents including [Cursor](https://www.cursor.com/), [Claude Code](https://www.claude.com/product/claude-code) have become much more powerful and automated compared with the start of the year. Quantitatively, the score on [SWE-Bench](https://www.swebench.com/) has risen from 20% as in Aug 2024 to over 70% for frontier models. 

 quietly become one of the most interesting & widely-adopted real-world agentic systems available to normal developers.
<!-- Talk about agent applicaitons, some benchmarks on the cost -->


## The Fundamental Inefficiency of Agents
<!-- talk about kv cache sharing problem -->

## Why is Current KV Cache Offloading Not Enough?
<!-- talk about how KV cache sharing can make it better -->

## More Challenges, yet Juicier Rewards
<!-- We will talk more about improvement beyond KV Cache Offloading. -->

## Conclusion
We have discussed the inefficiencies of current agent systems and how KV cache sharing can help alleviate some of these issues.
We explained

Disclaimer: This blog post was created with help from Gemini, VSCode copilot. Views are solely those of the authors and do not reflect employer values.