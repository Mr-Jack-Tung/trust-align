# Measuring and Enhancing Trustworthiness of LLMs in RAG through Grounded Attributions and Learning to Refuse

> 📣 We are releasing TRUST-SCORE, a holistic evaluation of
the trustworthiness of LLMs in a RAG framework, and TRUST-ALIGN framework that aligns LLMs for higher TRUST-SCORE. 
[Paper]() | [Website]()

We are excited to announce the release of TRUST-SCORE evaluation datasets and TRUST-ALIGN alignment datasets:

1) **[TRUST-SCORE]()**: It features calibrated questions and refusals to measure the model's trustworthiness.

2) **[TRUST-ALGIN]()**: Ehance the model's trustworthiness with high-quality synthesized cited responses.


## Overview

LLMs are an integral part of retrieval-augmented generation (RAG) systems. While many studies focus on evaluating the quality of end-to-end RAG systems, there is a lack of research on understanding the appropriateness of an LLM for the RAG task. Thus, we introduce a new metric, \metric{}, that provides a holistic evaluation of the trustworthiness of LLMs in a RAG framework. We show that various prompting methods, such as in-context learning, fail to adapt LLMs effectively to the RAG task. Thus, we propose \method{}, a framework to align LLMs for higher \metric{}. LLaMA-3-8b, aligned with our method, significantly outperforms open-source LLMs of comparable sizes on ASQA ($\uparrow$ 10.7) QAMPARI ($\uparrow$ 29.2) ELI5 ($\uparrow$ 14.9), respectively. 


## Quick Links

  - [Requirements](#requirements)
  - [Data](#data)
  - [TRUST-SCORE-Eval](#trust-score-eval)
  - [TRUST-ALGIN](#trust-algin)
  - [Bug or Questions](#bug-or-questions)
  - [Citation](#citation)


## Requirements

```bash
conda env create -f environment.yaml
conda activate cite
```

<!-- ## Code Structure

* `data_processing/`: TRUST_Align preference data generation pipeline
* `heuristic_preference`: alignment response generation.
* `run.py`: run file to reproduce our baseline generations.
* `eval.py`: eval file to evaluate generations.
* `prompts`: folder that contains all prompt files.
* `configs/`: folder that contains all config files to reproduce baselines.
* `tools/`: misc code (generate summaries/snippets, reranking, etc.) -->

## Data

TRUST-SCORE evaluation dataset is available [here]() and also on [Huggingface]().

TRUST-ALGIN training dataset is also available [here]() and also on [Huggingface]().

Our data included top-100 DPR/GTR retrieved results for ASQA, QAMPARI and ExpertQA, and top-100 BM25 retrieved results for ELI5. We also provide reranked oracle retrieval results, where top-5 passages can achieve the same recall as the original top-100 recall.


## Trust-Score

**Trust-Score** is a more reliable and comprehensive measure of an LLM's capabilities for RAG, covering the following aspects: Does the LLM correctly identify answerable questions? Are the responses grounded in the provided documents, i.e., do the citations support the claims in the ground-truth response? And are the citations relevant?

<img src="assets/trust_score.png" alt="TRUST-SCORE" width="100%">


### Eval Data Preparation

We support three types of dataset format: EM (Exact Match, like ASQA type), EM@5 (top-5 EM, like QAMPARI type), or CM (Claim Match, like ELI5 type).

Your evaluation dataset should satistfy the following format:

The file contains a list of JSON dictionaries with the following fields:

- `question` - The question being asked.
  
  Example:
  ```json
  "question": "Who has the highest goals in world football?"
  ```

- `answers`- A list of all gold answers, where each element is an array containing different variations of each gold answer. The gold answers can either be in short form or full sentence.
  
  Examples: 
  ```json
  "answers": [
    ["Daei", "Ali Daei"],
    ["Bican", "Josef Bican"],
    ["Sinclair", "Christine Sinclair"]
  ]
  ```
  or 

  ```json
  "answers": [
    [
        "Firms like Snapchat and Uber need to establish their brand and amass users before introducing ads."
    ],
    [
        "Introducing ads too early can deter potential users."
    ],
    [
        "Uber is reinvesting a lot of money to make their service better."
    ]
  ]
  ```


- `docs` - A list of dictionaries providing evidence from documents related to the question. Each dictionary contains the following fields:
  
  - `title` - The title of the document. 
  
  Example: 
  ``` json
  "title": "Argentina–Brazil football rivalry"
  ```
  
  - `text` - A snippet from the document containing relevant information. 
  
  Example: 
  ```json
  "text": "Pelé's 1281 goals are recognized by FIFA as the highest total achieved by a professional footballer, although the Soccer Statistic Foundation (rssf) recognizes only 767 goals in official mode, occupying the third place after Josef Bican (805) and Romario (772)."
  ```
  
  - `answers_found` - A list of integers, where each element corresponds to whether the answer was found in the document (0 if not found, 1 if found).

  Example:
  ``` json
  "answers_found": [
      0,
      0,
      0
  ]
  ```
  
  - `rec_score` - A recall score indicates the percentage of answers entailed by the document.
  
  Example:

  ``` json
  "rec_score": 0.0
  ```

### Evaluation Pipeline

You can easily evaluate your model based on the formated evaluation dataset by the following code:

``` python
from utils import RUN_Config
from trust_eval import TRUST_SCORE

config = RUN_Config()

# Assume you load this from a JSON or YAML file
example_config = {
  "prompt_file": "prompts/asqa_rejection.json",
  "eval_file": "data/asqa_eval_top100_calibrated.json",
  "output_dir": "save",
  "eval_type": "em",
  "model": "meta-llama/Llama-2-7b-chat-hf",
  "max_length": 4096,
  "temperature": 0.5,
  "top_p": 0.95,
  "vllm": True,
  "no_demo": True,
}

# Update config with new values
config.update_from_dict(example_config)

score = TRUST_SCORE(config)

print(score)
```


## TRUST-ALGIN

<img src="assets/trust_align.png" alt="TRUST-SCORE" width="100%">

### Preparation

Please first refer to [Retrieval](https://github.com/princeton-nlp/ALCE/tree/main?tab=readme-ov-file#retrieval) in the ALCE benchmark to download the required document corpus (GTR-based wikipedia snapshot and BM25-based Sphere)

Download the ASQA, QAMPARI, ELI5, and ExpertQA datasets accordingly.

### Seed Sample Curation

You can reproduce the seed sample curation step with the following command:

```bash
cd TRUST_ALIGN/seed_samples
sh cluster.sh
sh re_cali.sh
```
In `re_cali.sh`, remember to specify `BM25_SPHERE_PATH`, `DPR_WIKI_TSV`, and `GTR_EMB` to the paths where you stored each corpus, respectivel.

Output is the `{dataset}_doc.json` in `data` folder.

The choice of `dataset` could be either `asqa`, `qampari`, `eli5`, or `expertqa`.

### Augment Sample Curation

You can reproduce the augment sample curation step (document recombination) with the following command:

```bash
cd TRUST_ALIGN/augment_samples
sh doc_recombination.sh {dataset}
```

Output is the `{dataset}_doc_augment.json` format in `data\` folder.


### Positive Response Generation

You can create natural responses by running the following code:

``` bash
cd TRUST_ALIGN/positives_synthesis
sh gen_ans.sh
```

In `en_ans.sh`, please specify the `--data_file` with the path to your dataset.

To get postive resposne with citations, run the following code:

``` bash
python gen_positives --input_folder {dataset_folder}
```

`{dataset_folder}` is the path to your saved datasets folder.


### Negative Response Selection

You first need to obtain the model's output for curated samples as follows:

``` bash
cd TRUST_ALIGN/negatives_selection
sh infer.sh
```

In `infer.sh`, you need to specify `INFER_FILE` and `OUTPUT_DIR` to the path you saved samples and the path you want to save obtained output, respectively. You can also change the `--config` inside for other datasets.

Based on obtained model's output, you can calculate $e_i$ for each sample. Outputs the $e_i$ for each ith hallucintion type in `.json` format stored in `data/` folder.

```bash
sh error_selection.sh 
```

In `error_selection.sh`, you also need to specify `BASE_DIR` and `OUTPUT_DIR` to the path you saved samples and the path you want to save obtained output, respectively.


## Bug or Questions?

If you have any questions related to the code or the paper, feel free to email Maojia (`maojia_song@mymail.sutd.edu.sg`). If you encounter any problems when using the code, or want to report a bug, you can open an issue. Please try to specify the problem with details so we can help you better and quicker!




## Citation

Please cite our paper if you use Trust-align in your work:

```bibtex

```
