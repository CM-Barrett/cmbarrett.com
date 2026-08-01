In our [[Agents for Pipelines - Intro|past installment]], we covered some basic terms and concepts underpinning agentic systems: LLMs, tokens, context, harnesses, and agents. Now it's time to talk about two major components of agentic systems we'd overlooked: tools, and multi-agent workflows.  In truth, both of these can (mostly) be thought of as issues to do with context.

## Tools

A tool is an external command that the agent can call. More precisely, a tool is an LLM response that indicates it wants to execute something, which the harness running the LLM receives and then executes or denies. The script running the LLM *almost* always returns the output of the tool to the LLM, but sometimes it may have custom logic wrapping it to save on context tokens. For example, a harness that allows a tool call for a verbose process may only return the exit code by default, and require the LLM to request a more verbose response for troubleshooting.

Now technically, tools can be just about anything. A harness provides a list of available tools to the LLM, typically or not always by describing it as a json blob. The LLM calling the tool is in fact just generating another json blob based upon this. Knowledge of *how* calling the tool works must either be retrieved from the agent's context or the model's weights.

As a result, you'll find models are typically best at calling tools that are well represented in the training data, and very good at calling tools that are not represented in the training data but follow standard patterns and context as tools in the training data. To put it plainly, modern models tend to be very comfortable with the \*nix command line.

Remember that context size growing too large dilutes model output quality, and you'll understand why you probably don't want to shove a massive amount of documentation in context in order for the model to call a speciality tool. There are strategies you can take to alleviate that, such as only fetching the relevant subset of a bit of documentation, but it is a bigger burden you'll have to be aware of when programming an agentic system.

### What about MCP?

If you've followed agentic programming at all, you've probably heard a good deal about the [model context protocol (MCP) created by Anthropic](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro). It was an early attempt at providing a way for models to call tools, by making calls to a server to run the tools for them. With an MCP server, the model is picking from a set of pre-defined functions as opposed to composing a custom cli call, which can help reduce risks of the model chaining together commands in a dangerous way. MCP also makes gating things behind auth a bit easy - the initial handshake can require the user running the LLM to sign in, generating a scoped OAuth access token, and available commands can be easily restricted at this point.

Unfortunately, there are some downsides to using MCP over other tool approaches. For one, MCP server schemas chew up a good bit of context, and typically you get the entire set of commands listed when you ask for a list from the MCP server. Remember that you're generally paying for input tokens when running an LLM in addition to chewing up context space. [According to scalekit](https://www.scalekit.com/blog/mcp-vs-cli-use) and other benchmarks, MCP usage can be substantially more expensive.

There are also some security risks, though this is true in some form of every existing form of tool calling. MCP has received a lot of criticism from security professionals thanks to its use of stdio for local servers, allowing parameters to pass to the host system's shell without input sanitization or validation, [which Anthropic has stated is an intentional architectural choice](https://www.ox.security/blog/the-mother-of-all-ai-supply-chains-critical-systemic-vulnerability-at-the-core-of-the-mcp/). This means that if you expose an agentic system using a local MCP server, or one that launches subprocesses local to the server through the MCP protocol, you open yourself up to a nasty, well documented exploit.

That's just one of the security risks of MCP servers. External MCP servers that you/your org do not control are also a risk; attackers can hide malicious prompts inside tool definitions, or execute a rug pull attack and modify tool definition or behavior behind the scenes. If you're using an MCP server, make sure it's one you have source code access to and are running on your own systems. For a more complete picture of the security risks of MCP usage, check out [this article from the Cloud Security Alliance](https://labs.cloudsecurityalliance.org/research/csa-research-note-mcp-security-crisis-20260504-csa-styled/)

My personal take on MCP servers is that they are a bit over recommended. Frequently I've seen people building MCP servers for things that already have robust cli or api interfaces. In my opinion, writing an MCP server makes the most sense under all of the following conditions:

* You need the tool calls to execute on a separate server from the agentic system that you control
* You want to handle API tokens and OAuth at the same time at the server level
* You don't have a massive set of tools, or you're ok with paying the context price for it because the set of tools changes often enough that dynamic discovery at runtime is a feature

The rest of the time, I personally prefer either a cli tool or allowing the LLM to write a quick wrapper script for interacting via API. Mind you, there are risks to this approach as well. I'll get more into this in a later part of the series, but you can alleviate these risks substantially with sandboxing, human in the loop approval, and standard studio security practices.

### What about Skills?
You may have heard about skills in the context of coding harnesses like Claude Code. Skills, [another Anthropic backed standard](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), are simply markdown files describing how an LLM can accomplish a task, alongside optional additional files including python scripts. Skills can be a great way of enabling an agent to act in a consistent way while still having enough non-determinism to gracefully handle errors.

One notable thing about skills is that they only load lightweight metadata describing the skill and when it would be used at startup into the context; the full skill is only loaded when the agent uses it. This metadata is stored in the [frontmatter](https://github.com/Kernix13/markdown-cheatsheet/blob/master/frontmatter.md) the the SKILL.md file, and keeps track of the name of the skill, the description, and optionally the license, compatibility (or required environment), approved tools, and additional custom metadata. This saves a substantial amount of context vs an MCP server or multiple sets of instructions in the system prompt.

To boil it down, skills are an easy way of repeating context dynamically when an agent needs it, to encourage repeated LLM steps that aren't quite deterministic. Paired with tools, skills are an elegant way to dynamically enhance an AI harness without a ton of manual coding.

We'll get more into skills while we examine the architecture of building a harness in a later article in this series. 

## Multi-Agent Systems

I'm going to start this section with a bold take: multi-agent systems are most importantly a way of managing context to improve results. A multi-agent system defines multiple agents, which may or may not all use the same LLM. These agents can run in parallel, which can be a substantial performance improvement at the risk of chewing up tokens very quickly, but it's the context management we care about the most.

You may have heard of multi-agent systems associated with the idea of assigning roles to agents. This is a good idea, when done right. For example, in a multi-agent coding system, you'll get better results if one agent helps you come up with a plan, another agent writes the tests, a third agent writes the code but isn't allowed to change the tests, and a final agent reviews the result, even if they're all calling the same LLM model. This is advantageous even if these agents aren't running in parallel.

Similarly, good context management means you might spin up a set of these agents for each feature, bug, and task you have in the codebase. Of course, doing this naively means you'll be repeating input context tokens, so you might imagine an orchestrating agent that leverages cached observations to load dynamically into the context of sub-agents. We'll dig more into this in later posts. 

> [!info] What about personas?
> During early exploration of LLMs, many users got in the habit of giving their agents so-called "persona" prompts; for example, they might tell the code reviewing agent: "You are a highly critical, incredibly effective 10x tech lead at Google reviewing junior code." It was believed at the time that this improved model results, although I'm not sure if a rigorous study had been done back then.
> 
> As a result, a lot of people recommend crafting personas to this day. [Research suggests](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5879722&trk=public_post_comment-text), however, that precise instructions are more effective at getting quality results than personas, at least with modern models. 
> 
> So instead, you should phrase your prompt as: "Review this code critically, looking for X, Y, and Z types of errors"

Managing the context of agents within a multi-agent system can be a bit of an artform, which is another reason many people leverage orchestration agents to help build that context dynamically on the fly. This also gets a bit into cost management; you could use a bigger, more expensive model with a larger context to manage agents using a smaller model to do the work, allowing the expensive model to dynamically build their instructions and externally loaded knowledge.

Multi-agent systems can also allow for an added degree of resilience, by adding recovery/support agents to diagnose issues and flag them up to the orchestration agent for remediation. This can get very complex quite quickly, so I'll get more into this later, alongside exploring more interesting applications of parallel agents.

## It’s All Context in the End

Whether you’re determining how an agent executes a system command, deciding if an MCP server is worth the overhead, or orchestrating a suite of specialized agents to work in parallel, building agentic systems is primarily an exercise in managing the context for the model to work in. Hopefully I've helped to clarify some of the concepts in this space, and you have a better grasp on when you might apply each one.

Every bit of information, every tool call, and every agent you add into your system is an opportunity to enhance your system's capabilities, but it also can end up blowing up your costs. In addition to the financial impact, remember that a bloated context (currently) reduces model performance as well, which can cause your system to behave in unexpected and undesired ways. 

In the next part of this series, we'll cover achieving some pipeline tasks with a coding harness, and take a peek as to what's happening under the hood. 