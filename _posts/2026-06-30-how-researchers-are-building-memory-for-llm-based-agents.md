---
layout: post
title: "How researchers are building Memory for LLM-Based Agents"
date: 2026-06-30
categories: [ai, memory]
tags: [agent-memory, llm-agents, memory-architectures, long-context, evaluation]
description: "A technical reflection on how researchers are building memory for LLM-based agents, from textual memory systems to more internal and parametric approaches."
---

After reading a number of papers on memory for LLM-based agents, it became clear to me that memory is not being built in one single way. The design of memory architectures depends on both the task the agent is meant to solve and the properties the system is trying to optimize for. When interpretability and explicit retrieval matter most, researchers keep memory external and textual. However, when researchers want memory to sit closer to the model's internal pipeline, latent or parametric approaches become more attractive design choices.

Below are some of the main directions I have seen researchers follow.

## 1. Textual Memory

Much like vanilla RAG, memory is stored outside the model and retrieved when needed to augment the agent's behaviour or response. However, researchers do not all organize this memory in the same way.

**[Generative Agents](https://arxiv.org/abs/2304.03442).**  
This is a memory-centered agent architecture that uses a language model to store, synthesize, and apply relevant memories to generate believable long-term behaviour. Its main components are:

- **Memory stream:** a long-term memory module that records a comprehensive list of the agent's experiences.
- **Reflection:** a mechanism that periodically turns raw experiences into higher-level conclusions.
- **Planning:** a component that translates those conclusions and the current environment into detailed behaviours for action and reaction.

Generative Agents' key idea is that a loop of memory, reflection, and planning can produce believable long-term agent behaviour.

**[StructMem](https://arxiv.org/abs/2604.21748).**  
StructMem introduces a more organized alternative to storing memory as a flat text log. Instead of keeping all past experiences in the same form, it first stores memory as event-level structure and then builds cross-event memory by linking related events over time.

Its hierarchy has two main stages:

1. **Event-level binding** - For each dialog or event, the model extracts factual information and relational information, grounding both in time.
2. **Cross-event consolidation** - Related past events are retrieved and synthesized as higher-level memory.

So, compared with Generative Agents' memory stream, StructMem pushes memory toward event structure and time-based organization.

**[MAGMA](https://arxiv.org/abs/2601.03236)**  
MAGMA also tackles the issue of storing memory as a flat log. However, while StructMem does event-level structuring, MAGMA explicitly models different memory relationships in the form of multiple relational graph views. It is a multi-graph agentic memory architecture for long-horizon reasoning.

Its main thesis is that memory should not just be stored as past text, but as different types of relations:

- semantic relations
- temporal relations
- causal relations
- entity relations

Another important design choice is that retrieval is treated as query-guided graph traversal, not as one-shot lookup. Simply put, MAGMA can prioritize different edge types for different query types, such as causal edges for "why" questions or temporal edges for "when" questions.

**[A-MEM](https://arxiv.org/abs/2502.12110)**  
A-MEM is similar in spirit to StructMem and MAGMA in that it also tries to improve on flat memory logs. While StructMem is event-centric and MAGMA is a formal graph memory, A-MEM is a lighter, note-linked memory inspired by the Zettelkasten note-taking method.

Its memory process has three main parts:

- **Note generation** - New interactions are converted to notes with multiple structured fields.
- **Link generation** - When a new note is added, the system retrieves top-k most relevant historical notes and uses an LLM to decide whether meaningful links should be created.
- **Memory evolution** - Instead of making historical memory fixed, new memory can trigger updates to existing notes, which allows the memory network to evolve.

In short, A-MEM treats memory as an evolving linked note system: new experiences become structured notes, relevant historical notes are linked dynamically, and old notes can be updated as the memory network grows.

Taken together, these papers show that even within textual memory, researchers are not building memory in the same way. Some architectures keep memory as a broad stream of past experiences, others push it toward event hierarchies, explicit relational graphs, or evolving linked notes. The choice seems to depend not only on what task the agent is meant to solve, but also on what the designer wants to optimize for, such as interpretability, temporal grounding, relational reasoning, or retrieval precision.

## 2. Parametric and Internal Memory

While the previous group of papers keeps memory external and retrievable, another line of research asks a different question: can memory sit much closer to the model's internal pipeline itself?

**[MemoryLLM](https://arxiv.org/abs/2402.04624)**  
To strengthen the memory of LLMs, the authors of MemoryLLM propose a model with a fixed-size memory pool distributed across the hidden states of transformer layers. This memory pool can be updated with new knowledge even after deployment, without requiring standard model fine-tuning. The memory pool consists of memory tokens, which act as compressed hidden-state representations of previously injected knowledge.

Technically, MemoryLLM uses a base transformer, specifically Llama 2-7B, and augments each layer with a learned memory pool. Each transformer layer has its own memory block, and each memory block is a set of `N` memory tokens, where each token is a hidden vector of dimension `d`.

This approach is different from RAG or chat-history memory because memory tokens are stored as hidden vectors as part of the model pipeline. Unlike standard fine-tuning, the model's memory can be updated directly with new text after deployment, without retraining the full backbone. This paper could be said to have explored a middle ground between external memory systems like RAG and standard parametric memory in frozen model weights. It shows that memory can be internal, latent, updateable, and distributed across transformer layers.

Another work in this category is **[M+](https://arxiv.org/abs/2502.00592)**. This is an extension of MemoryLLM designed to address the issue of poor retention of very old information. Because MemoryLLM uses a fixed-size short-term memory pool, older information is gradually displaced as new memory tokens are written. M+ retains MemoryLLM's short-term memory pool and adds a separate long-term memory store where dropped memory tokens are archived instead of discarded.

Not only are old memories kept; they also augment generation. During generation, M+ retrieves relevant tokens from long-term memory and concatenates them with short-term memory at each layer, so the model can use both recent and distant latent memory. This work further strengthens the claim made by MemoryLLM about memory being internal and latent, and now not just updateable, but also capable of longer-term storage.

**[LongMem](https://arxiv.org/abs/2306.07174)**  
Another paper I found interesting is LongMem. Although it is a bit older (2023), I still found the idea quite stimulating. In fact, this paper pushed me to go read the famous *Attention Is All You Need* paper.

While MemoryLLM and M+ push memory into latent memory tokens distributed across transformer layers, LongMem takes a different route. Instead of building a dedicated latent memory pool inside the model, it stores past context as cached attention key-value states and adds a lightweight module to retrieve and fuse them.

Its architecture has three main parts:

- **Frozen backbone LLM:** the original pretrained transformer, used as a stable memory encoder
- **Cached Memory Bank:** stores attention key-value pairs from previous input segments
- **Residual SideNet:** a lightweight trainable network that retrieves and fuses memory for current generation

In simple terms, LongMem lets an LLM use long past context by storing previous segments as cached attention key-value pairs and using a lightweight SideNet to retrieve and fuse those memories during current language modeling.

Taken together, these papers show that researchers are building memory for LLM-based agents in meaningfully different ways. While the problem looks similar, their approaches to solving it are quite different. The design choice seems to depend not only on what task the agent is meant to solve, but also on what the designer wants to optimize for, such as interpretability, temporal grounding, relational reasoning, or retrieval precision. This is also reflected in the kinds of benchmarks used to evaluate them.

One interesting direction going forward is whether some domains will need more task-specific memory architectures. In some settings, flat logs may be enough. In others, especially where relations, time, and long-horizon reasoning matter more, more structured approaches may be necessary. Domains such as energy analytics and finance may require memory systems that preserve numerical trends, temporal signals, and intermediate analytical context, while legal tasks may depend more heavily on precise textual evidence, citation fidelity, and long-range reasoning over documents. If so, different domains may not just need better memory, but differently structured memory.
