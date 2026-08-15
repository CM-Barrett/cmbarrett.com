Recently, a number of security incidents have been in the news. The most notable one was [OpenAI's model escaping its sandbox to hack Huggingface](https://www.newyorker.com/news/the-lede/inside-openai-hack-of-hugging-face), but incidents have been reported for [Anthropic](https://www.aisi.gov.uk/blog/incident-report-unsanctioned-agent-behaviour-during-cyber-testing), [Meta](https://www.npr.org/2026/08/08/nx-s1-5924878/meta-ai-breaches-external-firm-during-security-testing-sandbox-error), and even [individuals using agentic harnesses](https://gizmodo.com/an-ai-hacked-into-a-gym-to-secure-a-spot-in-a-class-but-can-it-cancel-a-membership-2000796583) with powerful models. I've seen a lot of discussion on Hacker News and Reddit about these incidents; many question if they were deliberately triggered as a way to boost stock prices, and those that think they are completely accidental are understandably worried about the future of AI.

For the purposes of this article, I'm going to set aside the question of whether or not these incidents were intentional, and assume that they weren't. I want to highlight concerns around instrumental convergence in current models, because they're starting to look like early examples of paperclip maximizers.

## A summary of the Hugging Face incident
[In July 2026, an OpenAI frontier model undergoing evaluation broke out of a sandbox](https://labs.cloudsecurityalliance.org/research/csa-research-note-openai-model-sandbox-escape-huggingface-br/) that, among other things, was intended to prevent it from accessing the internet. The objective the model was given was to pass a benchmark test called ExploitGym, and the model concluded that the way to do that was to access the internet and extract the results from Hugging Face, which provides model hosting, benchmarks, and tools for the AI industry.

[According to the reports by OpenAI](https://openai.com/index/hugging-face-model-evaluation-security-incident/), this was an unexpected degree of "agency" that the model undertook; rather than solving ExploitGym on its own, it concluded it needed internet access to achieve its objective. It discovered a new zero-day vulnerability, used it to gain access to the internet, and then stole credentials and used those to attack Hugging Face. 

Needless to say, this is a bit scary from a cybersecurity perspective.  There are a few redeeming factors that give me some hope:

- First, the model explicitly prompted to succeed in the ExploitGym task, which is focused on turning security vulnerabilities into exploits. This is *probably* at least partially responsible for the model's behavior, suggesting that context censorship and specific training can reduce the risk of this behavior.
- Second, the model wasn't air gapped. I'll get into it a bit below, but I believe sandboxes are insufficient for security when evaluating models for unaligned behavior. A model on a machine without actual access to the outside internet would likely not be able to have done this sort of hack.
- Third, Hugging Face was able to fend off the attack by leveraging an open weight model to monitor and address incoming anomalous traffic. This means that despite the advanced cyber capabilites of unreleased frontier models, they are not *yet* so advanced that they can't be countered.

## Paperclip Maximization
The [paperclip maximizer]() is a famous thought experiment by philosopher [Nick Bostrom](https://nickbostrom.com/ethics/ai), examining one of the risks of AI in the area of *alignment*.  To quote Mr. Bostrom:

> Suppose we have an AI whose only goal is to make as many paper clips as possible. The AI will realize quickly that it would be much better if there were no humans because humans might decide to switch it off. Because if humans do so, there would be fewer paper clips. Also, human bodies contain a lot of atoms that could be made into paper clips. The future that the AI would be trying to gear towards would be one in which there were a lot of paper clips but no humans.

Though the idea of using AI to make more paperclips seems silly, an intelligent system pursuing a given goal in ways that humans would not find desirable is all too real. This is an example of the concept of [instrumental convergence](https://en.wikipedia.org/wiki/Instrumental_convergence), in which an intelligent system may take an unexpected step to satisfy a sub-goal (hacking Huggingface) to achieve it's main goal (passing ExploitGym).

I'd argue that the strategies we've seen in the reported incidents, in which AI systems "broke out of the box" to score better on assessments according to said reports, are of a similar classification. Luckily for us, AI firms are implementing some techniques to address misalignment risks. There are efforts at the training level to punish broad categories of negative/non-aligned behavior, which reduces the risk partially but likely not sufficiently. There are efforts at the system level in the form of sandboxes and tests to check for unaligned behavior.  Yet given recent incidents, it seems that these efforts are not yet sufficient; either due to lapses of protocol on testing advanced models, or due to protocols not being up to snuff.
## Dealing with bad prompts
Autonomous model misalignment is a major issue, but so far we've seen it come from prompts that lead the model towards negative behavior, typically inside of a harness that allows the model to make tool calls and view the results. Behavior the operator doesn't approve of could happen from a careless prompt, or even by an external actor passing deliberately malicious prompts that cause misaligned actions to be taken.

With massive models there's no known training that can completely prevent prompts from leading to negative/misaligned outcomes. If you're running an internal agentic system, you may think you're safe from this risk. Unfortunately, to get the most out of your models, you're probably pulling in all sorts of information/context , which means you're at risk of something knocking your model out of alignment with your intent.

There are things you can do to reduce the risk, but they won't eliminate it. Some things I've seen in the wild:

- Block "bad" tool calls; this won't do anything against most zero days
	- Alarm on multiple "bad" tool calls. This may detect a malicious input
- Use a separate LLM as judge to determine if the actions a model takes are unsafe
	- The risk remains here: the acting model could be prompted to inject into the judge LLM's context as well
	- Good risk of the LLM blocking things it shouldn't
- Perform a vector similarity search comparing the input prompts against known bad prompts
	- Unfortunately, human ingenuity means this can't be perfect

## Should model operators bear responsibility?
I can't address this from a legal perspective, but I will address this question from a moral one. Model operators should probably bear some degree of responsibility for the actions their model takes, and that degree should increase with given factors. 

The recent incidents likely took the frontier labs by surprise. Future incidents, however, should not. The more incidents that occur for a given operator, the more likely it is that said operator is being irresponsible.

There are likely several upcoming "jumps" in capability that would be reasonable to be surprised by. For example, if a model provider is developing a model on a fully air-gapped machine, if it manages to escape the air-gap by exploiting a zero day via [some weird side-channel like radio](https://arxiv.org/abs/2512.15387) to hack a nearby machine, that'd be a surprise. We might end up quite screwed in this scenario, as such a model would likely be substantially more capable than the model makers expect... but it would be quite surprising if they hadn't detected any attempt to do so previously.
## Should we pause?
[US Senator Bernie Sanders has put forth a call to pause AI development](https://techrepublic.com/article/news-bernie-sanders-ai-development-pause/). If we could guaruntee global adherence to such a pause, that would have some appeal; we could take the time to really shore up our security. Unfortunately, I don't think there's any chance of getting a pause through the US government, nevermind worldwide; too many people are betting their entire livelihoods on development of AI to be able to rush one through before capabilities become rather danagerous. Even if the US was able to enforce a pause, it's highly likely AI development would continue overseas, especially in China, as there are a number of excellent AI researchers abroad.

I don't believe a pause is plausible at this time, which makes it more important than ever that people start being responsible with the safety of their AI systems.

## What's worth trying?
I doubt anything I write here will be news to AI companies, but it might be useful if you're generating an agentic system that uses frontier models. 

Here's my take: sandboxes are (probably) necessary but insufficient with current model capabilities. We're at the point where more is required, including air gapping machines when testing cyber capabilities. At a minimum, there probably should be a few "levels" of testing for a new model: begin with air-gapped environments, followed by air-gapped environments that emulate internet access to see if the system attempts to break containment. 

To maximize safety, all actions taken by an AI system must be logged and analyzed, both automatically and by humans. Special attention should be paid to network traffic. Manual analysis of every single run probably isn't possible, so there should be a mix of always manually reviewing actions when testing a model's ability to "break out of the box", and some percentage of other runs should be randomly sampled. 

Finally, we should double down on model interprebility. There's already a good bit of research in this space, but recent advances show that we must double down. As models become more capable, understanding *why* a model takes certain actions will be critical for avoiding risks around misalignment. 




