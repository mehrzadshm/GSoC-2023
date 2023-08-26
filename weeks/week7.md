---
layout: default
title: Week 7- The Long Road to Fine-Tuning! A Midway Update
---

# Week 7 - The Long Road to Fine-Tuning: A Midway Update

---

## Overview

In the seventh week of this GSoC journey, we ventured deeper into the actual fine-tuning process. We made neccessary modifications to StarCoder's [official fine-tuning code] to tailor it for our developed dataset (see [week 6]). As the process unfolded, we were reminded of the computational hurdles that come along with large-scale machine learning, experiencing firsthand the challenge of resource limitations.


## Findings and challenges 
A key significant accomplishment of the week was adapting the official code to our needs, an important step that enabled us to initiate the fine-tuning process for StarCoder-1B LLM. 

> The modified version of the [official fine-tuning code] can be found at the [GSoC project repo].

Our biggest challenge, which we did  ahead of time, was hardware constraints. Almost a day into the training, we found ourselves at just 20% completion. Our calculations indicate that it could take nearly five days to fine-tune the model using our selected dataset of 100K samples from NsPM. Despite the slow training, we did save checkpoints at 1000 and 2000 steps, which will be instrumental for our evaluation endeavors next week.

<!-- ![Alt text](tuning-week7.png "a title") -->

<img src="tuning-week7.png"  width="900">

## Next week plan

For the week ahead, the focus will be on: 

- **Checkpoint Evaluation:** We aim to complete the development of our evaluation code module and put the saved checkpoints under tests to determine how much improvement is acheived (compared to the base model).

- **Time-Efficiency Strategies:** Given the considerable time investment required for training, we are exploring methods to expedite the process. This includes both software-level optimizations and potential hardware upgrades.

- **Alternative Model Training:** We also plan to examine the possibility of fine-tuning the CodeGen-350M model, which is approximately three times smaller than our current model. This could provide us with a more time-efficient alternative for our experiments.
----
[week 6]: https://mehrzadshm.github.io/GSoC-2023-blog/weeks/week6.html
[official fine-tuning code]: https://github.com/bigcode-project/starcoder/blob/main/finetune/finetune.py
[GSoC project repo]: https://github.com/dbpedia/neural-qa/tree/gsoc-mehrzad/gsoc/mehrzad