This is the code for the CoNLL 2019 paper _[Do Massively Pretrained Language Models Make Better Storytellers?](https://www.aclweb.org/anthology/K19-1079.pdf)_.

This repo contains code to download the generated text data, compute the automatic metrics on it, and produce the plots in the paper.

## Download the data

Download and unzip [this file](https://nlp.stanford.edu/data/gpt_analysis/data.zip) (1.1GB) to make a directory called `data` in `story-generation-eval`.

## About the model-generated stories (unannotated)

In `data/stories_unannotated`, each json file contains a dictionary mapping from the sample id (an integer) to a story (which can be generated by Fusion Model/GPT2, or a human sample from WritingPrompts dataset).

Each story is represented by a dictionary containing the following:
- `prompt_text`, `story_text`: string. The text of the prompt/story.
- `prompt_tokens`, `story_tokens`: list of strings. The model-level tokens in the prompt/story. This is word-level for Fusion and subword-level for GPT2.
- (Not for human stories) `gen_probs_orig`: list of floats, same length as story_tokens. The model's probability for each of the generated tokens. 
- (Not for human stories) `vocab_entropy_orig`: list of floats, same length as story_tokens. The entropy of the model's vocabulary distribution for each step.

If you want to generate the unannotated story files yourself:
- The code we used to generate text from the Fusion Model can be found in [fairseq](https://github.com/pytorch/fairseq/tree/master/examples/stories).
- The code we used to finetune and generate text from GPT2-117 can be found in Huggingface [transformers](https://github.com/huggingface/transformers).

## About the teacher-forcing data

In `data/teacherforcing` are two files, one for the Fusion Model and one for GPT2-117.
These are the result of running the models on the WritingPrompts test data using teacher forcing (i.e. not generating) in order to measure the per-step probabilities and vocabulary entropies. 

## About the automatic annotations

In `data/stories_metric_annotated`, each pickle file corresponds to a file in `data/stories_unannotated`. It contains a dictionary mapping from the sample id (an integer) to an annotated story.

Each annotated story is represented by a `Sample` (see `samples.py` for the class) which contains the information from `stories_unannotated` file, plus the metric annotations.

## Running the annotation code yourself

If you want to run the annotation code yourself, here is a guide to explain how `data/stories_metric_annotated` was produced:

### Step 1: Spacy annotations
Several of our automatic metrics use Spacy annotations, so we precompute those before computing our automatic metrics. Run 
```
python spacy_annotate.py
```
This will take each story in `data/stories_unannotated` and save a Spacy encoding of its prompt and story in `data/stories_spacy_annotated`.

The Spacy annotation can take a long time, so if you want to download the precomputed ones for our generated stories, you can do so [here]() (XGB). Unzip it into the `data` dir to create a new directory called `data/stories_spacy_annotated`.

### Step 2: Other prerequisites

Note that our annotation script additionally uses these three files:
- `data/arora_sentence_embedder.pkl`: This is needed to compute the Arora sentence embeddings for the story-prompt similarity metric. To see how it was produced, follow the instructions in `make_arora_sent_embedder.py`.
- `data/unigram_probdist.json`: This is needed to compute the mean log unigram probability metric. It was computed by iterating through the WritingPrompts training set and counting the frequency of each unigram.
- `data/concreteness.csv`: This is needed to compute the mean noun and verb concreteness metrics. It is from the paper _[Concreteness ratings for 40 thousand generally known English word lemmas](https://link.springer.com/article/10.3758/s13428-013-0403-5)_.

### Step 3: Run the annotation script

Run 
```
python metrics_annotate.py
```
This will take each story in `data/stories_unannotated`, load its Spacy encoding in `data/stories_spacy_annotated`, and save a metric annotated version in `data/stories_metric_annotated`.

## Analyzing the metric annotations

Run 
```
jupyter notebook
``` 
to open `analyze.ipynb`. This notebook loads the data in `data/stories_metric_annotated` and `data/teacherforcing`, and produces the plots in the paper. The notebook also allows you to browse the generated stories.

## Citation

If you use this code, please cite our paper:
```
@inproceedings{see2019massively,
  title={Do Massively Pretrained Language Models Make Better Storytellers?},
  author={See, Abigail and Pappu, Aneesh and Saxena, Rohun and Yerukola, Akhila and Manning, Christopher D},
  booktitle={Proceedings of the 23rd Conference on Computational Natural Language Learning (CoNLL)},
  pages={843--861},
  year={2019}
}
```
