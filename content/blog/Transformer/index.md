+++
title = "Transformer"
date = 2024-04-01
[taxonomies]
  tags = ["ML"]

[extra]
  toc = true
+++


# From AutoEncoder to Transformer (?)

> How does Transformer evolve today?
> 

[2024 GPT on a Quantum Computer] Integrate quantum computing with LLM behind ChatGPT - Transformer architecture - , which introduced as Quantum Machine Learning.

[2024 Evolution Transformer: In-Context Evolutionary Optimization] Introduce Evolution Transformer, which performance improved from supervised optimization technique and updating invariance of the search distribution.

<div style="display: flex;">
    <div style="flex: 50%;">
        [2024 Empowering Data Mesh with Federated Learning] Introduce Federated learning into Data Mesh, a decentralized data architecture. Distributed and privacy-preserved.
    </div>
    <div style="flex: 50%;">
        <figure>
            <img src="Transformer%20fc5244b57a6d46d7a6cd5ed1d1b90104/IMG_2567.jpeg" alt="Parallel computing, Distributed computing, and Concurrent computing">
            <figcaption>Parallel computing, Distributed computing, and Concurrent computing</figcaption>
        </figure>
    </div>
</div>


[2024 PerOS: Personalized Self-Adapting Operating Systems in the Cloud] Proposed a personalized OS ingrained with LLM capabilities - PerOS.

## ‘Connection’


<div style="display: flex;">
    <div style="flex: 50%;">
        Encoder and decoder in a typical autoencoder.
    </div>
    <div style="flex: 50%;">
        <figure>
            <img src="Transformer%20fc5244b57a6d46d7a6cd5ed1d1b90104/Untitled.png" alt="Schema of a basic autoencoder. [Reference from Wiki.]">
            <figcaption>Schema of a basic autoencoder. <a href="https://en.wikipedia.org/wiki/Autoencoder#Interpretation">Reference from Wiki.</a></figcaption>
        </figure>
    </div>
</div>

<div style="display: flex;">
    <div style="flex: 50%;">
        <i>Encoding</i> concept is also adopted in Transformer. 
    </div>
    <div style="flex: 50%;">
        <figure>
            <img src="Transformer%20fc5244b57a6d46d7a6cd5ed1d1b90104/IMG_2568.jpeg" alt="Encoder-Decoder architecture in Transformer">
            <figcaption>Encoder-Decoder architecture in Transformer</figcaption>
        </figure>
    </div>
</div>

Transformer is for NLP, while AutoEncoder is for image processing.

> Recent research about AutoEncoder: [See here in arXiv](https://arxiv.org/search/?query=autoencoder+&searchtype=all&abstracts=show&order=-announced_date_first&size=50)
> 

## Quick review of Transformer

![[Reference: What are Large Language Models](https://machinelearningmastery.com/what-are-large-language-models/)](Transformer%20fc5244b57a6d46d7a6cd5ed1d1b90104/IMG_2636.jpeg)

*[Reference: What are Large Language Models](https://machinelearningmastery.com/what-are-large-language-models/)*

[1950, Shannon] Shannon, C. E. (1951). Prediction and entropy of printed English. *The Bell System Technical Journal*, *30*(1), 50–64. https://doi.org/10.1002/j.1538-7305.1951.tb01366.x

[2017, Vaswani] Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, L., & Polosukhin, I. (2023). *Attention Is All You Need*.