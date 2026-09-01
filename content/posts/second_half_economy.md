---
title: "The Second Half of the Agent Economy: Token Abundance and Stake Scarcity"
date: 2026-09-03
description: ""
author: "Hanchen Li"
tags: ["AI", "Agents", "Economics", "Token Economy", "SaaS", "Attention"]
categories: ["general"]
ShowToc: true
TocOpen: false
---

<!--
## Overall 
1. Token cheaper and cheaper
2. if people release API will be distilled.
What left unimpacted? Stake (resources needed to exchange besides tokens for other scarce resources)

## Token cost prediction
We will prob have 100x cheaper token/intelligence
intelligence /watt  (do some calc)
hardware flops cost (do some calc)
profit margin reduces (gross margin overherd to be 90%, for telecom it is ~60% (get an annual report).)

## On Labs
Research -> High Stake senarios -> example, prediction, bio, security, ....

Neo cloud -> sell for general intelligence, work automation, 
"continual learning" will become something that every neocloud has. which means continual learning startups may become neocloud

## On SaaS
Increasinly using AI. 
Application will  not die. we want symbols. cultural symbols like mickey mouse. 
it requires us to put stakes in there

example Spotify will thrive. Even we all do ai generated music, some music are more equal than the others because our society puts more value on it so that you resonate. How? pay for ads, promotion acitivities.

## On Attention
Humans will have stakes.
engineering is cheap, distribution will not be. 
App devs are essentially vcs betting an app design will be good. 
Input: time prompting, money
Output: MAU, revenue


## Conclusion
The great buildout is still ongoing.
The first part is building the basic infra.
Second part is we will see more AI taking critical stakes.
-->


> **TL;DR**
> 1. Tokens, especially specialized tokens, are becoming increasingly abundant on the market.
> 2. A reasonable scenario is that today's level of intelligence becomes roughly **100× cheaper within 3 years**. 
> 3. When intelligence becomes abundant, differentiation shifts back to the scarce resources required for production: money, attention, or the trust for providing insurances. I call this **stake**.
> 4. Under this framing, labs will move toward high-stakes decisions, while neoclouds will become operating systems for digital labor; Software will retain value through habits, trust, and cultural meaning; and attention will become even more expensive.
> 5. We are on our way to build intelligence infrastructure. Yet we have yet let the intelligence take our stakes. This should be the real driver for mass adoption.

# Abundant Token

Researchers and engineers have built amazing models. 

Running on Gigawatt clusters, these models have turned intelligence into something you order by the million. Generating a decent website for a restaurant is now a line item. The more consequential change is not that the best models got better. It is that good-enough models got common. A capability that only a frontier lab could serve a year ago tends to show up soon after in a smaller open-weight model that a mid-sized company can host on its own hardware.

"We have access to a good model" is a shrinking advantage for business. For most of the current economics driving activities, an Opus 5 level model might already be close to enough. It remains possible that frontier models could create some new applications that requires frontier intelligence. But since Fable 5.1 still reports terminal bench, let's assume no new industry will be created.

So what remains valuable when tokens become abundant? 

Well, there are lots of things, but here is a word that represents the nature of these things: **stake**

By stake, I mean a scarce resource that must be committed in addition to tokens before an agent can create real value. It can be:

- capital that can be invested to purchase GPUs or bio materials;
- proprietary data or privileged access to an environment that may damage current business;
- reputation and trust accumulated over years;
- distribution and the attention of other people;
- the ability to be responsible for the consequences of a decision.

An LLM can produce ten acquisition strategies. It cannot make any of them matter until somebody commits a budget and a brand. It can write a drug discovery plan. It cannot pay the development cost, accept the possibility that the trials may totally fall apart, or accept liability for a medical accident. It can generate a million songs. It cannot form a fan base to monetize from without actual human participation.

Tokens will help to accelerate the production of possibilities. 
But only the stakes select which possibilities become real (or fail).

# A Rough 100× Scenario

"Tokens will get cheaper" is easy to believe. It is more useful to start from a number somebody actually measured.

Almost nobody publishes their serving economics; DeepSeek did. Its [February 2025 production disclosure](https://github.com/deepseek-ai/open-infra-index/blob/main/202502OpenSourceWeek/day_6_one_more_thing_deepseekV3R1_inference_system_overview.md) reports an average of 226.75 nodes of eight H800s, each node sustaining 73.7k input tokens per second during prefill and 14.8k output tokens per second during decode. At $2 per GPU-hour a node costs $16 an hour, so the accelerator cost works out to **$0.30 per million output tokens** and **$0.06 per million input tokens**. A short novel's worth of generated text, 100,000 tokens, cost about three cents of chip time — from a model near the frontier at the time. That is a floor, not a full cost: it excludes training, research compute, idle capacity, and the free consumer app.

Two multipliers take that to 100×. Hardware supplies most of it: [Epoch AI estimates](https://epoch.ai/data-insights/chip-performance-per-dollar) performance per dollar on shipped AI chips improved **49% per year** from 2023 through 2025, and 1.49^10 ≈ **54×** over a decade. The remaining **1.85×** has to come from software and model efficiency — and the same disclosure shows how much room is sitting there. DeepSeek-V3 activates [37B of 671B parameters](https://arxiv.org/abs/2412.19437) per token, so decoding costs roughly 74 GFLOP per token. At 1,850 tokens per second per GPU that is 137 TFLOP/s of useful arithmetic against the H800's 1,979 dense FP8 TFLOP/s: **about 7% utilization**. Decoding reads the entire active parameter set to emit one token, so it is bound by memory bandwidth rather than FLOPs, which is exactly what speculative decoding, larger batches, KV-cache reuse, and higher-bandwidth chips all attack. Needing 1.85× when you start at 7% is a modest ask.

At 100×, $0.30 becomes $0.003 per million output tokens. An agent burning 10 million output tokens a day costs $3.00 in chip time today and three cents in that world.

![Decomposition of a 100x reduction in cost per output token](../../images/second_half_economy/cost_decomposition.png)
<!-- GRAPH: waterfall chart decomposing the 100x. X-axis categories: "Today ($0.30/M output tok)",
 "Hardware price-perf (54x)", "Utilization + software", "Model sparsity/distillation", "2036 (~$0.003/M)".
 Y-axis: cost per million output tokens, log scale from $0.001 to $1. Each bar is a descending step.
 Annotate the hardware bar with "Epoch: 49%/yr" and the utilization bar with "7% MFU today".
 Takeaway: hardware does most of the work; the remaining ~2x is the easy part. -->

Energy behaves differently, and the difference matters. At [1.37 kW all-in](https://inferencex.semianalysis.com/chips/h100) per chip and 1,850 tokens per second, a token costs 0.74 J, or 0.206 kWh per million — about **$0.018** at the US industrial rate of [8.9 ¢/kWh](https://www.eia.gov/electricity/monthly/epm_table_grapher.php?t=table_5_03), only 6% of the accelerator cost. Today the chip is the constraint, not the power running through it. But Epoch also finds that [ML hardware energy efficiency doubles only every three years](https://epoch.ai/publications/trends-in-machine-learning-hardware), giving 2^(10/3) ≈ **10×** per decade against hardware's 54×. The two curves diverge by more than 5×, and the binding constraint migrates from "can we afford the accelerators" to "can we get the megawatts" — which is what the current buildout is really about.

None of this means a specific API price falls smoothly by exactly 100×. Chips arrive in steps, data centers do not follow semiconductor learning curves, and providers may spend efficiency on longer reasoning traces instead of lower prices; a cheaper token times a hundred more tokens per answer is not a cheaper answer. But 100× over a decade is conservative next to the recent record: [Epoch found](https://epoch.ai/data-insights/llm-inference-price-trends) the price of reaching a fixed benchmark score falling between **9× and 900× per year**. Intelligence will not become free, but many capabilities that feel expensive today will be priced like bandwidth.

Margins follow the same logic. Software trained investors to expect a near-zero marginal cost per user; tokens carry a physical cost on every request, and competition passes each efficiency gain to customers. At the time of DeepSeek's disclosure, R1 listed at $2.19 per million output tokens against that $0.30 floor — 7.3×, and the source of their theoretical "545% cost profit margin." The endpoint that inherited R1's traffic now lists at [$1.32 peak and $0.66 off-peak](https://api-docs.deepseek.com/quick_start/pricing). The floor moved too, so it is not like-for-like, but the direction is the one competition always pushes.

Mature infrastructure can still be an excellent business without 90% gross margins. Oracle, for example, reported a [63% margin across cloud and license revenue in fiscal 2025](https://www.sec.gov/Archives/edgar/data/1341439/000095017025087926/orcl-20250531.htm). Spotify reported a [34% Premium gross margin in 2025](https://s29.q4cdn.com/175625835/files/doc_financials/2025/ar/Spotify-20-F-Filing.pdf), in large part because valuable content owners must also be paid. The exact long-run margin for model APIs is unknowable, but the direction is clear: as equivalent intelligence becomes substitutable, providers will not be able to price it as magic. They will price it more like infrastructure.

# On Labs

The last item matters more than it first appears. Once a model is exposed through an API, its behavior becomes observable. A user can collect input-output pairs and train a cheaper model for a narrower distribution of tasks. This does **not** mean that a small model can copy an entire frontier model, and providers can contractually prohibit unauthorized competitive distillation. But task-specific compression is not hypothetical: [OpenAI itself offers a distillation workflow](https://openai.com/index/api-model-distillation/) for training smaller, cheaper models from the outputs of more capable ones.

The result is a ratchet. A frontier capability begins as scarce research, becomes a premium API, gets optimized and imitated, and eventually becomes a cheap primitive. The frontier lab can run ahead of this process, but it cannot make yesterday's intelligence scarce again.

If ordinary intelligence becomes cheap, frontier labs have two durable advantages left.

The first is continuing to invent genuinely new capabilities. Research stays valuable because the frontier is always scarce, even if each fixed point behind it is rapidly commoditized.

The second is moving closer to decisions where the model's output controls something scarce. A prediction is cheap; the trading book placed behind it is not. A protein sequence is cheap; the robotic lab time and clinical program are not. A vulnerability report is cheap; privileged access to production systems and authority to patch them are not.

This points toward high-stakes domains:

- **Prediction and capital allocation.** The model is coupled to money and judged by realized returns, not eloquent forecasts.
- **Biology.** The model is coupled to proprietary experimental data, lab capacity, regulatory approval, and patient outcomes.
- **Security.** The model is coupled to protected systems, credentials, response authority, and accountability when something breaks.
- **Industrial operations.** The model is coupled to factories, power grids, supply chains, and the cost of downtime.
- **Government and law.** The model is coupled to institutional authority, due process, and decisions that affect people's rights.

In each case, model quality matters enormously. But the model is only one component of a system whose other inputs cannot be copied from API traces. The moat is the closed loop between prediction, action, feedback, and responsibility.

This also changes the lab's product. Selling an answer is easy to benchmark and easy to substitute. Selling a verified outcome requires integration, monitoring, insurance, escalation paths, and a willingness to be held accountable. The lab moves from "here are some tokens" to "we will operate this function."

# On Neoclouds

Today's neoclouds mostly sell access to scarce accelerators. In the next phase, they will sell access to abundant intelligence.

At first this sounds like a worse business: if tokens get cheaper, why would an intelligence cloud be valuable? Because enterprises do not merely need a model endpoint. They need a managed workforce of agents that can access internal systems, remember how the organization works, improve from feedback, respect permissions, recover from failures, and produce auditable results.

The unit of sale shifts from GPU-hours to completed work.

That requires a new control plane:

- identity and permissions for every agent;
- persistent memory and organizational context;
- evaluation, monitoring, and rollback;
- routing work across frontier and specialized models;
- human escalation when confidence is low;
- learning from accepted, rejected, and corrected work;
- security and compliance across the full trajectory.

In this world, **continual learning becomes a standard cloud feature**. Every serious provider will need a way to turn a customer's interaction history into better future performance. Sometimes that will mean updating weights. Often it will mean updating memory, retrieval indexes, skills, policies, or the agent harness. The implementation matters less than the product promise: the service should understand the organization better in month twelve than it did in month one.

This creates an interesting path for continual-learning startups. A company that begins by improving models from production feedback may discover that it needs to host inference, store trajectories, manage permissions, run evaluations, and deploy agents. It gradually becomes a neocloud. Conversely, a neocloud that begins with GPUs may need to build continual learning to avoid becoming a commodity compute reseller.

Their moat is not the token. It is the customer's feedback loop—and the trust required to operate it.

# On Software

There is a popular claim that agents will kill applications. If a model can generate an interface and write arbitrary code, why keep paying for software?

Because an application is not just its code.

A serious application contains years of workflow decisions, customer data, permissions, integrations, compliance work, habits, and shared vocabulary. Salesforce is not merely a set of database screens. Figma is not merely a canvas renderer. An enterprise resource planning system is not valuable because nobody else can generate forms. These products are valuable because organizations have agreed to coordinate through them.

Agents will change the surface of SaaS. Interfaces will become more dynamic. Features that once justified separate products will collapse into prompts. Some lightweight tools whose only moat is implementation will disappear. But the systems that hold state, mediate trust, and define a workflow may become more important, because agents need authoritative places from which to act.

Consumer applications have another defense: **symbols**.

People do not consume only for functional quality. We consume things because other people recognize them. Mickey Mouse is technically reproducible as pixels; Disney's accumulated cultural meaning is not. A luxury bag carries materials and workmanship, but also a public symbol. A game, a social network, and a musician all become more valuable when they are part of a shared conversation.

AI makes production abundant, which can make symbols more—not less—important. When anyone can generate a competent song, image, game, or app, quality is no longer enough to tell us where to look. We rely more heavily on curation, identity, provenance, community, and the knowledge that other people are paying attention to the same thing.

Spotify is a useful example. Generative music could produce an effectively infinite catalog, but listeners still have finite time. Spotify's value is not simply that it stores audio files. It knows listening history, owns a discovery surface, provides a shared object for playlists and fandom, and connects artists to audiences. Its [Discovery Mode](https://artists.spotify.com/en/discovery-mode) makes the stake explicit: artists and labels can prioritize particular songs for algorithmic discovery, and Spotify charges a commission on streams from those contexts. The scarce resource being allocated is not music. It is listener attention.

Even in a world of excellent AI-generated music, some songs will be "more equal than others" because society coordinates around them. Money will be spent on promotion. Artists will appear at events. Fans will build identity around a scene. Friends will send the same track to one another. The song matters because humans put stakes behind it.

# On Attention

Once engineering gets cheap, distribution becomes the bottleneck.

Imagine that one person with agents can build 100 polished applications per year. This sounds like a 100× increase in startup output. From the user's perspective, however, it is also a 100× increase in things asking to be noticed. The number of hours in the day has not changed. The number of home-screen slots has not increased by 100×. A company can generate more ad creative, but every competitor can do the same.

The abundance of software therefore does not eliminate risk. It moves risk downstream.

An app developer starts to look less like a builder and more like a venture capitalist. The developer allocates tokens, prompting time, advertising money, reputation, and months of attention across a portfolio of ideas. Most will fail—not because they were difficult to build, but because nobody cared. A few may return monthly active users, revenue, or influence.

The basic balance sheet becomes:

| Invested stake | Possible return |
| --- | --- |
| Human attention and judgment | User attention |
| Token budget and compute | Revenue |
| Advertising and distribution | Monthly active users |
| Brand and reputation | Trust |
| Proprietary data and access | Better future decisions |

This is why humans remain central even when agents perform most of the engineering. Humans possess and allocate the stakes. We decide which project receives another month, which generated character becomes a mascot, which recommendation gets capital, and which mistake somebody must answer for.

Eventually agents may control some of these resources too. An agent could hold a budget, operate a brand, negotiate with other agents, and accumulate a reputation. But that would not abolish stake. It would mean agents had entered the scarce-resource economy rather than merely producing cheap tokens beside it.

# Conclusion for the Second Half

The great AI buildout is still ongoing.

The first half built the basic infrastructure: chips, data centers, foundation models, inference systems, agent harnesses, customized intelligence. Its central metric was how much intelligence we could produce and how cheaply we could produce it.

The second half will be about attaching that intelligence to scarce things.

Labs will attach models to scientific experiments, security authority, and capital. Neoclouds will attach agents to organizational memory and work. SaaS companies will attach them to trusted systems of record and cultural symbols. Developers will attach them to distribution bets. Eventually, agents themselves may hold budgets, reputations, and legal responsibilities.

The winning companies will not simply have the most tokens. They will control the best feedback loops and earn the right to put real resources behind the tokens they generate.

**Intelligence will become abundant. Commitment will not.**

*Disclaimer: This blog post was created with help from an AI writing assistant. Views are solely from the author and do not reflect employer values. This is not financial advice.*
