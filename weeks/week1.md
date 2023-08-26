---
layout: default
title: Week 1- Revisit project roadmap
---

# Week 1 - Revisit project roadmap 🛣️

---

## Overview
The first week of the project was a reconsideration phase, centered around selecting the most suitable pre-trained model for this project. In the original proposal, I intended to use a general-purpose GPT languge model. This decision was based on a shared belief within the community that auto-regressive models are the prime candidates for tasks like ours- the translation of natural language queries into formal SPARQL queries. However, the rise of open-source models particularly designed for code generation prompted us to revisit my approach. After short-listing options, [CodeGen] family models were identified as promisign starting points due to their competetive performance in code generation tasks and openness (released under Apache-2.0 license). 


## Insights
While the roadmap for the project implementation may appear straightforward at first glance, several key questions have emerged that are critical to our progress:
- Which model size yields the best results?
- How can we effectively augment user input as a prompt to provide clues for relevant DBpedia entities?
- What training approach should we employ?
    - Zero/Few-shot learning
    - Adding a translation head with/without fine-tuning
    - Fine-tunning with general SPARQL queries
    - Fine-tuning with DBpedia-specific SPARQL queries 

In the course of my research on answering these questions, I came across [NSQL] project. It shares a similar vision to my GSoC project, as it aims to train foundation models explicitly for SQL query generation. The discovery of the NSQL project has provided valuable insights that will further guide our design decisions in the upcoming weeks. Inspired by their approach, a promising avenue is to fine-tune one of the CodeGen variants with a large courpos of general SPARQL queries (formal statements) and then further fine-tune the model with DBPedia-specific SPARQL queries. 




## Next week plan
For the upcoming week, the plan is to dive deeper into CodeGen and NSQL models as a deep understanding of their nuances will be instrumental for our project. We aim to inspect the data and approches used for pre-training selected modesl.

----
[CodeGen]: https://huggingface.co/docs/transformers/model_doc/codegen
[NSQL]: https://www.numbersstation.ai/post/introducing-nsql-open-source-sql-copilot-foundation-models

