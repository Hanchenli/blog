---
title: "ICLR 2026 Reflections on Self-Improving AI: Reclassifying Continual Learning and Fixing Agent Skills"
date: 2025-04-30
description: ""
author: "Hanchen Li"
tags: ["Continual Learning", "Context Engineering", "Reflections", "Self-Improving", "Agents"]
categories: ["general"]
cover:
    image: images/continual_learning/brazil.png
    alt: ""
    relative: false
ShowToc: true
TocOpen: false

---

I had the priviledge to attend ICLR 2026 this year due to the generous support from UCB Sky Lab. I helped to present Agentic Context Engineering at the main conference plus two oral presentations Lifelong Agent and MemAgent workshops. So my experience mainly centered around agent memory and continual learning. I am writing this reflection mainly to summarize my takeways from conversations with other researchers. Hope this blog strikes a string with fellow readers and does not strike strings with companies' legal teams for leaked information. 

The blog will be consisted of three parts. 1. Continual Learning. 2. The triangle of Model, Harness, and environments. 3. Why Skills are potentially bad format, and how we could fix it.

## Reclassifying Evolving Agents or Continual Learning
<!-- Add graph about three examples -->
After the terms become popular, the classification becomes increasingly vague due to the mentioning of such words in numerous papers. I believe that many works are essentially targeting different use scenarios, though they consider themselves continual learning in the title. 

For example, let's consider two scenarios. The first case would be an agent trying to solve a particular task in hard benchmarks like Frontier CS [repo](https://github.com/FrontierCS/Frontier-CS). The agent can iterate on its solution, and the final goal is to achieve a better score on that specific problem. The second one would just be people are using ChatGPT, and we are trying to improve the model to get better user retention based on all the traces. By intuition, these two scenarios are very different (in resource constraints, methodology). However, both could potentially be called "evolving intelligence". How should we separate just by looking at the problems themselves?

The most satisfying classification criteria that I found: how much the agent assumes about future tasks that it will improve for the future while executing for the current one.

In an OpenEvolve solving Frontier CS question setup, the agent is fully aware of the future problem that it tries to solve since it is the same one, which makes repetitively experimenting with solutions rather enticing. All the takeways that are accumulated in the current run will be directly applicable in the future.

In the evaluation setup as such as the one used in [ACE](https://arxiv.org/abs/2510.04618), [Combee](https://arxiv.org/abs/2604.04247), or [GEPA](https://arxiv.org/abs/2507.19457), the future knowledge is only about the problem type, but the full description would not be provided during the update process. For example, we would be running the training on a speficic agent use case such as a finance domain task to generate a system prompt. This makes overfitting a problem (note the comparison with the previous case), but the agent would not need to update its ability in general problem solving. It only needs to adapt to the specific format of the problems.

For the more general case of Cursor or ChatGPT updating its model checkpoint for all users, the scenario is different again. Probably the only assumption we can make about future tasks is that users will be proposing coding/personal assistant style questions. This means that task specific details generated in each scenarion should probably  be omitted, since the same task would most likely not be repeated. In this case, weight update is probably the most effective way.

These three examples demonstrate that by making clear of the assumption about future task knoweledge, the problem of continual learning or self-improving agents could be cleanly separated into categories. This potentially reduces confusion in disucssions or peer reviews.


## The Three Pillars of Evolving Agents: Models, Harness, and Environments
![Pillars](../../images/agent_kv/pillars.png)
In one of my conversations after MemAgent workshop (with Benjamin), another researher proposed a view of seeing different directions for improving agents: model checkpoint, harness engineering, and feedback from environment. It was a short dicussion but I find the three pillars highly conclusive. Thus I am reusing the abstraction but filling in my own understanding here. 

Model checkpoints are the basis for agents. It is simply the base LLMs that run on inference engines. 

Harness and environment are usually discussed interleavingly. But improvement in retrieval models could hardly be classified as harness engineering. Thus, we could seperate them into environment. The distinction would be harness determines the format of inputs to the model checkpoints, while feedback from environment determines the content of the inputs. For example, adding a step that incorporates skills into agent would be harness engineering, but determining how to best retrieve the relative skill (ex. through RAG) would be environment. 

In many scenarios, harness engineering methods are dirty fixes for model checkpoints. For example, the only reason to spin up sequential subagent is probably because the model becomes forgettable and inefficient with ever-increasing context length. In an ideal world where RNN works perfectly, maybe the agent could simply keep feeding the text to the model. We would not need to do anything like ACE if the models would internalize all its past experiences. The very fact that these methods exist hints us that our model checkpoints truly lack the capabilities to do them on their own. 

Unfortunately model checkpoints updates much slower than the harness. How much would the next generation of models fix these capabilty issues? We would have to wait for the frontier labs to figure them out. But the final landscape may be something similar to a meta-harness that auto-adapts to new models rapidly before the models update their weights again.

On the other hand, feedback from environment remains irreplaceble. If a model has never had access to the news, it simply would not be able to react towards it.

## How Do We Fix Agent Skills?
<!-- Skill -> updated skill -->
It would make little sense to discuss checkpoint in academia.
Let us discuss the biggest thing in harness/environment at the moment, agent skills.

Here is why I think the current format of skill could be improved:
1. Skills are not modification-friendly. They are markdown files that each section is self-contained. Ideally, for an agent flow, you would want to update skills from time to time. But currently the only way out is by rewrite sections.
2. Skills may not be RL-friendly. At this point we should be fairly confident that companies like Anthropic would post-train their model to use the Skills format better. However, at inference time skills are usually composite. It may cause 

Looking forward, we could probably re-design agent skills to fix the above problems. Some initial ideas are: 
1. We can make certain sections of Skills appendable. For example, in some sections
2. Good work

Someone from industry at ICLR were seeking collarboration from researchers on how to improve agent skills. The conversation inspired this section.  I argue that as researchers, we should instead explore a better format for context plugins before labs dumb billions of dollar in RL for skills.



### Acknowledgement
This blog was mainly written on Hanchen's flight back from Panama City to SFO. This time claude code is locked up in the Cyber The ideas are from discussions with people who I met in the conferences, including but not limited to Idan, Benjamin, Yihao, Qizheng.
The definition of "evolve" was discussed by previous blogs by Joseph Gonzalez [here](). Here we are giving a more clear metric for separating these scenarios. 


