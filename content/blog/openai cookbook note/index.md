+++
title = "OpenAI Cookbook - Note"
date = 2024-05-20
[taxonomies]
  tags = ["Prompt", "OpenAI", "GPT", "Note"]

[extra]
  toc = true
+++

# Better Prompts

> Reference: [Techniques to improve reliability.](https://cookbook.openai.com/articles/techniques_to_improve_reliability)
> 
- Tricks work: `Lets think step-by-step` , `You are an expert`
- Give clearer instructions and context
- Split complex tasks into simpler subtasks
- *Structure the instruction to keep the model on task
- *Ask for justifications of many possible answers, and then synthesize
- *Generate many outputs, and then use the model to pick the best one
- Prompt the model to explain before answering
    - Zero-shot
        
        Using `Let's think step by step`. This works well for multi-step arithmetic problems, symbolic reasoning problems, strategy problems, and other reasoning problems.
        
        *(Reference: 2022 Large Language Models are Zero-Shot Reasoners)*
        
    - Few-shot example-based
        
        Q: xxx
        A: xxx
        Q: xxx
        
        This works for math problems, questions related to sports understanding, coin flip tracking, and last letter concatenation. Examples ≤ 8 shall be enough.
        
        *(Reference: 2022 Chain of Thought Prompting Elicits Reasoning in Large Language Models)*
        
- Fine-tune custom models to maximize performance
    - To meet the requirement of thousands of examples, costly to write, using few-shot prompt to generate candidate explanations — Self-taught Reasoner (STaR).
        
        *(Reference: STaR: Bootstrapping Reasoning With Reasoning.)*
        

## Extensions to chain-of-thought prompting

- Selection-inference prompting
*Reference: 2022 Selection-Inference: Exploiting Large Language Models for Interpretable Logical Reasoning*
- Faithful reasoning architecture
*Reference: 2022 Faithful Reasoning Using Large Language Models*
- Least-to-most prompting
*Reference: 2022 Least-to-most Prompting Enables Complex Reasoning in Large Language Models*

## Others

- Maieutic prompting
*Reference: 2022 Maieutic Prompting: Logically Consistent Reasoning with Recursive Explanations*

## More

- Self-consistency
*Reference: 2022 Self-Consistency Improves Chain of Thought Reasoning in Language Models*
- Verifiers
*Reference: 2022 TrainingVerifiers to Solve Math World Problems*

## Theories of reliability

- Probabilistic graphical models
*Reference: 2022 Language Model Cascades*

## Takeaways

[Bibliography from the article](https://cookbook.openai.com/articles/techniques_to_improve_reliability#bibliography)

![Untitled](OpenAI%20Cookbook%20-%20Note%20f7d2e973623443139a7c328db33725e6/Untitled.png)

# Prompt examples

- Instruction
- Completion
- Scenario
- Demonstration (few-shot learning)
- Fine-tuned. This means no need to add instruction, where special characters like ‘###’ or ‘→’ would tell the model what to do next.

Reference: [How to work with large language models.](https://cookbook.openai.com/articles/how_to_work_with_large_language_models)