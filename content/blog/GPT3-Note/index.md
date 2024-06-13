+++
title = "GPT3 Note"
date = 2023-03-07
[taxonomies]
  tags = ["GPT", "note"]

[extra]
  toc = true
+++


> [OpenAI Api Note](/blog/GPT3-Note/GPT3%20-%20Note/openai-api-note/)
>
> [Introducing GPT-3.5 ](/blog/GPT3-Note/GPT3%20-%20Note/introducing-gpt-3d5/)


<!-- *Shortcut*: [ChatGPT ](#chatgpt)  -->

---

# Language Models are Few-shot Learners

[Paper address](https://arxiv.org/abs/2005.14165)

# Abstract Takeaway

A large corpus of task-specific and fine-tuning **text datasets** are still needed nowadays. Set human’s ability as objective, **task-agnostic** and **few-shot performance** are two main goals to achieve. Here large amount of training data (**scaling up** language model to 175 billion params) is the German-beer. 

Problems to notice: **methodological issues** related to large amount of web corpora and some **failure on specific** **datasets**.

Being **Good at** overall: translation, question-answering, cloze task, on-the-fly reasoning, domain adaptation.

> **text-generation based on input prompts/conditions**: generating news articles, product description, creative writings… (text completion). language translation, as well.
> 

# Introduction Takeaway

- History-sense

Single-layer representations (**word vectors**) → task-specific architectures 

RNNs with multiple layers representations + **contextual state** → task-specific architectures

Pre-trained recurrent (**transformer language models**) → task-agnostic architectures, task-specific datasets & fine-tuning

- Limitation of the task-specific properties of datasets & fine-tuning is better to remove.

**Meta-learning** is one of the possible solution, meaning that there could be some repeated sub-tasks (**in-context learning**) embedded within a single sequence and could be shared between sequences (the sequence is supposed to be forward-pass in the paper). But still **fine-tuning results** would be the winner.

Notice that there is an intension of meta-learning, which is to **not distinguish** the origin of learning new tasks from scratch at inference time or the fact of pattern recognition correlated to the training samples.

Another way to ease the limitation is the **scaling** of ****training with increasing capacity of the transformer language model (100 milli params → 300 milli → 1.5 billi → 8 billis → 11 billi → 17 billi). Log loss is proved to correlate well with multiple **downstream tasks** by the scaling practice. 

- **So could in-context learning benefit from the scaling strategy?** It is tested by training GPT-3 to measure the ability of in-context learning.

Three cases are set for this test: 

a) **few-shot learning / in-context learning**: few **demonstrations** (ie input-output pairs to demonstrate what the prompt is like and what expected output would be preferred) are given to the model **during the development process**, the amount of demonstrations would be form 10 to 100 (ie the maximum examples the model able to gather for learning context).

b) **one-shot learning**: one demonstration only.

c) **zero-shot learning**: no demonstrations and **only an instruction** (instruction is like to tell the model ‘Generate the summary for the attached article’ directly).

> Quick development of a model: architectures → datasets → training → after-training (fine-tuning / task-desired). In this process, a trend of moving **from task-specific to task-agnostic** is presenting. In other words, to **generalize** the model so that it can be the basic of solutions across sites.
> 

# Methodologies Takeaway

![Picked from paper. Explain how to differentiate the 3+1 cases mentioned for evaluating the GPT-3.](GPT3%20-%20Note/Untitled.png)

*Picked from paper. Explain how to differentiate the 3+1 cases mentioned for evaluating the GPT-3.*

This work is about **evaluating the three cases** (ie Few / One / Zero - Shot), which fall in the stage of ‘training’ actually. Consider if without fine-tuning, how well can the **pre-trained model** achieve?

- Training which model? The architecture is the same as GPT-2. Main exception is the **parameters amount** (125 billi → 175 billi).
- **Datasets**? [Common Crawl](https://commoncrawl.org/the-data/) + [WebText2](https://openwebtext2.readthedocs.io/en/latest/) + Books1 + Books2 + English-Wikipedia
- **Evaluations**?
    
    For one correct completion from several options, compare LM likelihood of each completion.
    
    For most tasks, compare per-token likelihood.
    
    For several tasks, use normalization of the unconditional probability of each completion. 
    
    F1 similarity score, BLEU, …
    

# Results Takeaway

Results on these **specific tasks**:

- traditional language modeling tasks: Cloze tasks, sentence/pararaph completion
- ‘closed book’ question answering
- translation
- Winograd Schema-like tasks (ie determine which word a pronoun refers to)
- commonsense reasoning / question answering
- reading comprehension
- SuperGLUE benchmark suite
- NLI (natural language inference) (ie understand relationship between two sentences)
- on-the-fly reasoning, adaption skills, open-ended text synthesis
    - arithmetic
    - word scrambling & manipulation tasks
    - SAT exam questions
    - news article generation
    - learning & using novel words
    - correct english grammer

> Possible **concrete applications**: code & writing auto-completion, grammar assistance, narrative generation, improving search engine responses, answering questions…
> 

## Analysis Takeaway

Large models should be trained with the concern of **training data contamination**, specifically the test & evaluation datasets and the training datasets. **Memorization of training data** should be prevented intensionally. But due to happened-bugs and high training cost, this work turned to study the **impact of the detected dataset overlap**.

# Limitations and Future Works Takeaway

1. Weaknesses on **text synthesis**: **repeat** semantically at document level, **lose coherence** over super long passages, self-**contradiction**, **non-sequitur** sentences / paragraphs several times, difficult with **common sense physics**.
2. About reading comprehension: difficult with determining if two words are used the same way in a sentence and the **pronouns explanation**.
3. Deeper **structural & algorithmic limitations**: autoregressive language models do not include any **bidirectional architectures** (ie no **time-series** analysis) / other training objectives like **denoising** / re-reading and tuning response.
4. Still **task-specific** due to the pre-training objective. Consider piping inputs other than plain text (**modalities**: other forms of **interactions** such as videos, images..).
5. Poor sample **efficiency**: **incredible large corpus** are needed in training, comparing to those absorbed in human life-time.
6. It is still unable to **distinguish** the origin of learning new tasks from scratch at inference time or the fact of pattern recognition correlated to the training samples.
7. Hard to do **inference**. Maybe learning **distillation** would help.
8. Decisions are **not interpretable**. **Bias and prejudice** in content are **retained**.

# Other Impacts Takeaway

> Too good to distinguish synthetic text from human-written text.
> 
1. intensionally **misuse**
2. bias, fairness, **ethic** issues
3. **energy efficiency** (petaflop/s-days of computing: 1000~ vs 10~, GPT-3 vs GPT-2, 175B vs 1.5B)

---

# ChatGPT

[Reference: Introducing ChatGPT](https://openai.com/blog/chatgpt)

> A sibling model to **InstructGPT**, which task is to follow an instruction in a prompt and provide a detailed response.
> 

# Methods

Training using **RLHF** (Reinforcement Learning from Human Feedback) and the same methods as InstructGPT.

## InstructGPT

[Reference: Aligning language models to follow instructions](https://openai.com/research/instruction-following)

[Paper address: Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)

> Keywords: #language models, #**follow intentional instructions**, #GPT-3
> 

GPT-3 is fit to handle natural language tasks with text inputs formatted as in the data structure called ‘**prompts**’. GPT-3 is **not aligned** with users, in other words, it may generate **toxic responses**. 

**RLHF** (reinforcement learning from human feedback) is used to **fine-tune** GPT-3 in a **supervised** way and then here comes the InstructGPT. The ‘**prompts**’ here are provided by users through OpenAI’s api, which would receive **manual review** before putting into the training dataset. Here is **how to** use RLHF method to fine-tune GPT-3:

get **prompts** from users → add to training dataset → labelers give **demonstrations** to GPT-3 → model generates outputs → **labelers** rank outputs → use **ranking** data to train a **reward model** → utilize rewards as in **PPO** (Proximal Policy Optimization) to train GPT-3 again.

![Picked from InstructGPT’s reference. Explain how to fine-tune GPT-3.](https://cdn.openai.com/instruction-following/draft-20220126f/methods.svg)

*Picked from InstructGPT’s reference. Explain how to fine-tune GPT-3.*

> **Frauds** of previous GPT-3 to tackle: **make up** facts, **toxic output** generation
> 

Labelers think 1.3B InstructGPT has better outputs than 175B GPT-3.

The InstructGPT work is more like an ‘**alignment**’ practice. Limitations of it mainly are: aligning to **labelers preference** is still suspicious, model would **bias** towards the **cultural values of English-speaking people**, susceptible to **instructions behaviors**, frauds from previous GPT-3 are **still exist.**

* As we can see👇, the model-training parts of InstructGPT and ChatGPT are **almost the same**. Due to the API design of GPT-3 and GPT-3.5, the main difference is the **amount of model parameters**, and thus the **maximum tokens amount** is different (from 2049 to **4096** specifically). Such parameters scale difference could be the result of so called ‘**data collection setup**’ in the Introducing ChatGPT.

![Picked from ChatGPT’s reference. Almost the same as InstructGPT’s.](GPT3%20-%20Note/ChatGPT_Diagram.svg)
*Picked from ChatGPT’s reference. Almost the same as InstructGPT’s.*

**Limitations**

- **plausible-sounding on incorrect** / nonsensical responses (due to: there is no true/fake distinguish hint in training data, to be cautious vs. to be knowledgable, the [**behavior cloning problem**](https://www.alignmentforum.org/posts/BgoKdAzogxmgkuuAt/behavior-cloning-is-miscalibrated) in **imitation learning** which point out the difficulty of being more confident on knowing more than human as a model )
- **sensitive to prompt / instructions**, that’s why we have **OpenAI Cookbook**
- **verbose** & overuses some phrases (eg. im sorry im just an AI model…)
- should **ask** more on **clarifying questions** when there is not enough context
- harmful instructions / biased responses **still exist**