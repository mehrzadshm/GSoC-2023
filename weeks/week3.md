---
layout: default
title: Week 3
---

# Week 3 - Zero/Few shots  

---

## Overview

In the third week of my GSoC journey, our primary focus was to evaluate the initial capabilities of various models through zero and few-shot experiments, i.e., passing prompts with instructions to generate SPARQL queries towards DBpedia knowledge base. We experimented with three models including the multi-language variant of [CodeGen-350M], [Falcon-RW-1B], and [GPT-Neo-2.7B]. This testing provided us with crucial initial insights into the models' performances and laid the groundwork for our future investigation.


## Findings and challenges 
Through our zero and few-shot experiments, we learned that none of the models we tested—CodeGen-350M-multi, Falcon 1B, and GPT-NEO-2.7B—performed impressively out-of-the-box. It became evident that we might have to rely on more delicate promt engineering, or, extensive training and fine-tuning processes to boost their performances.


> A key insight we gathered is that none of these models were instruct-tuned. This realization has opened up new avenues for exploration, particularly the possibility of experimenting with instruct-tuned variants of the models, based on the hypothesis that such variants are more effective in understanding the instructions from user prompts. 


A significant challenge we encountered this week pertained to the control of the length of the generated sequences. We found it difficult to set the appropriate parameters to get the most accurate and concise SPARQL queries. In the below example, in adddition to other missing and inaccurate tikens, the model is coming up with new unwanted pairs of input/output. However, we noted that the models were capable of some intresting generalization. For example, as can be seen form below snippe; when we provided an example asking for the capital of a country, the model was able to grasp the pattern and modify the predicate to ask about the population. This level of generalization is promising for future fine-tuning and training.

`In:`
```
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

device = "cuda:0" if torch.cuda.is_available() else "cpu"


checkpoint = "tiiuae/falcon-rw-1b"


tokenizer = AutoTokenizer.from_pretrained(checkpoint, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(checkpoint, trust_remote_code=True).to(device)


text = """
input= What is the capital of Japan?
Output= PREFIX dbo: <http://dbpedia.org/ontology/>
PREFIX dbr: <http://dbpedia.org/resource/>
SELECT ?capital 
WHERE {
  dbr:Japan dbo:capital ?capital .
}


input= What is the population of Japan?

"""

input_ids = tokenizer(text, return_tensors="pt").input_ids.to(device)
generated_ids = model.generate(input_ids, max_length=128)
print(tokenizer.decode(generated_ids[0], skip_special_tokens=True))

```

`Out:`
```
SELECT?population WHERE {dbr:population dbo:population.}

input= Which of the two cities is located in Japan?
Output= PREFIX dbo: <http://dbpedia.org/ontology/>
PREFIX dbr: <http://dbpedia.org/resource/>
SELECT?city WHERE {dbr:city dbo:capital?city }

input= What is the capital of Germany?
Output= 
input= Which city is located in Germany?
Output= dbr:Germany dbo:capital?capital

```

More examples of our week 3 experiments can be found in the [dedicated repo directory]


## Next week plan

As we move into the fourth week, we have identified a set of key tasks to further our project.

- **Prompt Formatting:** The first objective is to refine the prompt formatting. We plan to place the task description at the top and correctly demarcate the start of input/output, making it clearer for the models to understand the context. Additionally, we aim to incorporate prefixes or schema information to assist the models in generating more precise SPARQL queries over DBpedia.

- **Few-shot Query Experimentation:** The second goal involves expanding our few-shot query experiments across varying domains and purposes. By extending the variety of few-shot prompts, we aim to explore the models' capabilities to generalize across different contexts.

- **Benchmarking with QALD data:** Lastly, we plan to build a small benchmark using [QALD] (Question Answering over Linked Data) dataset. By assessing the models' performances on this dataset, we will have a clearer picture of their relative effectiveness.

----
[CodeGen-350M]: https://huggingface.co/Salesforce/codegen-350M-multi
[Falcon-RW-1B]: https://huggingface.co/tiiuae/falcon-rw-1b 
[GPT-Neo-2.7B]: https://huggingface.co/EleutherAI/gpt-neo-2.7B 
[dedicated repo directory]: https://github.com/dbpedia/neural-qa/tree/gsoc-mehrzad/gsoc/mehrzad
[QALD]: https://github.com/KGQA/QALD_9_plus