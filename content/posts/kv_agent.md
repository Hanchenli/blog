---
title: "Why Agents are Efficiency Nightmares and How to Fix them?"
date: 2025-01-07
description: "Demonstrating the Benefits of KV Cache Sharing in Agent Serving and Future Directions"
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

Prerequisite: LLM inference basics. 
Suggested reading:
[LLM Inference](https://arpitbhayani.me/blogs/how-llm-inference-works/);   [KV Cache Offloading](https://blog.lmcache.ai/en/2024/09/17/lmcache-turboboosting-vllm-with-7x-faster-access-to-100x-more-kv-caches/)

## Powerful Agents; But Draining Bank Account.
Everyone said 2025 was the year of agents. With continuous improment in base model and emerging techniques like reinforcement learning, coding agents including [Cursor](https://www.cursor.com/), [Claude Code](https://www.claude.com/product/claude-code) have become much more powerful and automated compared with the start of the year. Quantitatively, the score on [SWE-Bench](https://www.swebench.com/) has risen from 20% as in Aug 2024 to over 70% for frontier models with a simple agent scaffold. 

However, the cost of running these agents is still prohibitively high. 
We conducted a simple benchmark experiment running Claude Code with APIs to get a quantitative measure of the actual token cost. Our experiment with 30 random verified tasks from SWE-bench revealed some eye-opening numbers. The agent achieved a 63.3% resolution rate, successfully fixing 19 out of 30 complex repository issues autonomously. However, this performance comes with a cost: while the average expense was $0.74 per issue, a single complex Django bug could spike the cost to over $3.20. Most interestingly, over 90% of the token volume was spent on 'cache reads,' indicating that the agent spent the vast majority of its budget repeatedly re-scanning the codebase to verify its steps.

We found that handling a single issue in a mid-sized public repository costs around 1 USD when using Claude Code via API.
This is in stark contrast to the subscription license cost of 20 USD per month.
There could be various reasons behind this pricing discrepancy, such as market dynamics, provider strategies, or the implicit value of the labeling work being done by the users.
However, despite the high costs, the capabilities of these agents make them indispensable for many users.
As proved by the usage-based pricing model adopted by [Manus](https://manus.im/), many users are willing to pay for the advanced functionalities provided by these agents, even at a premium.

However, as the adoption of these agents grows, the efficiency of their operation becomes increasingly important.
This directly improves the Return-On-Investment (ROI) for users and makes these tools more accessible to a broader audience.

## The Fundamental Inefficiency of Agents
In order to understand why agents are so expensive to run, we need to look at how they operate fundamentally.
We analyze trace from Claude Code using Claude-4.5-sonnet and mini-swe-agent using GPT-5 to give a brief overview.
We used each agent to solve a coding task from [SWE-bench_Verified](https://www.swebench.com/) dataset and recorded the message history sent.
The cost report is based on token usage from the API responses and actual bills.
### Overview Statistics
We report some high-level statistics from the traces below: 

| Method | Average Total Tokens | Average Cost ($) | Average Calls to LLM | Prefix Cache Hit Rate | 
| :--- | :--- | :--- | :--- | :--- |
| Claude Code | 1,765,565.63 | 0.74 | 74.17 | 96.3% |
| Mini-SWE-Agent | 230,709.64 | 0.192 | 17.916  | 94.559% |


### Microscopic View
We randomly select one [task (#80)](https://huggingface.co/datasets/princeton-nlp/SWE-bench_Verified/viewer/default/test?views%5B%5D=test&row=80) from the [SWE-bench_Verified](https://huggingface.co/datasets/princeton-nlp/SWE-bench_Verified) dataset. The problem setup is to fix an issue in the `django/django` repo from commit `2e0f04507b17362239ba49830d26fec504d46978`.

**Problem statement:**

> *"JSONField are not properly displayed in admin when they are readonly.*
>
> *Description: JSONField values are displayed as dict when readonly in the admin. For example, `{"foo": "bar"}` would be displayed as `{'foo': 'bar'}`, which is not valid JSON.*
>
> *I believe the fix would be to add a special case in `django.contrib.admin.utils.display_for_field` to call the `prepare_value` of the JSONField (not calling `json.dumps` directly to take care of the `InvalidJSONInput` case)."*

And this is exactly the prompt that Claude Code received.

![Trace 1-46](https://raw.githubusercontent.com/kobe0938/blog/master/claude-code/assets/trace1-46.png)

Surprisingly, before any fancy reasoning, Claude Code ran a couple of **"warm-up" steps** (trace ID `#2`, `#3`, `#4`) before the actual task. Warm-up steps do nothing but input the prompt for:

- Tool list (`#2`)
- Explore subagent (`#3`)
- Plan subagent (`#4`)

Warm-up steps are used for caching purposes—later when those tools and subagents are called, the cache will be hit, resulting in faster response time. The summarization agent (`#1`) and new topic agent (`#5`) are used for summarizing the context and generating a new title for display—just as the ChatGPT sidebar works.

The main agent (`#6`) comes with a huge system prompt, including git history, status, tool list, etc. The **18 tools** in the tool list not only have the ability to use normal tool calls like `Bash`, `Grep`, `Read`, `WebFetch`, `AskUserQuestion`, etc., but also the ability to invoke and delegate certain tasks to subagents like:

- Explore subagent (`#7`)
- Plan subagent (`#46`)

These subagents will invoke tool calls from their own tool lists.

Immediately after the main agent (`#6`), it invokes the **Explore** (also called file search agent) subagent (`#7`), which will invoke tool calls from its tool list to explore the codebase. It starts with a different system prompt where its main goal is to explore the codebase:

> You are Claude Code, Anthropic's official CLI for Claude. You are a file search specialist for Claude Code, Anthropic's official CLI for Claude. You excel at thoroughly navigating and exploring codebases.

Interestingly, the Explore subagent (`#7`) is not the only subagent that Claude Code can invoke. Instead, it invokes **3 Explore subagents in parallel** to explore the codebase, each with a different goal:

1. **Explore JSONField implementation** (lifespan: `#7-#26`)
2. **Explore admin display_for_field** (lifespan: `#8-#37`)
3. **Explore readonly field rendering** (lifespan: `#9-#45`)

The context of the main agent (`#6`) is **not** carried to the subagents, which is beneficial for the subagents to have a fresh start. Each Explore subagent can invoke **1-3 tools in parallel**, where the tools are from the tool list of the Explore subagent—a subset (**10/18**) of the main agent's tool list.

The [ReAct](https://arxiv.org/pdf/2210.03629) mechanism is used here: the Explore subagent will invoke a tool call, then based on the tool output, it will observe and invoke another tool call to explore the codebase further until it deems it has explored enough.

Finally, after the slowest Explore subagent finishes its exploration at step `#45`, at step `#46`, the main agent appends the findings (summarizations) from all 3 Explore subagents to the context, and then invokes the Plan subagent (`#47`) to plan the fix.

---

![Trace 47-76](https://raw.githubusercontent.com/kobe0938/blog/master/claude-code/assets/trace47-76.png)

Similar to the Explore Agent, the Plan Agent (`#47`) also has a different system prompt, where its main goal is to plan the fix:

> You are Claude Code, Anthropic's official CLI for Claude. You are a software architect and planning specialist for Claude Code. Your role is to explore the codebase and design implementation plans.

The Plan Agent did not carry all the context from the main agent nor the Explore subagents, which is beneficial for the Plan Agent to have a fresh start. Instead, it only contains the **summarization** of the Explore subagents' findings. The toolbox is a subset (**10/18**) of the main agent's tool list. The goal for the Plan Agent is to design an implementation plan that:

> Please design an implementation plan that:
>
> 1. Identifies the exact changes needed to display_for_field
> 2. Considers whether we need to instantiate a form field from the model field or if there's a better approach
> 3. Identifies any edge cases or potential issues
> 4. Recommends the best approach given Django's architecture

---

![Trace 77-92](https://raw.githubusercontent.com/kobe0938/blog/master/claude-code/assets/trace77-92.png)

Similarly, the Plan Agent also follows the ReAct pattern and loops through tool calling from `#47` to `#72`, where the context accumulates from **11,552 tokens** to **38,819 tokens**. After having a good plan (see details in `#72`), the Plan Agent will return to the main agent (`#73`) with the plan.

The main agent will then invoke a series of tool calls to:

- Review the plan (`#73`)
- Ask user for clarification (`#74`)
- Write the plan into a markdown file (`#75`)

Finally, the main agent will exit the plan mode (`#76`) and enter the execute mode (`#77`) to execute the plan after interactively asking the user for plan approval (`#76-#77`).

The **execution phase** (`#77-#91`) still follows the ReAct pattern. The main agent will use the plan markdown file as a todo list:

> 1. Add json import to `utils.py`
> 2. Add JSONField handling to `display_for_field()`
> 3. Add tests to `test_admin_utils.py`
> 4. Run the tests to verify

After executing some tool calls to read or edit files, it will cross out the todo items in the plan markdown file. Once all the todo items are crossed out, the main agent will end with a conclusion message (`#92`).

During this phase, there are some other subagents being invoked—e.g., the **Extract Bash Command** subagent (`#93`), where there's only a one-shot prompt template for the subagent to extract the bash command in order to not run dangerous commands like `rm` without user confirmation by accident.

And this is the whole diagram of the claude code trace:

![Claude Code Trace Diagram](https://raw.githubusercontent.com/kobe0938/blog/master/claude-code/assets/claude_code_diagram.png)

---
### Key Observations
We highlight three key observations from the trace analysis above that contribute to the inefficiency of agents:
1. **High number of calls**: Agents make a large number of calls to the LLM server due to the need to repetitive get actions and report status. Each call incurs incremental prefill and may cause KV cache miss that requires recomputation.
2. **Appending Context**: Agents often append new context to the existing conversation history, leading to longer input sequences and increased computational load for each call.

Fundamentally, due to the need to fully incorporate the context of the task into the LLM inputs and the heavy intermediate states (KV cache) related to these contexts, agents are highly memory-bound workloads. Sadly, memory bandwidth and capacity have not improved as fast as compute (FLOPS) in recent years.

## Why is Current KV Cache Offloading Not Enough?
From the table above, we can see that both agents have a high prefix cache hit rate (around 90%).
Readers with background may wonder: **90+%** Prefix Cache hit rate? Why not just run KV Cache Offloading and save 10x?

<!-- talk about how KV cache sharing can make it better -->

![Two Problems that Continuum tries to prevent](../../images/agent_kv/problem.png)

However, there are three further considerations here: 
1. **Contention for Space**: Due to the appending context behavior, the KV cache space is highly contended among different agent programs due to long context.
2. **Queueling Delay in Scheduling**:  While
the current agent program is waiting on the tool, if the sched
uler allocates the GPU memory to other requests to maximize
throughput, the KV cache for the current program will be
removed from GPU memory. This incurs a queueing delay for each request when the program resumes after tool execution.
3. **Remaining Compute**: As mentioned above, each call still requires prefill compute even with KV cache hit. With a high number of calls, the remaining compute becomes non-negligible.

![Total Waiting Time](../../images/agent_kv/delay.png)

These issue limits agents from fully benefitting from KV cache offloading. For example, in the graph above, we enabled [vLLM](https://github.com/vllm-project/vllm) with [LMCache](https://github.com/LMCache/LMCache) to serve agent traces under a certain jps. However, due to the high contention and queueling delay, each agent still incurs significant waiting time even with KV cache offloading enabled.

![TTL-based Eviction Policy](../../images/agent_kv/ttl.png)


Some research works have demonstrated by mitigating the first two issues, KV cache offloading can bring significant speedup and cost reduction for serving LLMs.

For example, [Continuum](https://arxiv.org/abs/2511.02230) proposes to reuse the Time-to-live (TTL) concept to preserve the KV cache for programs that are likely to be resumed soon to mitigate the first two problems. This changes the eviction policy from LRU to TTL-based, which is more suitable for agent workloads.

![Evaluation Results of Continuum](../../images/agent_kv/eval.png)

As demonstrated by the experiments, Continuum can bring up to 3.66x improvement for serving LLM agent traces with long context on SWE-Bench and Berkeley Function Calling Leaderboard workloads. The above graph shows the evaluations results. Each method is paired with LMCache as the base system. For A100 GPUs, we pair 100GB DRAM per GPU and for B200 GPUs, we pair 200GB DRAM per GPU. We set TP=4 for 70B model. The requests arrive according to a Poisson process with a certain JPS. The graphs show the average latency per request under different JPS.

The paper arxiv links is here: [Continuum Paper](https://arxiv.org/abs/2511.02230), and a preview version of code is available at: [Continuum Code](https://github.com/Hanchenli/vllm-continuum).


## Looking Forward: Beyond KV Cache Management
However, as shown in the graphs, each agent execution still takes significant time even with KV cache offloading.
Even without any contention, each call still requires multiple long prefill and decode operations throughput the trace. These computations add up in the end.

To further improve the efficiency of agents, we need to look beyond just KV cache management.
Some potential directions include:
1. **Model Optimization**: Researchers have been proposing methods that reduces memory footprint for intermediate states recorded in the agent trace. Examples include [KV cache compressions](https://github.com/NVIDIA/kvpress), and more efficient attention mechanisms like [linear attention](https://arxiv.org/abs/2006.16236), [Mamba](https://arxiv.org/pdf/2312.00752) mechanisms.
2. **Adaptive Context Management**: As a recent trend, agentic context engineering ([ACE](https://arxiv.org/abs/2510.04618)) has been growing popularity. Many work proposes to filter the input text before feeding them into the LLM. Example works include [ReSum](https://arxiv.org/pdf/2509.13313), [AgentFold](https://arxiv.org/pdf/2510.24699), and Manus's [Context Engineering Report](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)...  These methods prevents unncessary context from flooding the LLM context window, reducing computation from the source.

Going forward, agents are and will probably remain memory-bound for the foreseeable future. Addressing this fundamental bottleneck requires a careful co-design of algorithms and systems. This enables not only powerful but efficient future agents. After all, ROI is the key metric for GenAI to succeed in the real industry in the long run.

## Conclusion
We have discussed the inefficiencies of current agent systems and how KV cache sharing can help alleviate some of these issues.
We explained

Disclaimer: This blog post was created with help from Gemini, VSCode copilot. Views are solely from the authors and do not reflect employer values.
