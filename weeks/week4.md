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


`In:`
```
eos_token = tokenizer.eos_token
eos_token_id = tokenizer.eos_token_id

print(f'End of Sequence token:\t{eos_token}\t with id={eos_token_id}\n')

question = "What is the capital of France?"
sparql_query = "SELECT ?capital WHERE { dbr:France dbo:capital ?capital .}"

# check if the tokenizer automatically adds the necessary special tokens and attention mask.
inputs = tokenizer.encode_plus(question + sparql_query, return_tensors="pt")
inputs
```

`Out:`
```
End of Sequence token:	<|endoftext|>	 with id=0

{'input_ids': tensor([[ 8197,   438,   322, 18926,   432, 45600,    49,  4620,  1018, 21147,
          4998,   301,   343,   839,    44,  6082,   723, 36855,    44, 21147,
          1018, 21147,   638,   111]]), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]])}

```

> For more details, check out the notebook in the [dedicated repo directory].






`In:`
```
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

`Out:`
```
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
[StarCoder models by BigCode]: https://huggingface.co/bigcode
