---
layout: default
title: Week 2
---

# Week 2 - A closer look at CodeGen 

---

## Overview

The focus of the second week has been to gain a comprehensive understanding of the pre-trained models of the CodeGen family, especially the [350M] and [7B] size multi-language models. To provide more context, these models are designed to generate code from natural language prompts and are pre-trained on a vast corpus of code. Additionally, we explored the [NSQL models], which are trained on variants of CodeGen. This exploration will help us leverage their SQL query generation capabilities in our SPARQL query generation task.


## Findings and challenges 
A major part of our exploration was the examination of "[the-stack]" dataset, a massive dataset comprising permissively-licensed source code files spanning 358 programming languages. This dataset was instrumental in training the 2.5 version of the CodeGen models that we are intrested in. An important step was to identify the size of SPARQL-specific data within "the-stack." Using below snippet, we found approximately 24,000 rows of SPARQL-specific data (see the code snippet).

```
from  datasets  import  load_dataset

ds_sparql = load_dataset("bigcode/the-stack", data_dir="data/sparql", split="train")

```

This is a very small amount of data compared to the size of 1-milion samples of general queries seen during the pre-training phase of NSQL models. This disparity raises questions about the sufficiency of the SPARQL-specific data for our task and calls for a strategy to supplement this data, such as synthetic data generation.
 

Another key observation was related to NSQL approach for formatting model prompts based on user input. NSQL augments the user's raw input (a query articulated in natural language) with the description of tables (schema), which can guide the model's translation process. This observation inspired us to explore methods for optimally formatting prompts based in accord with DBpedia ontology. One potential soloution is to use ontology embeddings to link natural languge questions to related classes and properties.   


One key challenge we encountered this week related to the model size. The 7B CodeGen model has a significant size of more than 26GB, which led to a substantial inference time on a CPU—over 80 seconds. As 4-bit quantization did not seem to be supported for the current checkpoints (this is still to be verified), we decided to load the model on a dual RTX 3090 GPU instead. This change significantly improved the inference time, reducing it to less than 6 seconds.


## Next week plan
As we proceed into the third week, our primary objective is to evaluate the prediction performance of the selected CodeGen and NSQL models. We will use existing datasets such as LC-QUAD-2, which comprises pairs of natural language and SPARQL queries, for this purpose. This evaluation will provide crucial insights into the models' effectiveness in translating natural language queries into SPARQL queries, thereby helping us further refine our approach.

Simultaneously, we will also focus on a parallel task of collecting synthetic SPARQL queries. This task will involve getting the [NSpM] template-based query population running, a strategy that has seen successful implementation in [previous GSoC projects]. We aim to run the pipeline for template-based synthetic data generation, which will further expand our training data resources and potentially enhance the model's performance.

----
[NSQL models]: https://huggingface.co/NumbersStation
[350M]: https://huggingface.co/Salesforce/codegen-350M-multi
[7B]: https://huggingface.co/Salesforce/codegen25-7b-multi
[previous GSoC projects]: https://github.com/dbpedia/neural-qa/tree/master/gsoc/saurav 
[the-stack]: https://huggingface.co/datasets/bigcode/the-stack
[NSpM]: https://github.com/dbpedia/neural-qa
