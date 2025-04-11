---
title: "AI Glossary"
last_modified_at: 2025-04-10T00:00:00-00:00
categories:
  - Glossary
tags:
  - Glossary
classes: wide
---

- **LLM**: large language model
- **LMM**: large multimodal models
- **test-time compute**: computational resources and processes required during the inference phase of an LLM. Directly impacts the performance, responsiveness and scalability of AI systems in applications. Strategies such as generating multiple candidate outputs (best-of-N sampling) are used to improve output quality at the cost of additional computation during inference.
- **transformer architecture**: most dominant architecture for LLMs, which is based on the attention mechanism. The attention mechanism allows the model to weigh the importance of different input tokens when generating each output token. This is like generating answers by referencing any page in the book. With transformers, the input tokens can be processed in parallel, significantly speeding up input processing. While the transformer removes the sequential input bottleneck, transformer-based autoregressive language models still have the sequential output bottleneck.
- **sampling**: the probabilistic process an LLM uses to construct an output. The probabilistic nature of LLMs requires to pay attention to inconsistencies and hallucinations.
- **greedy sampling**: to pick the outcome/token with the highest probability.
- **sampling variables**: variables affecting the sampling process, like *temperature*, *top-k*, and *top-p*.
- **temperature**: allows to redistribute the probabilities of the possible values used in sampling. The higher the temperature, the less likely it is that the model is going to pick the most obvious value. The lower the temperature, the more likely it is that the model is going to pick the most obvious value.
- **top-k**: a sampling strategy to reduce the computation workload without sacrificing too much of the model’s response diversity. The number *k* is limits the logits used for sampling. A smaller *k* value makes the text more predictable but less interesting, as the model is limited to a smaller set of likely words. 
- **top-p**: allows for a more dynamic selection of values to be sampled from. In *top-p* sampling, the model sums the probabilities of the most likely next values in descending order and stops when the sum reaches *p*. Only the values within this cumulative probability are considered. Common values for top-p (nucleus) sampling in language models typically range from 0.9 to 0.95. *top-p* does not reduce computation load, instead by considering the most relevant values for each context, it allows outputs to be more contextually appropriate. 
- **logit**: a token in the model’s vocabulary.
- **logit vector**: the size of the vocabulary of a model.
