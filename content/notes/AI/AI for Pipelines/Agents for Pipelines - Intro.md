
## Intro
AI agents (or specifically, agents powered by large language models ) have enormous potential to benefit workflows in animation, especially pipeline and infrastructure workflows. They also have the potential to substantially increase a studio's technical debt, as well as massively spike costs, if used incorrectly. This series of articles aims to help animation industry professionals, such as TDs and SWEs, learn to use agentic workflows while avoiding common pitfalls.

At SIGGRAPH 2026, I attended a Birds of a Feather session on agentic workflows in production pipelines. My biggest takeaway was that the film industry lacked a clear common understanding of what agentic workflows are; there were a lot of misconceptions and outdated beliefs about agents, which I think would make adopting agentic workflows much harder than it should be. This isn't an indictment of any participant; there's a lot of hype, FUD, misconceptions, and outdated advice out there, so it's hard to figure out what the "right" way to do things is from an outside perspective.

I'm writing this in July 2026. I'm aiming to write from the perspective of first principles, so that if the field has advanced in a year or two, it's still more or less relevant. Still, if you're reading this in 2027 or later, read it with a critical glance; I'll update this section if I modify things to stay more current. As for why you might want to listen to me, I'm a former animation industry professional who has trained deep neural networks from scratch, hosted LLMs, and pioneered agentic workflows for production use cases. I've been in this intersection before most people had heard of the term LLM and I'm hoping to save you some of the stumbling blocks that I hit.

To start, we'll do a quick overview of the basics.

## Basic Concepts

### LLMs
**L**arge **L**anguage **M**odels, or LLMs, are the core of what most people are talking about when they discuss AI agents. LLMs are trained on massive quantities of text (and sometimes more), and learn different plausible outputs given an input. 

Most LLMs are using a transformer architecture underneath the hood, which has certain limitations. For example, while LLMs have proven fantastic at deduction in certain areas of mathematics, they don't seem likely to be capable of the sort of abductive reasoning. Abductive reasoning is the capability of generating novel hypotheses from incomplete data, like Einstein inferring general relativity. For more insight into this, I suggest reading the position paper by DeepMind research Tom Zahavy, [LLM's Can't Jump](https://www.tomzahavy.com/projects/llms-cant-jump)

An LLM, at it's core, is an autoregressive sequence predictor. It doesn't generate whole sentences at once; it predicts _one token at a time_, appends it to its own context, and repeats the process. 
### Tokens, Encoding, and Logits
If you've looked at LLM pricing, you've probably encountered services that provide inference priced by the token. These days, a token is (typically) four characters long, and acts as a basic unit of text for an LLM. Models break up your text into these tokens, encode them, run them through the neural network weights, and then decode the result. Tokens are not always necessarily whole words.

Encoding, is the process of converting the tokens into embedding space, a vector space that allows LLMs to train & run efficiently. Each token gets a unique id from a vocab list. A trained embedding model converts those tokens into high-dimensional vectors that fit within the model's embedding space. It's notable that the embedding model is trained to keep similar concepts "near" each other in this embedding space. Models using a transformer architecture will look at all words at once, so there is also a positional encoding element (or sometimes rotational, but we'll ignore that for now) added into the word vector to allow the model to understand the structure of the text.

During inference, the output of the resulting vectors get mapped back into unique words. This is typically done with something called an "unembedding matrix", where the vector is multiplied against the entire vocabulary size to determine the probability scores of the next token. This raw score output is called a logit, and then is mathematically transformed back to normal text.

To summarize: LLMs take input text, break it up, map it to unique ids, project those into multidimensional vectors, run them through the neural network weights, and then take the result, create a probability distribution of the most likely next token, and then convert that into english words. This is why you'll often see services priced by input/output tokens, and APIs will make mention of something called logits.

### Context
When we discuss "context" in LLMs, we're talking about the total sum of input text the LLM can access during reasoning. Context can range from static data manually entered in input prompts, to dynamically retrieved data obtained via a tool call. Context management can be critical for using LLMs on tasks; too little, and the LLM may give the wrong result, but if the context gets too long, LLM results degrade in quality. 

Context is fundamentally about memory and state management, and typically includes the output tokens generated by the LLM. In a film pipeline, state management is almost always deterministic. For example, opening v45 of character.usd is entirely unaffected by the existence of versions 1 through 44. With LLMs, however, the context explicitly alters present performance. In a typical session, the entire history of user inputs, model outputs (often including reasoning), tool calls, and more ends up chewing up space. As these tokens accumulate, we run into something frequently called context rot, where the underlying attention mechanism of the LLM gets diluted across a crowded vector space. This degrades performance, particularly around reasoning and matching user instruction, and increases costs.

**Scoping context appropriately is a central challenge in building systems with LLMs**

### Agents
When we say "agent" in the context of LLMs, we're talking about a process where a system is using an LLM to plan, reason, and execute tools in response to a given input. Typically, this involves a loop of some sort, where the agent may react to its own output. I would argue a key distinction of an LLM agent is that the LLM is making decisions on what tools, if any, to run in response to a given input.

Agents can get pretty complex pretty quickly, but they can also be quite simple. For example,  if you write a Python script that takes an input, passes the input to an LLM, exposes a basic arithmetic tool, and then prints the output, that *could* be considered an agent. It would be an evolution over a standard Python script, as it would allow the user to enter any text in natural language, and would likely only call the tool if the user asks it to do math in its input.  Mind you, it would be a somewhat wasteful use of tokens.

An agent loop is when we feed an agent back the result of its output and then act in response. Imagine an agent that can access a simple (restricted) Python REPL.  If the LLM is making a tool call to write Python in said REPL, you'll get a much better result if it can see the result of running the code, and modify it in the case of errors. 

Note that when an agent is reading the output of a tool call, it is in fact modifying its own context.
### Harnesses
A harness (sometimes called an agentic harness) is a system that manages one or more agents, their contexts, tool calls, output, and errors. The best known class of harnesses currently are coding harnesses, such as Claude Code, Codex, and OpenCode. These harnesses manage wrangling agents for the purpose of software development, and have evolved into their own convoluted ecosystem.

A harness does not have to fundamentally be tied to a given model, though harnesses from big AI companies like Claude will typically encourage (or require) use of their own proprietary models. A harness also does not need to be exclusively for coding; Claude Design (and open-source alternative Open Design) are notable graphic design harnesses. 

The approach a harness takes can have a big impact on the resulting work done with it. It's been found that some harnesses can cause models to score much better on benchmarks than they do under standard harnesses, showing the impact of context, execution, and tool management on the resulting work. As a result, make sure you choose a well regarded harness for your problem domain.

Generally speaking, most power users of AI likely will want to use a harness to interact with it. Your domain may benefit from a custom harness, but it's likely you can get started with a coding harness. I recommend getting started with OpenCode or Pi Dev, two open-source harnesses that make it easy to use a large number of providers, including local providers.

## A Note on Model Capabilities
LLMs are evolving at a breakneck pace, and it seems like every day there's a new model release that scores amazing on one benchmark or another. Different models have differing performance on tasks; for most tasks, the biggest, newest models tend to perform best. If you last tried coding leveraging LLMs back in early 2024, for example, you likely weren't particularly impressed. Modern state of the art models are a *substantial* upgrade over those earlier models, and have even been used to make breakthroughs in mathematics. 

Some models are trained to tackle certain tasks better, at the expense of other capabilities. The number of places where a smaller model trained this way can beat a much larger general purpose model are decreasing, but there are still some strange jagged edges. You can think of this as shaping the latent space of the model, allowing it to spend more effort/have more knowledge of one domain at the expense of another.

As a result, it's not plausible for me to keep a list of highly recommended models. I'll talk more about model selection, as well as budget management, in a later entry in this series.

## Conclusion & What’s Next

Now we have a common ground covering the basic idea of core mechanics: tokens, context boundaries, agent loops, and harnesses. AI agents aren't magic, but they are made up of quite a bit of complicated concepts wrapped up in simpler execution code. When built thoughtfully with robust harnesses and tight context scoping, they can automate complex, non-deterministic pipeline tasks that traditional scripts struggle with. When thrown together haphazardly, they become unpredictable, costly black boxes that generate subtle bugs across your production pipeline; to revive an old joke in deep learning, they become the high interest credit cards of technical debt.

In the next article in this series, we will move past the definitions and tackle common concepts in the agent programming space: how to think about context, prompts, tools, and multi-agent systems.