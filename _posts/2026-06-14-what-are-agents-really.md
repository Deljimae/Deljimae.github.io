---
layout: post
title: "From AIMA’s Rational Agents to LLM Agents"
date: 2026-06-14
categories: [ai, agents]
tags: [ai-agents, llm-agents, aima, reasoning, tooling]
description: "A personal and technical reflection on how my understanding of agents evolved from AIMA’s classical definition to modern LLM agents."
---

<div class="post-epigraph">
  <p>“An agent is anything that can be viewed as perceiving its environment through sensors and acting upon that environment through actuators.”</p>
  <p class="post-epigraph__attribution">— Russell and Norvig</p>
</div>

As someone with an engineering background, my first encounter with agents was in CPE 471 (Artificial Intelligence I) back in undergrad. The textbook for the course was *Artificial Intelligence: A Modern Approach* (AIMA), Third Edition, by Stuart Russell and Peter Norvig.

AIMA presents four approaches to AI: **Thinking Humanly, Acting Humanly, Thinking Rationally, and Acting Rationally**.

![Four approaches to AI](/blog_assets/aima_four_approaches.svg)

*Figure 1. Four classical approaches to AI, adapted from AIMA.*

However, our focus for this article is **“Acting Rationally”**: the study of the design of intelligent agents.

## What Is an Agent?

In CPE 471, we used AIMA’s core definition:

> “An agent is anything that can be viewed as perceiving its environment through sensors and acting upon that environment through actuators.”

![Agent-environment loop](/blog_assets/agent_environment_loop.svg)

*Figure 2. A simplified agent-environment interaction loop, adapted from AIMA.*

- **Sensors** are the channels through which an agent receives information from its environment.
- **Actuators** are the means through which it acts on that environment.

In AIMA, a **rational agent** is not one that is always correct, but one that selects actions expected to achieve the best outcome given what it has perceived and what it knows.

We saw a vacuum cleaner example in class. With the help of a sensor, the agent perceives which square it is in and whether there is dirt in that square. Using the actuator, it can choose to move left, move right, suck up the dirt, or do nothing.

We also considered a simple example of a software agent. A software agent receives keystrokes, file contents, and network packets as inputs, and acts on the environment by displaying on the screen, writing files, and sending network packets. But honestly, the software example felt boring compared to the vacuum cleaner. It was not until I started working on modern LLM agents that the true power of a software-based agent finally clicked for me.

*Think of this article as “what I thought agents were then versus what I think agents are now.”*

## Modern AI Agents

Today, many people hear “agent” and immediately think of an **LLM agent**, but that is really a special case of the broader agent idea from AIMA.

For the purpose of this article, I would define an **AI agent** as a software system built around a model, typically an LLM, that perceives inputs from an environment, reasons over those inputs, and takes actions toward a goal. With that definition in mind, let’s map those old CPE 471 ideas to modern architectures.

Just like the textbook vacuum cleaner has sensors for perceiving its environment, **observations** are the modern analogue of sensory input. Today, they come from user messages, tool outputs, browser states, APIs, or retrieved documents.

Modern agents also possess actuators for taking action. In practice, this manifests as tool calls, browser clicks, code execution, API requests, sending messages, or writing files.

There is one more component that matters in both the vacuum cleaner example and the modern agent setting: the **decision or policy core**.

In simple agents, this may be a hand-written rule like:

- if dirty -> suck
- else move left/right depending on location

In modern LLM agents, this role is often approximated by the LLM together with prompting and orchestration logic. The model interprets the observations it receives, reasons over them, and decides whether to respond directly, call a tool, ask for clarification, or take no action.

All of this happens in a loop:

![Observe, decide, act, update](/blog_assets/observe_decide_act_update.svg)

*Figure 3. A simple modern agent loop: observe, decide, act, and update based on feedback.*

## Common Modern Agent Patterns

Once an agent has observations, possible actions, and a decision core, the next question becomes: how should these components be orchestrated?

One’s choice of pattern usually depends on the nature of the task the agent is expected to perform. These patterns do not all fall in the same category. Some are reasoning-action loops, some are memory/reflection strategies, and some are system-level orchestration choices.

Common modern agent patterns include:

- **ReAct (Reasoning and Acting):** an interleaved loop in which the model reasons, takes an action, observes the result, and then continues reasoning. [Paper](https://arxiv.org/abs/2210.03629)

- **Reflexion:** an agent pattern where the model critiques its past attempt and uses that verbal feedback as episodic memory for the next attempt. [Paper](https://arxiv.org/abs/2303.11366)

- **ReWoo:** unlike ReAct, which interleaves reasoning with tool observations, ReWoo first plans a sequence of reasoning and tool-use steps with deferred observations, reducing repeated prompt overhead. [Paper](https://arxiv.org/abs/2305.18323)

- **Advisor Strategy:** a cost-aware orchestration pattern where a smaller model runs the primary loop and escalates to a stronger “advisor” model only when needed. [Article](https://claude.com/blog/the-advisor-strategy)

- **Multi-Agent Systems:** systems that distribute subtasks across multiple agents, each responsible for a different part of the problem or workflow. [Overview](https://www.ibm.com/think/topics/multiagent-system)

## Closing

Looking back, the AIMA definition of an agent feels far more useful to me now than it did in undergrad, especially after seeing how its software applications have evolved. While the core definition itself has not changed, the systems built from it have.

Modern LLM agents are still agents in the classical sense: they observe, decide, and act. What has changed most is the sophistication of the decision core, along with the tools, memory systems, and environments surrounding it. Software agents can now manipulate complex tools, navigate diverse memory systems, and operate in dynamic environments.

In my next few posts, I want to examine the active research landscape driving this evolution. I will look more closely at the emerging benchmarks used to evaluate agent behavior, the tooling frameworks that make these systems possible, and the different paradigms of agent memory representation.
