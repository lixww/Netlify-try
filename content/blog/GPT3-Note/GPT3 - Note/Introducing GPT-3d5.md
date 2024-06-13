+++
title = "Introducing GPT-3.5"
date = 2023-03-18
[taxonomies]
  tags = ["GPT"]

[extra]
  toc = true
+++


# Previous Works

# Pattern in Model Training

- What is the **Model**?

GPT-3.5 is developed based on the **autoregressive transformer language model** with super heavy pre-training knowledge gained from large internet corpus, which unsupervised pre-training process is guided by autoregressive method for language modeling. After pre-training, it is then supervised training by in-context learning (or few-shot learning) based on the form of prompts. Last the fine-tuning engineering part (RLHF & instruction tuning) would further make the model task-specific and aligned.

- What is the **Pattern**?

The pattern is that how we would explain about how does the unsupervised learning work. How does the objective or target function interact in the fine-tuning process. Also the similarity between explaining autoregressive and recurrent (no, there is still a difference, an evolution difference?). And scaling is the honey always. Training data processing is the very first point to start with as well.

- What is the **Reinforcement Learning from Human Feedback (RLHF)**?

> Reference: [ChatGPT 背后的“功臣”—— RLHF技术详解](https://huggingface.co/blog/zh/rlhf)
> 

First, pre-train a large language model (LLM), eg GPT-3, Gopher, etc.

Second, train a feedback model (RM), used to give human-preferred feedbacks for an output of the LLM. Like scoring the result and ranking. Note that a RM could be large as well. For example, OpenAI utilize a 175B-LLM together with a 6B-RM.

Third, use RM to fine-tuning LLM by reinforce learning.

# Features & Limitations

# Future & Impacts