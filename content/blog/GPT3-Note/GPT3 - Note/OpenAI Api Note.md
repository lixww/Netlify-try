+++
title = "OpenAI Api Note"
date = 2023-03-13
[taxonomies]
  tags = ["GPT", "OpenAI"]

[extra]
  toc = true
+++


# How many models choice do i have?

> Reference: [Models Overview](https://platform.openai.com/docs/models)
> 

| Models | Intro | Note |
| --- | --- | --- |
| GPT-3.5 (a set): gpt-3.5-turbo is recommended | understand  & generate natural language / code (language model) | gpt-3.5-turbo, gpt-3/5-turbo-0301, text-davinci-003, text-davinci-002, code-davinci-002 |
| DALL-E | generate & edit images given natural language prompt (image generation) |  |
| Whisper | convert audio into text (speech recognition) |  |
| Embeddings (a set) | convert text into a numerical form (text-relateness measurement) |  |
| Codex (a set) | understand & generate code, including translating natural language to code (coding suggestion) | code-davinci-002, code-cushman-001 |
| Moderation | (fine-tuned) detect whether text may be sensitive/unsafe (ie sentiment analysis) |  |
| GPT-3 (a set) | understand & generate natural language (language model) | text-curie-001, text-babbage-001, text-ada-001, davinci, curie, babbage, ada |
| Point-E | generate 3D print clouds from prompts (3D-model generation) |  |
| JukeBox | generate music from prompt (music composition) |  |
| CLIP | generate text snippet from image (caption generation) |  |
|  |  |  |
|  |  |  |

## InstructGPT models

> Reference: [Model index for researchers](https://platform.openai.com/docs/model-index-for-researchers)
> 

InstructGPT models are trained in 3 ways:

- SFT: **supervised fine-tuning** on **human demonstrations**
    
    `davinci-instruct-beta`
    
- FeedME (feedback made easy): **supervised fine-tuning** on human-written demonstrations and on model samples rated 7/7 by human labelers on an overall quality score
    
    `text-davinci-001`, `text-davinci-002`, `text-curie-001`, `text-babbage-001`
    
- PPO ([proximal policy optimization](https://openai.com/research/openai-baselines-ppo)): **reinforcement learning** with **reward models** trained from comparisons by humans.
    
    `text-davinci-003`
    

# 💰：How do tokens affect my billing plan?

![Pick from [Introducing ChatGPT and Whisper APIs](https://openai.com/blog/introducing-chatgpt-and-whisper-apis)](/blog/GPT3-Note/GPT3%20-%20Note/OpenAI%20Api%20Note/Untitled.png)

*Pick from [Introducing ChatGPT and Whisper APIs](https://openai.com/blog/introducing-chatgpt-and-whisper-apis)*

> Yes, there is a **limit** for the context a conversation can maintain or utilize.
> 

![Untitled](/blog/GPT3-Note/GPT3%20-%20Note/OpenAI%20Api%20Note/Untitled%201.png)

**4096-tokens** is the ceiling of a contextual conversation supported by the ChatGPT models.

> Reference: [Tips for Managing tokens.](https://platform.openai.com/docs/guides/chat/managing-tokens)
> 

# The underlying format consumed by ChatGPT models:

Structured format input is needed as for CharGPT models, which is called Chat Markup Language (CharML).

```json
[
	{
		"token": "xxx"
	},
	"xxx xxx xxx",
	{
		"token": "xxx"
	},
	"xxx xxx xxx"
]
```

# Best practice: tips for instructing chat models

From an example API call to `gpt-3.5-turbo`:

```python
# Note: you need to be using OpenAI Python v0.27.0 for the code below to work
import openai

openai.ChatCompletion.create(
  model="gpt-3.5-turbo",
  messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}
    ]
)
```

It could be noticed that the ‘system’ role may serve as the ‘few-shot’ part in the few-shot learning. In other words, **demonstrate with a few examples to the model**. This is called **prompt**.

![Reference: [*Chain of Thought Prompting Elicits Reasoning in Large Language Models* Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)](https://github.com/openai/openai-cookbook/raw/main/images/chain_of_thought_fig1.png)

*Reference: [*Chain of Thought Prompting Elicits Reasoning in Large Language Models* Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)*

Other than ‘few-shot’ hints, tricks to gain an ideal response are:

- State clear instruction
- Specify the format of expected output
- Ask model to think thoughtfully (step-by-step thinking / pros & cons debating / chain-of-thought prompting)
- Split problem into subquestions

> Reference: [OpenAI Cookbook - Techniques to improve reliability](https://github.com/openai/openai-cookbook/blob/main/techniques_to_improve_reliability.md)
> 

## How does fine-tuning work?

> Reference: OpenAI Cookbook, [Fine-tuning](https://platform.openai.com/docs/guides/fine-tuning) from OpenAI’s guide
> 

Actually, the **few-shot learning**. Several GPT-3 models are available to fine-tune: `davinci`, `curie`, `babbage`, `ada`. Fine-tune a find-tuned model is also possible.

STaR (Self-taught Reasoner) is one of the methods using prompts to fine-tune the model to reason out answers.

![Reference: [*STaR: Bootstrapping Reasoning With Reasoning* by Eric Zelikman and Yujuai Wu et al. (2022)](https://arxiv.org/abs/2203.14465)](https://github.com/openai/openai-cookbook/raw/main/images/star_fig1.png)

*Reference: [*STaR: Bootstrapping Reasoning With Reasoning* by Eric Zelikman and Yujuai Wu et al. (2022)](https://arxiv.org/abs/2203.14465)*