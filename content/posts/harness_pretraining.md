---
title: "The Hidden Links Between Harness and Pretrain, and Why they should be Co-Designed from the Beginning"
date: 2069-02-20
description: "Harness and pretraining are linked in both directions: the harness feeds the data flywheel that pretraining consumes, and pretraining biases dictate what the harness must manually correct. This post argues they should be co-designed, and that human strategies should be distilled into the harness rather than the weights."
author: "Hanchen Li"
tags: ["Agents", "Harness", "Pretraining", "Skills", "Context Engineering", "Self-Improving"]
categories: ["general"]
cover:
    image: images/harness_pretraining/main.png
    alt: "Two-way arrows between Harness and Pretraining"
    relative: false
ShowToc: true
TocOpen: false
---

<!-- GRAPH (cover, main.png): the training pipeline drawn as four boxes in a row —
     "Pretrain → Midtrain → Posttrain → Harness" — with two hidden arcs drawn
     UNDER the pipeline connecting the two ends directly: one arc from Harness
     back to Pretrain labeled "data flywheel (trajectories → next run)", one
     from Pretrain to Harness labeled "reflexes the harness must correct".
     Visual metaphor: the two ends of the pipeline secretly holding hands
     behind it. Clean, poster-style. -->

The way we talk about building agents has a natural order to it: pretrain, midtrain, posttrain — and then, almost as an afterthought, the harness. First you grow the mind, then you hand it tools. In this picture the harness sits at the very end of the pipeline, downstream of everything, scaffolding bolted on after the real work is done.

I think this picture is wrong. The harness and pretraining — the two ends of the pipeline — are secretly holding hands behind it. Going forward, the harness decides what trajectories the data flywheel collects, and therefore what the next pretraining run eats. Going backward, pretraining decides which reflexes the model is born with, and therefore what the harness must spend its life correcting. This post traces both links and argues the two should be co-designed from the beginning.

TL;DR, roughly in order of how new I think each claim is:

- **Long term, the harness probably matters more than RL.** The harness feeds the data flywheel that turns real usage into training signal; RL's job is to get the wheel spinning.
- The flywheel does not fix everything. **Pretraining bakes in biases — like "everything in my context is correct" — that the flywheel inherits rather than breaks.** Breaking them is manual work, done in the harness.
- Pretraining gives models an animal's reflexes, and some reflexes are wrong for the environment the agent actually lives in.
- So **human strategies — forgetting, restarting — should be distilled into the harness, not the weights.** They fight the pretraining prior; the harness is the symbolic layer where they belong.

I use "harness" in the same sense as my [ICLR reflections post](../continual_learning/): the layer that determines the *format* of inputs to the model checkpoint — context layout, tools, subagent structure — as opposed to the environment, which determines the *content*.

## Harness → Pretrain

### The Data Flywheel

The clearest precedent is Tesla FSD. Every car is a data-collection device: when the driver intervenes or disengages, that moment gets flagged, uploaded, labeled, and folded into the next training run. Deployment produces data, data produces a better model, a better model justifies wider deployment. The product *is* the data pipeline.

Coding agents run the same loop. Claude Code and Cursor sessions produce complete trajectories — the context the model saw, the tools it called, the edits it made — plus a natural quality signal: was the edit accepted, did the tests pass, did the session end in a commit or in frustration. Filter for the good trajectories and you have SFT data that is exactly on-distribution for the product. <!-- TODO: cite a public statement from Anthropic/Cursor on training over user trajectories, if one exists -->

![Data flywheel for coding agents](../../images/harness_pretraining/flywheel.png)
<!-- GRAPH (flywheel.png): a circular flywheel diagram with four stages:
     "Deploy harness" → "Users generate trajectories" → "Filter good trajectories
     (accepted edits, passing tests)" → "SFT / mid-train on them" → back to "Deploy".
     Optionally annotate the Tesla FSD analogy on the outside of the circle
     (interventions → labels → retrain → OTA update). Takeaway: the harness is
     the entry point of the loop — it defines what a "trajectory" even is. -->

### The Harness Completes the Flywheel

The underrated part: the flywheel only closes because the harness fixes the data format. The harness decides what a trajectory looks like — how tool calls are serialized, how files appear in context, how turns are structured. Collected data is therefore already in the serving format and can be fed straight back into SFT.

Compare this against RL as a way of improving the same agent:

- **Good:** enormous data volume — every user session is a candidate — and the training is simple and efficient. It is supervised learning on filtered trajectories: no reward model, no rollout infrastructure, no credit-assignment headaches.
- **Bad:** you need users first, and a cold-start model may not be good enough to produce trajectories worth training on.

That cold-start gap is where RL fits: it amplifies the good behaviors already in the model so the harness works well enough for people to adopt it. But once real usage arrives, the flywheel scales with users, while RL scales with rollout compute and reward engineering. **RL gets the flywheel spinning; the harness is the flywheel.**

![RL bootstraps, the flywheel compounds](../../images/harness_pretraining/rl_vs_flywheel.png)
<!-- GRAPH (rl_vs_flywheel.png): conceptual line chart, x-axis = time / deployment
     scale, y-axis = marginal capability gain per unit effort. Two curves:
     "RL" starts high and flattens (limited by rollout compute + reward
     engineering); "Harness data flywheel" starts near zero (cold start, no
     users) and grows past RL as user count grows. Mark the crossover with an
     annotation like "enough users → flywheel takes over". Label it as a
     conceptual sketch, not measured data. -->

### The Harness Lock-In Dilemma

This flywheel has a consequence I have not seen discussed much. **Once you commit to a harness, all your user data is shaped by it — and training on that data locks you in further.** Trajectories encode your specific tool names, context layout, and subagent conventions. SFT on them makes the model better *at that harness*, which makes the harness harder to change, which makes the next batch of data even more harness-specific.

How well this generalizes is genuinely unclear. If a lab trains for two years on trajectories from harness A and then wants to ship harness B, how much capability transfers? Is agentic skill mostly harness-agnostic problem solving, or mostly fluency in one format? Nobody has published a clean answer. <!-- TODO: cite if any harness-transfer study exists -->

Lots of interesting work fits here: measuring transfer across harnesses, re-rendering flywheel trajectories into multiple formats, or designing harnesses whose data stays maximally reusable. A small design decision made early compounds into billions of dollars of training data being either portable or stranded.

![Harness lock-in loop](../../images/harness_pretraining/lockin.png)
<!-- GRAPH (lockin.png): a self-reinforcing loop diagram: "Ship harness A" →
     "User data is in harness-A format" → "Train on it" → "Model gets better
     at harness A specifically" → "Switching to harness B gets costlier" →
     back to start. A dashed escape arrow labeled "re-render / augment
     trajectories?" pointing out of the loop. Takeaway: the flywheel that
     helps you also traps you. -->

## Pretrain → Harness

### Bias in Pretrain

The reverse link starts from a fact about pretraining data: we filter it hard. Gibberish, broken code, and incoherent text are scrubbed out because training on them hurts. But the filtering teaches a lesson we never intended: *text that appears in context is correct* — in the pretraining distribution, it almost always was.

So if I write "1+1=3" in the context, the model's prior says to treat it as true. Not because the model is gullible, but because confident false statements were mostly removed from its training diet. It has essentially never seen trustworthy-looking context that is wrong, so it has no reflex for handling it.

A subtler coding example: ask a model for a CUDA kernel, and at some point it writes a Triton version — from then on it will continue in Python essentially forever, even if you wanted CUDA C++. Why? Because in pretraining data, **a file that is half Python is almost certainly all Python** — it is a Python file. Language-switching mid-document is vanishingly rare in the corpus, so once the trajectory enters Python, the prior for staying there is overwhelming. The model isn't being stubborn; it is doing exactly what next-token prediction on real files taught it to do.

The right mental model is biological: **pretraining trains the basic reflexes, and some reflexes will kill you in the wrong environment.** The human gasp reflex is great on land and fatal underwater. Divers don't retrain the reflex out of their nervous system; they learn an explicit rule — control your breathing — that overrides it in that environment. The reflex stays; the rule sits on top.

### Harness is the Symbolic Layer

That is what the harness is: the symbolic layer on top of the model's reflexes.

Humans have a great symbolic-layer skill: we are good at *forgetting and restarting*. When an approach is poisoned, we walk away, sleep on it, and come back with a fresh view. Models, by construction, sample forward — every failed attempt stays in context and keeps biasing the next token, as [this blog on why humans don't just sample](https://joyemang33.github.io/blog/2026/humans-dont-just-sample/) puts it well. But the model doesn't need to learn this. We can add it manually: context compaction, subagents spawned with clean context, a harness rule that says "after two failed attempts, restart from scratch instead of patching." Each is an explicit override of a pretraining reflex — the diver's breathing rule.

This leads to the claim I most want to emphasize, because it cuts against the field's current instinct:

**When we distill human problem-solving strategies into agents, we should not distill the human actions into the model itself. We should distill them into the harness.**

### Why?

Because those behaviors are *against pretraining probability*. "Distrust your own context" fights the filtered-corpus prior. "Abandon a half-Python file and restart in CUDA" fights the it's-a-Python-file prior. Post-training can bend these priors, but you are pushing against the accumulated mass of the entire corpus — expensive, and the prior leaks back through in situations the post-training didn't cover.

The harness sidesteps the fight. Instead of training the model to distrust a poisoned context, the harness *deletes the poisoned context and starts fresh* — and now the desired behavior is the high-probability continuation, not a suppressed reflex. The harness doesn't override the prior; it re-arranges the input so the prior points the right way. That is a cheaper and more reliable mechanism, and it is why the knowledge gained from each solved problem should land in a harness component — a skill file, a rule, a context-management policy — rather than being forced into the weights.

![Reflex layer vs symbolic layer](../../images/harness_pretraining/symbolic_layer.png)
<!-- GRAPH (symbolic_layer.png): two-layer stack diagram. Bottom layer:
     "Model (pretrained reflexes)" with example reflexes listed inside:
     "context is correct", "half-Python file → stays Python", "keep sampling
     forward". Top layer: "Harness (symbolic rules)" with matching overrides:
     "verify / distrust inputs", "restart with fresh context", "forget and
     retry". Draw each override arrow pointing down at the reflex it corrects.
     Side annotation with the diver analogy: reflex = gasp, rule = "control
     your breathing". Takeaway: the harness overrides reflexes instead of
     retraining them. -->

## Conclusion

The two links form a loop. Going one way, the harness defines the data flywheel: it fixes the trajectory format, turns users into a training-data source, and — once bootstrapped by RL — becomes the dominant driver of improvement, at the cost of a lock-in dilemma we don't yet know how to escape. Going the other way, pretraining hands the harness a set of reflexes, some wrong for agentic environments, and the harness is the symbolic layer that must break them manually, task by task.

Treating this as one co-design problem changes real decisions. Harness designers should ask which pretraining reflexes they are silently inheriting, and which ones their trajectories will reinforce in the next training run. Pretraining teams should ask which corrections can be left to the symbolic layer instead of being fought in the weights. And anyone distilling human expertise into an agent should default to putting it in the harness — the model already has the reflexes; what it needs is the rulebook.

<!-- ### Acknowledgement -->
