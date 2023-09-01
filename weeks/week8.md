---
layout: default
title: Week 8- Checkpoint Reviews; Streamlining Evaluation & Post-Processing
---

# Week 8- Checkpoint Reviews: Streamlining Evaluation & Post-Processing

---

## Overview
In the eighth week of my GSoC journey, the spotlight was on the model evaluation process and the review of the fine-tuned checkpoints. Our preliminary observations confirmed that a post-processing stage is an indispensable prerequisite for the evaluation process.



## Findings and challenges
Having trained the StarCoder-1B model for over 23 hours (see [week 7]), we managed to save checkpoints at the 1000 and 2000 steps. The initial results are promising, although they've led to some new questions about model behavior. An example of input/output for the last fine-tuned checkpoint is given below.   

`[In]:`
> ```
> prompt = """
> # write a SPARQL query for:
> # What was the population of Italy in 2000 ?
> 
> # use following prefixes:
> PREFIX dbr: <http://dbpedia.org/resource/>
> PREFIX dbo: <http://dbpedia.org/ontology/>
> """
> ```
`[Out]:`

**Before fine-tuning:**
> ```
> # write a SPARQL query for:
> # What was the population of Italy in year 2000?
> 
> # use following prefixes:
> PREFIX dbr: <http://dbpedia.org/resource/>
> PREFIX dbo: <http://dbpedia.org/ontology/>
> PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
> PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
> PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
> PREFIX owl: <http://www.w3.org/2002/07
> ```

**After fine-tuning:**
> ```
> # write a SPARQL query for:
> # What was the population of Italy in 2000?
> 
> # use following prefixes:
> PREFIX dbr: <http://dbpedia.org/resource/>
> PREFIX dbo: <http://dbpedia.org/ontology/>
> 
> Answer: select?x where{dbr:Italy dbo:population?x1.?x1 dbo:year?x }<|endoftext|>
> ```

At first glance, the model appears to have undergone a dramatic transformation! successfully emulating the overall structure of our anticipated responses. As it can be seen above, before fine-tuning, the model solely generates a set of prefixes. However, a more nuanced examination reveals some unexpected behaviors. 

The model inconsistently prefixes main SPARQL tokens with the phrase `Answer:`, even though such formatting is totally absent from our fine-tuning dataset. Our evaluations based on a test set of 500 prompt samples, confirms that this is a recurring behavior, and not just an isolated occurrence. Additionally, the model neglects to insert spaces between certain tokens—a flaw we had previously observed in other models during our preliminary investigations (refer to [week 4]). Most critically, the model continues to exhibit a significant limitation: it fails to accurately generate data as represented in the DBpedia knowledge graph. As can be seen above, instead of using `dbo:populationTotal` the model makes up an ontology entity as `dbo:population`. This is also an issue we had foreseen and discussed in [week 4].




## Next week plan

Having our evaluation script polished this week, for the week ahead we will focus on: 

- **Post-Processing Improvements:** This is essential for performing a realistic evaluation based on the Bleu metric. Specifically, we aim to remove all redundant tokens ( (e.g. <|endoftext|>)) and adjust spacing issues within the generated queries.

- **Checkpoint Evaluation:** Complete the evaluation for the last saved checkpoint, after post-processed generated sequences.

- **Fine-tuning Optimization:** We will continue looking into methods for reducing the overall training time while looking for possible hardware upgrades.

  
----
[week 7]: https://mehrzadshm.github.io/GSoC-2023-blog/weeks/week7.html
[week 4]: https://mehrzadshm.github.io/GSoC-2023-blog/weeks/week4.html
