+++
title = "On-Device Learning Note"
date = 2024-06-25
[taxonomies]
  tags = ["On-device ML", "Note", "Edge ML", "Tiny ML"]

[extra]
  toc = true
+++

[*Harvard Edge CS249R - Chapter 12 On-device Learning*](https://harvard-edge.github.io/cs249r_book/contents/ondevice_learning/ondevice_learning.html)

# Relevant Notions

On-Device Leaning, which is closed to TinyML, also called Federated Learning, is for embedded and edge ML deployments.

# Limitations & Directions

On-device learning has limitation with i)compute resources, and ii)dataset size, accuracy, and generalization.

Comparatively, cloud learning is more complex, more efficient, and larger.
- For complexity, on-device learning might limit the number of layers & parameters.
- For efficiency, on-device learning might need quantization & pruning.
- For large-size, on-device learning might need compression.

On-device learning has challenges in small dataset size, accuracy & efficient tradeoff, and generalization. Consider the speed requirement of some tasks and the non-IID (independent & identically distributed) data that on-device learning handling.
- Susceptible to over-fitting due to small dataset.
- Low-quality noisy user-generated data is affecting accuracy.
- While one iteration time is fast, overall training time could be long.
- Size & speed & accuracy tradeoff, think of an 8-bit quantized model vs a 32-bit one.

# On-device Adaptation

Resource consumption comes from → so to have corresponding solutions:
i)ML model itself → reduce the complexity;
ii)Optimization Process → modify optimizations;
iii)Storing & processing the using datasets → new storage-efficient data representations.

## Reducing model complexity

- Traditional ML
Naive Bayes classifier, support vector machine (SVM), linear regression, logistic regression, select decision tree.
- Pruning
Remove neurons or connections in the network, that don’t contribute significantly to the prediction.
    
    ![Fig 12.2 Network pruning.](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled.png)
    
    *Fig 12.2 Network pruning.*
    
- Existing frameworks, to reduce memory overhead, for continuous learning (lack of lightweight, back-propagation).
alibaba/MNN, apache/TVM, TensorFlow Lite.
- Cutting-edge methods
Quantization-aware scaling (QAS), sparse updates.

## Modifying optimization process

### Quantization-aware scaling

*Quantization* is a process of mapping a continuous range of values to a discrete set of values, including reducing precision of weights and activations from 32-bit floating point to lower-precision format eg 8-bit integers).

![Untitled](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled%201.png)

QAS tries to minimize the quantization error, by adjusting the scale factors during quantization, based on the distribution of weights and activations, without hyper-parameters tuning. 
This involves 2 steps: i)quantization-aware training, ii)quantization & scaling.

### Sparse updates

Reduce memory consumption of full backward computation, by pruning gradient during backward propagation, ie skipping computing gradients of less important layers and sub-tensors.
Contribution analysis is needed for optimizing sparse update scheme, which measures the the accuracy improvement from biases and weights.

### Layer-wise training

Sequential layer-by-layer training, instead of training end-to-end, avoid having to store intermediate feature maps.

### Trading computation for memory

Release some of the memory storing intermediate results, which recomputed as needed.

## New data representations for data compression

- Prioritize sample complexity, which is the amount of data required.
- Reduce dimensionality & volume of training data, eg 
i)by transforming training data into a lower-dimensional embedding & factorizing them into a dictionary matrix, multiplied by a block-sparse coefficient matrix; 
ii)by representing words from a large language training dataset in a more compressed vector format.

# Transfer Learning

A model developed for a particular task can be reused as the starting point for a model on a second task. Regarding the context of on-device AI, this is to fine-tune a pre-trained model using smaller dataset on the device. Knowledge transfer, so called sometimes.
Eg researchers (Esteva et al.,2017) developed a transfer learning model to detect cancer in skin images, which is pre-trained on 1.28 million images to classify a  broad range of  objects and then specialized for cancer detection, by training on a dermatologist-curated dataset of skin images.

## Pre-deployment specialization

Workflow:
start with a pre-trained model → fine-tuning → validation → deployment.

## Post-deployment specialization (on-the-fly personalization)

Workflow:
data collection → model fine-tuning → validation → deployment.

## Core concepts

- Source task, target task.
- Representation transfer, includes
(data) instance transfer; feature-representation transfer; parameter (weights) transfer.
- Fine-tuning.
- Feature extraction, ie use pre-trained model as a fixed feature extractor.

## Taxonomy

Inductive transfer learning; transductive; unsupervised.

![Untitled](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled%202.png)

## Constraints & considerations

- Domain similarity
- Task similarity
- Data quality an quantity
- Feature space overlap
- Model complexity

# Federated Model Learning

A solution to train models partially on edge devices, only communicate model updates to the cloud. 

## FederatedAveraging

Google proposed a algorithm, FederatedAveraging, in 2016.

![Performs SGD over several different edge devices.
Figure 12.5: Google’s Proposed FederatedAverage Algorithm. Credit: McMahan et al. [2017](https://arxiv.org/abs/1602.05629)](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled%203.png)

*Performs SGD over several different edge devices.
Figure 12.5: Google’s Proposed FederatedAverage Algorithm. Credit: McMahan et al. ([2017](https://arxiv.org/abs/1602.05629)).*

## Key vectors for further optimizing FL

- communication efficiency
- model compression
- selective update sharing
- optimized aggregation
- handling non-IID data
- client selection

*G-Board*, as an example of federated learning.

*MedPerf*, an open-source benchmarking for federated learning.

# Security Concerns

- Data poisoning
- Adversarial attacks
- Model inversion
- On-device learning security concerns:
transfer-based attacks; optimization-based attacks; query attacks with transfer priors.
- Mitigation of on-device learning risks:
similarity-unpairing (diminish the similarity between on-device models and full-scale models); CleverHans tool; defense distillation.
- Securing training data:
encryption; differential privacy; anomaly detection; input data validation.
- On-device training frameworks:
Tiny Training Engine; Tiny Transfer Learning; Tiny Train.

# Components in an AI Framework

*Reference: Chapter 4 AI Workflow, and [Chapter 6 AI Frameworks](https://harvard-edge.github.io/cs249r_book/contents/frameworks/frameworks.html)*

![Fig 4.1](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled4.png)

*Reference: [Chapter 4 AI Workflow. 4.1 Overview](https://harvard-edge.github.io/cs249r_book/contents/workflow/workflow.html).*

## Computational Graph

There are static one, ie executed after definition, and dynamic one, ie built on-the-fly.

![Fig 6.4.2](On-Device%20Learning%20Note%20906a832982624367b2e60f713507fa1d/Untitled5.png)

Ahead-of-Time compilation of static one results in faster execution compared with dynamic one.

Optimization advantages of static computational graphs:
- graph fusion
- constant folding
- memory layout optimization
- parallelization
- hardware-specific optimizations

## ML inference frameworks for embedded systems & microcontrollers

| TensorFlow Lite Micro | Tiny Engine | CMSIS-NN |
| --- | --- | --- |
| an interpreter | compiler-based | a software library |

*Comparison of above three frameworks, [see here](https://harvard-edge.github.io/cs249r_book/contents/frameworks/frameworks.html#tbl-compare_frameworks).*

## Future

ML systems Breakdown:
AI applications → Computational graphs → Tensor programs → Libraries and runtimes → Hardware primitives.

### High-performance compilers & libraries
*Horizontal perspective.*

TensorFlow XLA, CUTLASS (Nvidia), TensorRT (Nvidia).

### ML for ML frameworks

Hyperparameter optimization, eg Bayesian optimization, random search, grid search.
Neural architecture search (NAS).
AutoML.
