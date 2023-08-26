---
layout: default
title: Week 6- Data preperation
---

# Week 6 - Data preperation 📄

---

## Overview
The sixth week of our quest for fine-tuning code LLMs for DBpedia Question-Answering was dedicated to the process of constructing our prompt tuning dataset. After careful considerations, we decided to start fine-tuning with the smallest version of StarCoder, i.e., [starcoderbase-1b], as our based Code LLM. The main activities of the week involved: 
- Development of specific modules to extract and refine data from the [NsPM] dataset. 
- Aligning dataset creation modules from the [official fine-tuning script] with the requirements of our custom dataset.



## Findings and challenges 

**Data Extraction & Refinement:** The week began by the process of extracting data from NsPM dataset which is a rich source, with an approximate size of 7.7 milion pairs of natural language questions and SPARQL queries. The dataset's existing structure required a series of transformations to make it fit the format by which our base model has been pre-trained. Examples of refined data can be seen below:

`[In]:`
```
for index, row in df.sample(n=5).iterrows():
    print(f"Row {index}:\nNL={row['nl_question']}\nSPARQL={row['sparql_query']}\n")
```
`[Out]:`
```
Row 235481:
NL=What is the rebuilding year of tai wai station ?
SPARQL=select ?x where{dbr:Tai_Wai_station dbo:rebuildingYear ?x }

Row 154910:
NL=What was the significant project of loudoun house's architect ?
SPARQL=select ?x where{dbr:Loudoun_House dbo:architect ?x1 . ?x1 dbo:significantProject ?x }

Row 93671:
NL=Which route did aqua line have ?
SPARQL=select count(*) as ?x where{dbr:Aqua_Line_(Pune_Metro) dbo:routeStart ?x }

Row 62059:
NL=What are the suites in the westin grand ?
SPARQL=select count(*) as ?x where{dbr:The_Westin_Grand,_Vancouver dbo:numberOfSuites ?x }

Row 38691:
NL=What is the installed capacity of greifswald power station ?
SPARQL=select count(*) as ?x where{dbr:Greifswald_Power_Station dbo:installedCapacity ?x }
```

**Data Size:** 
As we move into the fine-tuning phase, it's becoming clear that computational resources will play a pivotal role. Given the huge size of our dataset, we had to reconsider the size of the data. We decided to strat by 100k samples that are randomly extracted from the original dataset. This is to ensure handling of the intensive operations ahead! 


**Prompt Composition & Data Format:** We invested time in devising an effective way to formulate prompts for the fine-tuning process. This involved not just questions and SPARQL queries, but also additional information that could help the model generate more accurate and contextually relevant responses. Regarding the format of the dataset, we followed the default data column names, i.e., "prompt" and "completion". Examples of our finalized prompt-tuning dataset can be seen below:

`[In]:`
```
for index, row in df.sample(n=5).iterrows():
    print(f"Prompt:\n{row['prompt']}\n\nSPARQL:\n{row['completion']}\n{'='*80}")
```
`[Out]:`
```
Prompt:
# write a SPARQL query for:
# Did anterior cruciate ligament injury have eMedicine topic ?

# use following prefixes:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SPARQL:
ask where{dbr:Anterior_cruciate_ligament_injury dbo:eMedicineTopic ?x }
================================================================================
Prompt:
# write a SPARQL query for:
# What are the number of binomial authority epicrocis imitans has ?

# use following prefixes:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SPARQL:
select count(*) as ?x where{dbr:Epicrocis_imitans dbo:binomialAuthority ?x }
================================================================================
Prompt:
# write a SPARQL query for:
# How many destinations did cochise airlines have ?

# use following prefixes:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SPARQL:
select count(*) as ?x where{dbr:Cochise_Airlines dbo:destination ?x }
================================================================================
Prompt:
# write a SPARQL query for:
# What is the media type of the night in lisbon ?

# use following prefixes:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SPARQL:
select ?x where{dbr:The_Night_in_Lisbon dbo:mediaType ?x }
================================================================================
Prompt:
# write a SPARQL query for:
# What is the birth name of the pike's stockade's architect ?

# use following prefixes:
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>

SPARQL:
select ?x where{dbr:Pike's_Stockade dbo:architect ?x1 . ?x1 dbo:birthName ?x }
================================================================================
```
> The modified version of the [official fine-tuning script] can be found at the [GSoC project repo].


## Next week plan

For the week ahead, the focus will be on initial fine-tuning: We plan to commence the actual fine-tuning process of our selected model, i.e., [starcoderbase-1b].

----
[starcoderbase-1b]: https://huggingface.co/bigcode/starcoderbase-1b
[official fine-tuning script]: https://github.com/bigcode-project/starcoder/blob/main/finetune/finetune.py
[GSoC project repo]: https://github.com/dbpedia/neural-qa/tree/gsoc-mehrzad/gsoc/mehrzad