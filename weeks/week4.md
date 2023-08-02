---
layout: default
title: Week 4
---

# Week 4 - In progress ...  

---

## Overview

The fourth week of my GSoC journey started by focusing on refining the *formatting of prompts*. This was an essential step to establish a baseline for translating raw natural language questions into their corresponding SPARQL queries before tweaking the original parameters of the pre-trained models. It can be viewed as inspecting the initial behavior of the candidate models in terms of their initial capabilities in handling the task of `text-to-sparql`. This week, the release of smaller variants of the [StarCoder models by BigCode], sparked our interest. We decided to include the 3B variant in our evaluation process for the fourth week. 


## Findings and challenges 
This week we revisited the challenge we encountered last week, that is, taking control of the length of the generated sequences. The soloution seems to be lied within the proper use of special tokens, in particular, `EOS` (End Of Sequence) token. The preassumption was that the tokenizer *might* automatically add special tokens to the input sequence. However, we observed despite using the `tokenizer.encode_plus` method, EOS token is not presented within the encoded input sequence. The following code blocks refer to our experiemnts with the `bigcode/starcoderbase-3b` checkpoint. 



```
[In]
eos_token = tokenizer.eos_token
eos_token_id = tokenizer.eos_token_id

print(f'End of Sequence token:\t{eos_token}\t with id={eos_token_id}\n')

question = "What is the capital of France?\n"
sparql_query = "SELECT ?capital WHERE { dbr:France dbo:capital ?capital .}"

# check if the tokenizer automatically adds the necessary special tokens and attention mask.
inputs = tokenizer.encode_plus(question + sparql_query, return_tensors="pt")
print(inputs)

if eos_token_id not in inputs['input_ids'][0]:
    print(f'\nNo EOS token found in tokenized input')
```


```
[Out]
End of Sequence token:	<|endoftext|>	 with id=0

{'input_ids': tensor([[ 8197,   438,   322, 18926,   432, 45600,    49,   203,  4620,  1018,
         21147,  4998,   301,   343,   839,    44,  6082,   723, 36855,    44,
         21147,  1018, 21147,   638,   111]]), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
         1]])}

No EOS token found in tokenized input
```

> **Note:**For more details, check out the notebook in the [dedicated repo directory].



```
[In]
prompt_without_eos = """
Compelete "SPARQL" by translating "question" into a SPARQL query usig DBpedia prefixes below:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

"question"= what is the capital of Japan?
"SPARQL"= SELECT ?capital WHERE { dbr:Japan dbo:capital ?capital . }

"question"= what is the capital of USA?
"SPARQL"= SELECT ?capital WHERE { dbr:USA dbo:capital ?capital . }

"question"= what is the capital of Italy?
"""

prompt_with_eos = """
Compelete "SPARQL" by translating "question" into a SPARQL query usig DBpedia prefixes below:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/><|endoftext|>

"question"= what is the capital of Japan?
"SPARQL"= SELECT ?capital WHERE { dbr:Japan dbo:capital ?capital . }<|endoftext|>

"question"= what is the capital of USA?
"SPARQL"= SELECT ?capital WHERE { dbr:USA dbo:capital ?capital . }<|endoftext|>

"question"= what is the capital of Italy?
"""


inputs = tokenizer.encode_plus(prompt_without_eos, return_tensors="pt").to(device)
outputs = model.generate(
    inputs['input_ids'],
    max_new_tokens=50,
    attention_mask=inputs['attention_mask'])
print('Before adding EOS:\n', tokenizer.decode(outputs[0]))

print("="*35)

inputs = tokenizer.encode_plus(prompt_with_eos, return_tensors="pt").to(device)
outputs = model.generate(
    inputs['input_ids'],
    max_new_tokens=100,
    attention_mask=inputs['attention_mask'])
print('After adding EOS:\n', tokenizer.decode(outputs[0]))
```


```
[Out]
Before adding EOS:
 
Compelete "SPARQL" by translating "question" into a SPARQL query usig DBpedia prefixes below:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

"question"= what is the capital of Japan?
"SPARQL"= SELECT?capital WHERE { dbr:Japan dbo:capital?capital. }

"question"= what is the capital of USA?
"SPARQL"= SELECT?capital WHERE { dbr:USA dbo:capital?capital. }

"question"= what is the capital of Italy?
"SPARQL"= SELECT?capital WHERE { dbr:Italy dbo:capital?capital. }

"question"= what is the capital of France?
"SPARQL"= SELECT?capital WHERE { dbr:Fr
===================================
After adding EOS:
 
Compelete "SPARQL" by translating "question" into a SPARQL query usig DBpedia prefixes below:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/><|endoftext|>

"question"= what is the capital of Japan?
"SPARQL"= SELECT?capital WHERE { dbr:Japan dbo:capital?capital. }<|endoftext|>

"question"= what is the capital of USA?
"SPARQL"= SELECT?capital WHERE { dbr:USA dbo:capital?capital. }<|endoftext|>

"question"= what is the capital of Italy?
"SPARQL"= SELECT?capital WHERE { dbr:Italy dbo:capital?capital. }
<|endoftext|>
```



As can be seen, when adding EOS tokens manually, although `max_new_tokens` is set to 100 when generating the second output sequence, the model stops generating tokens as we desired! 

----

Another key observation was that the tokenizer is not recognizing the space at certain positions, e.g., before `"."` and `"?"`.  One possibile explanation is that the tokenizer was trained on a dataset where such spaces were not common and hence it's not preserving them. This theory particularly makes sensne for spaces around `"."` char as it is not common in the majority of programming laguages. Our proposed soloution was to perform pre-processing; adding extra spaces at problematic positions. This way, when the tokenizer processes the text, it will treat the punctuation chars as separate tokens.


```
[In]
# Fixing mis-tokenized spaces around "?" and "." tokens

sparql_query = "SELECT ?capital WHERE { dbr:France dbo:capital ?capital .}"

inputs = tokenizer.encode_plus(sparql_query, return_tensors="pt")
print('Raw query tokenization => ', tokenizer.decode(inputs['input_ids'][0]))

sparql_query = sparql_query.replace("?", " ?").replace(".", " . ")
inputs = tokenizer.encode_plus(sparql_query, return_tensors="pt")
print('Pre-processesd query tokenization => ', tokenizer.decode(inputs['input_ids'][0]))
```

```
[Out]
Raw query tokenization =>  SELECT?capital WHERE { dbr:France dbo:capital?capital.}
Pre-processesd query tokenization =>  SELECT ?capital WHERE { dbr:France dbo:capital ?capital . }
```

The above example shows the importance of taking the required time to deeply understand the limitations of the pre-trained tokenizer for the task at hand.
> **TODO**: SPARQL queries with more complex syntax need to be tested against the tokenizers!

---

Finally, the above examples reveal a critical challenge we were expecting to encounter since the begging of the project: how can we effectively map the named entities with their corresponding DBpedia resources. For example,  as it can be seen above the model failed to map "USA" with the correct DBpedia resource `dbr:United_States`. Addressing this problem is a *priority* for us, as the correct mapping to DBpedia entity URIs are pivotal for the accurate generation of SPARQL queries. We are actively exploring potential solutions for this problem. 

One soloution is to stick to the seq2seq approach by providing the model with a large corpus of pairs of natural language questions and their corresponding SPARQL queries containing DBpedia URIs. However, the quality of the mappings will heavily depend on the *quality* and *quantity* of the training data. Also, it's generally harder to interpret and debug the output, as manipulation of entity mappings within the seq2seq model is not explicit and directly controllable by us.

Alternatively, we can develop an **entity recognition and linking (disambiguation)** module to explicitly deal with entity mappings. A key advantage of this approach is the possibility of adjusting the NER model or entity linking tool separately if they're not performing well, while in an end-to-end approach, we have to adjust the entire model or the training data if the output is not satisfactory.

An effective way to incorporate the results of an explicit entity linking module within our seq2seq flow could be to preprocess the input questions before feeding them to the seq2seq model. Here's how we think this could work:

1. **Entity linking:** First, process each natural language question using the entity linking module to identify entities in the question and replace each entity with its DBpedia URI. For example:

    >`In:  What is the capital of France?`

    >`Out: What is the capital of dbr:France?`

2. **Train Seq2Seq model:** Feeding the processed question into our seq2seq model that has been trained to generate SPARQL queries from questions that have already been processed with the entity linking module. This way, the model doesn't have to learn to perform entity recognition and linking itself; it can focus on generating the rest of the SPARQL query.



----
[StarCoder models by BigCode]: https://huggingface.co/bigcode
