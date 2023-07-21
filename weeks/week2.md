---
layout: default
title: Week 2
---

# Week 2 - A closer look at CodeGen 

---

## Overview
The second week's focus has been to gain an in-depth understanding of pre-trained models of CodeGen family, particularly. looking at a [350M], [7B] size, multi-language models, we also looked into [NSQL models] which are trained on the variants of CodeGen models. 

```
from  datasets  import  load_dataset

ds_sparql = load_dataset("bigcode/the-stack", data_dir="data/sparql", split="train")

```

## Insights
The number of general-purpose queries used in the first round of fine-tuning 

Assuming the data was actually usedin these twoit is much smaller in size. 


A key observation was related to NSQL approach for augmenting user's raw input, i.e, a query articulated in natural language, 


A challenge was related to the 7b model size which is more than 26G. The inference time on CPU was more than 80 seconds. Since 4-bit quantization seems to be not supported for the current checkpoints (to be verified) the model was loaded on a dual RTX 3090 GPU, and inference time was reduced to be less than 6 seconds. 


## Next week plan
Get prediction performance scores based on existing datasets such as LC-QUAD-2, that contain pairs of natural languge and SPARQL queries. 

This inspection should provide crucial insights into their effectiveness in translating natural language queries into SPARQL queries and help us further refine our approach.
Meantime, the other parallel task is to collect synthetic SPARQL queries by having NSpM template-based query population (see previous GSoC projects) up and running synthetic I will Running the Pipeline for template-based synthetic data generation 

----
[NSQL modesl]: https://huggingface.co/NumbersStation
[350M]: https://huggingface.co/Salesforce/codegen-350M-multi
[7B]: https://huggingface.co/Salesforce/codegen25-7b-multi
[see previous GSoC projects]: https://github.com/dbpedia/neural-qa/tree/master/gsoc/saurav 
