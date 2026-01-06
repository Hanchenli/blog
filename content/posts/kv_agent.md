---
title: "Why Agents are Efficiency Nightmares and How to Fix them?"
date: 2025-01-07
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

Prerequisite: LLM inference basics. 
Suggested reading:
[LLM Inference](https://arpitbhayani.me/blogs/how-llm-inference-works/);   [KV Cache Offloading](https://blog.lmcache.ai/en/2024/09/17/lmcache-turboboosting-vllm-with-7x-faster-access-to-100x-more-kv-caches/)

## Wonderful Agents; But Draining Bank Account.
Everyone said 2025 was the year of agents. With continuous improment in base model and emerging techniques like reinforcement learning, coding agents including [Cursor](https://www.cursor.com/), [Claude Code](https://www.claude.com/product/claude-code) have become much more powerful and automated compared with the start of the year. Quantitatively, the score on [SWE-Bench](https://www.swebench.com/) has risen from 20% as in Aug 2024 to over 70% for frontier models with a simple agent scaffold. 

However, the cost of running these agents is still prohibitively high. 
We conducted a simple benchmark experiment running Claude Code with APIs to get a quantitative measure of the actual token cost.
<!-- jingzhuo can you add some things here -->

We found that handling a single issue in a mid-sized public repository costs around 1 USD when using Claude Code via API.
This is in stark contrast to the subscription license cost of 20 USD per month.
There could be various reasons behind this pricing discrepancy, such as market dynamics, provider strategies, or the implicit value of the labeling work being done by the users.
However, despite the high costs, the capabilities of these agents make them indispensable for many users.
As proved by the usage-based pricing model adopted by [Manus](https://manus.im/), many users are willing to pay for the advanced functionalities provided by these agents, even at a premium.

However, as the adoption of these agents grows, the efficiency of their operation becomes increasingly important.
This directly improves the Return-On-Investment (ROI) for users and makes these tools more accessible to a broader audience.

## The Fundamental Inefficiency of Agents
In order to understand why agents are so expensive to run, we need to look at how they operate fundamentally.
We analyze trace from Claude Code and mini-swe-agent to give a brief overview.
We used each agent to solve a coding task from [SWE-bench_Verified](https://www.swebench.com/) dataset and recorded the message history sent.

### Overview Statistics
We report some high-level statistics from the traces below: 

| Method | Average Total Tokens | Average Cost ($) | Average Calls to LLM | Prefix Cache Hit Rate | 
| :--- | :--- | :--- | :--- | :--- |
| Claude Code | 15,000 | 1.0 | 45 | 92% |
| Mini-SWE-Agent | - | - | 30 | 88% |


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

## Why is Current KV Cache Offloading Not Enough?
From the table above, we can see that both agents have a high prefix cache hit rate (around 90%).
Readers with background may wonder: **90+%** Prefix Cache hit rate? Why not just run KV Cache Offloading and save 10x?

<!-- talk about how KV cache sharing can make it better -->

<embed src="/images/agent_kv/kv.pdf" width="100%" height="600" type="application/pdf">

However, there are three further considerations here: 
1. **Contention for Space**: Due to the appending context behavior, the KV cache space is highly contended among different agent programs due to long context.
2. **Queueling Delay in Scheduling**:  While
the current agent program is waiting on the tool, if the sched-
uler allocates the GPU memory to other requests to maximize
throughput, the KV cache for the current program will be
removed from GPU memory. This incurs a queueing delay for each request when the program resumes after tool execution.
3. **Remaining Compute**: As mentioned above, each call still requires prefill compute even with KV cache hit. With a high number of calls, the remaining compute becomes non-negligible.

Some research works have demonstrated by mitigating the first two issues, KV cache offloading can bring significant speedup and cost reduction for serving LLMs.
For example, [Continuum](https://arxiv.org/abs/2511.02230) proposes to reuse the Time-to-live (TTL) concept to preserve the KV cache for programs that are likely to be resumed soon to mitigate the first two problems.

However, for agents, the remaining compute issue becomes more prominent due to the high number of calls made to the LLM.

## Looking Forward: Beyond KV Cache
<!-- We will talk more about improvement beyond KV Cache Offloading. -->
As we have discussed, while KV cache offloading can help alleviate some of the inefficiencies in agent operation, it is not a complete solution.
To further improve the efficiency of agents, we need to look beyond just KV cache management.
Some potential directions include:
1. **Model Optimization**: Developing more efficient models with smaller footprints during long context.
2. **Adaptive Context Management**: do not append unncessary context to the main loop

Fundamentally memory bound.
We need better algorithm and system co-design to make agents more efficient and accessible.

## Conclusion
We have discussed the inefficiencies of current agent systems and how KV cache sharing can help alleviate some of these issues.
We explained

Disclaimer: This blog post was created with help from Gemini, VSCode copilot. Views are solely those of the authors and do not reflect employer values.