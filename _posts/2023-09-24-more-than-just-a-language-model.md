---
layout: post
title: More Than Just a Language Model
date: 2023-09-24 11:59:00-0400
description:
tags:
categories:
giscus_comments: true
related_posts: false
thumbnail:
featured: false
---

In my [previous post]({% post_url /2023-09-22-simple-chatbot %}) I discussed that, at a very high level, one can think of a Large Language Model (LLM) as a machine that takes an input (a prompt) and returns the completion of that input.

For example, given the prompt,

```
What is the largest planet in the solar system?
```

OpenAI's GPT-3.5-turbo model returns a completion of

```
The largest planet in the solar system is Jupiter.
```

Moreover, one can modify the prompt given to the model to get an answer with a specific structure.  For example, given the modified prompt (where I am precise to specify that a single word should be returned as the answer),

```
What is the largest planet in the solar system?  Use one word in your answer.
```

the same GPT-3.5-turbo model returns a completion of

```
Jupiter.
```

This manipulation of prompts to produce better results from the model is known as *prompt engineering*.  However, it would be missing a big part of the potential of LLMs to think that prompt engineering is *only* about manipulating prompts.

# What Really is a Large Language Model

To get a bigger view of how to think about LLMs, let's look at the main parts of a traditional computer at a **very** high level.

Specifically, a traditional computer has:

* A **CPU** for computation
* Fast **volatile memory** for storing information
* Slower **persistent memory** for storing memory between program invocations
* An **interface** such as a **programming lanauge** to instruct the CPU on what things to compute
* **Peripherals** that allow data to be collected from different sources to be used by the CPU
* A **BUS** for allowing the different components to communicate

This description is very high level and glosses over a lot of technical details, but conveys the general idea of the different parts of a computer.

Now consider the following (somewhat contrived) prompt for an LLM:

```
You are a helpful assistant.

Your Knowledge:

* The average speed of a Corgi dog is 23 mph

Your Process:

You use a Thought/Action/Action Input/Observation pattern:
'''
Thought: The thought you are having to solve the task
Action: The action you want to take.  One of [search]
Action Input: The input to the action
Observation: An observation based on the action
'''

(...this Thought/Action/Action Input/Observation pattern can repeat N times)

== TOOLS ==

You have access to the following tools:

Tool 1:
Action: lookup
Action Input: The query to search
Description: Used to search your persistent memory

Tool 2:
Action: search
Action Input: The query to search
Description: Used to search the internet

== OPTIONS ==

Select ONLY ONE option below:

Option 1: To use a tool, you MUST use the following format:
'''
Thought: Because of the my observations, I think ...
Action: MUST be one of [search]
Action Input: The input to the tool
Observation: The observation from the action
'''

Option 2: To output the final answer, you MUST use the following format:
'''
Final Answer: The final answer for the task
'''

Constraints:
* You MUST use any information in Your Knowledge section before using any tools
* You MUST use a tool if the Your Knowledge section doesn't contain the info you need

Your Task:

Determine if a Corgi dog can run faster than a typical human.
```

which a LLM completes as

```
A Corgi dog can run faster than a typical human.
```

This uses the [ReAct](https://arxiv.org/abs/2210.03629) framework for structuring prompts to get LLMs to **reason** and **act** to solve the problem given to them.

If we look at this prompt we'll notice a few things:

1. The LLM is acting like a computational device (similar to a CPU).  However, instead of performing arithmetic operations, it is completing input text.

2. The **Your Knowledge** section, and generally the entire prompt acts like **volatile memory**

3. The use of ReAct allows the LLM to use tools, in this case the **lookup** tool, which is similar to a traditional computer accessing **persistent storage**

4. The description of the prompt, in particular the ReAct framework, acts as an **interface** to instruct the LLM what to compute, very roughly similar to programming languages in traditional computation.

5. The tools that the ReAct framework allows the LLM to use are similar to **peripherals** a traditional computer program can use.

6. A program that invokes the LLM, gives it input, gets its response, and invokes a tool if needed, acts similar to a **BUS** in a traditional computer.

# What Does this Mean?

From this, don't think of a LLM just as a machine that completes input text.  Just as thinking a traditional processor is just a machine that can perform arithmetic quickly, thinking of a LLM as a machine that can answer questions, or chat like a person misses the bigger picture of what an LLM can do.

Instead, as Andrej Karpathy puts it in his quote below, think of LLMs as a new type of compute resource: **a general-purpose differentiable computer**.

<div style="display: flex; justify-content: center;">
  <blockquote class="twitter-tweet"><p lang="en" dir="ltr">The Transformer is a magnificient neural network architecture because it is a general-purpose differentiable computer. It is simultaneously:<br>1) expressive (in the forward pass)<br>2) optimizable (via backpropagation+gradient descent)<br>3) efficient (high parallelism compute graph)</p>&mdash; Andrej Karpathy (@karpathy) <a href="https://twitter.com/karpathy/status/1582807367988654081?ref_src=twsrc%5Etfw">October 19, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 
</div>

Next, don't think about prompt engineering as just manipulating prompts to get an LLM to perform the action that you want.  Instead, just like programming languages allow thinking at a higher level to instruct a CPU on what to do, **think of prompts as being like a special programming language to interact with the LLM**.

That is, although natural language can be used to interface with an LLM, papers about [Chain-of-Thought](https://arxiv.org/abs/2201.11903), [ReAct](https://arxiv.org/abs/2210.03629), and other techniques demonstrate that a specific structured language can be used to *better interface* with LLMs.

Thus, just as one needs to understand how computers work to really understand how to utilize a programming language, one must really understand how LLMs work to fully utilize prompts.

Last, don't just think of tools or prompt techniques as interesting hacks, but rather as fitting into the bigger picture that LLMs are a new form a compute.

In summary, LLMs **are a new form a compute** and advances in prompt techniques are showing how to best harness the power of that compute.  Although this new compute may be analogous to traditional compute, interacting with LLMs requires novel techniques that continue to make research into prompt engineering very exciting.

# The Future

Right now LLMs are being used in a lot of places, in particular for knowledge lookup and in chat interfaces.  I think this is great, but it makes me think about how when processors were first commercially available, they were used in places where performing a lot of fast arithmetic was needed.

I then compare that to where we are now knowing that using a processor for arithemtic is only scratching the surface of what can be done.  That is, when processors first came into use, for many people, the idea of the World Wide Web, augmented reality, or even just a computer on every desk was unimaginable.

In the same way, I wonder where we will be in 30 years.  Will we look back at our use of LLMs now and think, "Wow, the people then didn't have any idea just what would be possible with what they have just created"?
