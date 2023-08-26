---
layout: default
title: Week 5- Gearing Up for Fine-tuning
---

# Week 5 - Gearing Up for Fine-tuning ⚙️

---

## Overview
The fifth week began by experimenting with an intruct-tuned model, [codegen25-7b-instruct]. Our proposition is that an instruct-tuned model, which is specifically designed to follow detailed instructions in the prompt, will be ideal for our purpose of translating natural language questions over DBpedia into SPARQL queries.

Alongside this exploration, we also initiated the groundwork for the prompt formatting module. This module is envisioned to be an instrumental component for creating our first experimental prompt-based fine-tuning dataset.



## Findings and challenges 

Week 5 unveiled new insights while also presenting new challenges! 

**Controlling Sequence Length:** Our experiments in the previous week with StarCoder variants showed that adding `<eos>` (End of Sentence) tokens effectively controlled the length of generated tokens. However, this week, with our new model, it necessitated a different approach. We experimented with utilizing the `<eom>` token (End of Messege) along with a post-processing step to discard tokens generated after encountering the `<eom>` token. 

**Limitations of Base Model:** 
Although the model shows intresting generalization capabilities (see below example), it also shows some biased behavior. 

`[In]:`
```
prompt = """# write a SPARQL query with following prefixes:
# PREFIX dbr: <http://dbpedia.org/resource/>
# PREFIX dbo: <http://dbpedia.org/ontology/>

# what is the capital of Japan?
SELECT ?capital WHERE {dbr:Japan dbo:capital ?capital .}


# people born in the capital of Italy before 2000?
"""

inputs = tokenizer.encode_plus(prompt, return_tensors="pt").to(device)
outputs = model.generate(inputs['input_ids'], max_new_tokens=100, attention_mask=inputs['attention_mask'])
print(tokenizer.decode(outputs[0]))
```
`[Out]:`
```
# write a SPARQL query with following prefixes:
# PREFIX dbr: <http://dbpedia.org/resource/>
# PREFIX dbo: <http://dbpedia.org/ontology/>

# what is the capital of Japan?
SELECT ?capital WHERE {dbr:Japan dbo:capital ?capital .}


# people born in the capital of Italy before 2000?
SELECT ?person WHERE {
  ?person dbo:birthPlace dbr:Italy .
  ?person dbo:birthDate ?birthDate .
  FILTER(YEAR(?birthDate) < 2000)
}


#
```

-  A noticeable biased behavior from our model was its tendency to generate SPARQL queries in a specific format, where the statements were presented line by line (not respecting the formatting of the prompt!). While this differentiation doesn't affect the outcome (since DBpedia's SPARQL interpreter treats both formats as equivalent), it hinted at a critical insight: It seems that the model is heavily biased towards the examples it encountered during the pre-training phase, suggesting the limited variety of pre-training data.
- The selected model for this week's experiments was instruct-tuned using [codegen25-7b-mono] as its base. This means that it hasn't been exposed extensively to SPARQL examples. As mentioned in previous posts, to remedy this, we are considering further training using the SPARQL data from the Stack dataset provided by BigCode, plus our own DBpedia-centered dataset. 


## Next week plan
As we transition into the next week, our primary focus is set on the construction of our prompt tuning dataset. Drawing inspiration from our earlier findings and building on our existing work, we plan to:

**Prompt Formatting Implementation:** Dive into the development and implementation of the necessary modules that will allow us to curate and refine our dataset tailored for *DBpedia prompt tuning*.

**Leveraging NsPM Data:** Recognizing the potential of the [NsPM] inititive, we aim to harness its data for our use-case. Our objective is to transform this dataset, adapting it to be coherent with our project goals, thereby ensuring our model is exposed to a wide variety of relevant SPARQL-related examples.


----
[codegen25-7b-instruct]: https://huggingface.co/Salesforce/codegen25-7b-instruct
[codegen25-7b-mono]: https://huggingface.co/Salesforce/codegen25-7b-mono
[NsPM]: https://github.com/dbpedia/neural-qa