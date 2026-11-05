---
layout: article
title: "Replaying a Published Weight Edit of Qwen3.5-0.8B on MATH-500"
description: "I re-run a publicly released weight edit of Qwen3.5-0.8B on MATH-500: its reported gain reproduces, and the unedited model with thinking turned off reaches similar accuracy with a third of the tokens."
permalink: /writing/weight-repair/
date: 2026-11-05
---
# Replaying a Published Weight Edit of Qwen3.5-0.8B on MATH-500

## Abstract

Takehi et al. [1] report that an LLM, working directly on the raw weights of Qwen3.5-0.8B, found an edit that stops the model's runaway looping and raises its MATH-500 accuracy from 7.9% to 48.1%. I re-ran their released edit under their protocol. The gain reproduces: +44.0 points on 145 paired questions (95% CI +37.4 to +50.7). I also tested a baseline the post does not report: the unedited model with thinking turned off reaches similar accuracy (52.3% vs 54.0%) with about a third of the tokens.

## 1. Introduction

**What the original post reports.** In a blog post [1], Takehi et al. argue that frontier LLMs can read and write neural network weights directly. One of their experiments targets Qwen3.5-0.8B [5]. With thinking enabled, this model often repeats itself until it hits the 32,768-token limit, so it rarely reaches an answer on MATH-500 [3, 4]. Their editing model, given the weights and a budget of 50 forward passes, traced the looping to layers 15 and 19 and changed 131,072 of the model's weights. The post reports MATH-500 accuracy rising from 7.9% to 48.1%, and the share of answers hitting the token limit falling from 93% to 33%. It explains the gain as loop repair: the edit "lets the model finish answers it could already work out". The post also reports gains on GSM8K and IFEval, and the authors released the edit and their logs [2].

**Why I ran this.** A 40-point gain from changing 0.016% of a model's weights, found by another LLM without training, is a striking result, and striking results deserve independent checks. I wanted to answer two questions. First, does the released edit give the same gain when someone outside the original team runs it? Second, since the gain comes from stopping the looping, how does the edit compare with a simpler way of stopping it that needs no weight changes at all, such as turning thinking off? That comparison shows what the edit actually adds, and at what cost in tokens.

**What I did.** I ran experiments and benchmarks with the released edit. I did not search for edits myself; I applied the authors' published edit file to the original weights and evaluated it on MATH-500 with their settings. My contributions are:

- An independent replication of the post's MATH-500 result for the released edit.
- A comparison the post does not include: the unedited model with thinking turned off, which also avoids the looping.

I did not evaluate GSM8K or IFEval, and nothing here tests how the edit was found.

## 2. Setup

**Model and edit.** Qwen3.5-0.8B [5], original weights. I applied the released 50-pass edit [2] with the authors' own script and checked it against their manifest.

**Decoding.** Thinking on, 32,768-token cap, temperature 0.6, top-p 0.95, top-k 20, presence penalty 1.5, four samples per question, with the authors' answer extraction and grading. These match the settings in the original post.

**Questions.** Each configuration was run on a subset of MATH-500 questions: 334 for the base model with thinking on, 145 for the edited model and 220 for the base model with thinking off. Comparisons between configurations use only the questions both completed, with paired bootstrap confidence intervals.

## 3. Results

### 3.1 The edit's gain reproduces

**Table 1.** MATH-500 with thinking on. Numbers in the first column are as reported in the original post [1]; the second column is my run.

| MATH-500 | Original post [1] | My replication |
|---|---|---|
| Base accuracy | 7.9% | 8.5% (334 questions); 10.0% on the 145 paired with edited |
| Edited (50 passes) accuracy | 48.1% | 54.0% (145 questions) |
| Improvement | +40.2 pts | +44.0 pts (95% CI +37.4 to +50.7) |
| Base hit token limit | 93.0% | 92.2% |
| Edited hit token limit | 33.1% | 29.5% |

My base accuracy and looping rate are close to the post's, and the edited gain is the same size.

### 3.2 Turning thinking off matches the edit

As the post describes, the base model's failure is runaway generation: more than nine in ten samples never finish. Turning thinking off is another way to avoid it, without editing any weights. This comparison is mine; it does not appear in the post.

**Table 2.** My runs: edited model against the unedited model with thinking off.

| MATH-500 | Accuracy | Hit token limit | Mean tokens |
|---|---|---|---|
| Base, thinking off (220 questions) | 52.3% | 11.5% | 7.6k |
| Edited, thinking on (145 questions) | 54.0% | 29.5% | 21.0k |

On the 129 questions both configurations completed, the edited model leads by +2.1 points (95% CI -3.7 to +7.9). The interval covers zero, so the difference is not statistically significant, and the edited model spends about three times as many tokens. This is consistent with the post's own explanation that the edit removes looping rather than adding new ability.

## 4. Conclusion

The released edit's MATH-500 gain reproduces at the size the post reports. On the same questions, the unedited model with thinking turned off reaches similar accuracy with about a third of the tokens. Both results fit the post's account of the edit as a loop repair; the thinking-off baseline shows that, on this benchmark, a no-edit setting recovers a similar share of the accuracy.

## 5. Limitations

**Partial benchmark.** My results cover subsets of MATH-500 rather than all 500 questions. I did not evaluate GSM8K, IFEval or other benchmarks reported in the post.

**Numeric precision.** Runs used different GPUs and precisions (bf16 and float16), which can change sampled outputs.

## References

1. Rikiya Takehi, Jayden Teoh, Alex Zhang, and Omar Khattab. 2026. [Sparks of Weight-Space Programming: LLMs can natively read & write neural network weights](https://rikiyat.github.io/blog/weight-space-programming/). Blog post.
2. Rikiya Takehi et al. 2026. [lm_understand-logs](https://github.com/RikiyaT/lm_understand-logs). GitHub repository with released edits and logs.
3. Dan Hendrycks, Collin Burns, Saurav Kadavath, et al. 2021. [Measuring Mathematical Problem Solving With the MATH Dataset](https://arxiv.org/abs/2103.03874). *NeurIPS 2021 Datasets and Benchmarks*.
4. Hunter Lightman, Vineet Kosaraju, Yura Burda, et al. 2023. [Let's Verify Step by Step](https://arxiv.org/abs/2305.20050). arXiv:2305.20050.
5. Qwen Team. 2026. [Qwen3.5-0.8B model card](https://huggingface.co/Qwen/Qwen3.5-0.8B). Hugging Face.
