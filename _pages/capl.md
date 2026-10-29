---
layout: article
title: "CAPL: Context-Aware Preference Rules for Steering a Frozen LLM"
description: "A small model writes context-scoped preference rules that steer a frozen 9B generator; in a synthetic pilot, CAPL beats static profiles but retrieval scores highest."
permalink: /writing/capl/
date: 2026-10-23
---
# CAPL: Context-Aware Preference Rules for Steering a Frozen LLM

## Abstract

Preference agents personalize a large language model without changing its weights: a smaller model reads a user's history and writes natural-language rules that are passed to the larger model in its prompt [1]. A rule such as "keep answers concise" does not say *when* it applies, yet the same person may want a terse debugging answer and a detailed executive briefing. This study presents CAPL (Context-Aware Preference Learning), an implementation in which each inferred rule carries an explicit context condition, supporting history identifiers, and a conflict flag. A prompted Qwen3.5-2B model extracts the rules and a frozen Qwen3.5-9B model generates the response, both in 4-bit NF4 on a single NVIDIA L4, with no weight updates. In a live pilot with ten synthetic users, 20 evaluation queries, and five prompt-construction methods (100 generations), CAPL reaches 71.25% mean automated style-rule satisfaction, compared with 48.75% for no personalization, 58.33% for a static profile, and 56.67% for task-only rules. TF-IDF retrieval of past examples scores highest at 88.33%. The gains over profiles concentrate in executive and casual queries; on technical queries CAPL is no better than a static profile. These are descriptive results on synthetic data with deterministic style checks, and they motivate a closer look at what information survives the step from history to rules.

## 1. Introduction

Personalizing a large model usually means fine-tuning it per user, retrieving that user's past interactions into the prompt, or summarizing them into a profile. Shashidhar et al. [1] propose a fourth option: train a small *preference agent* to write natural-language rules that steer a frozen large model. Personalization becomes an explicit, inspectable text object rather than a set of weights or a pile of retrieved examples.

A rule on its own, however, is unscoped. Preferences are often situational: the same user may want concise bullets and a short code sample when debugging, and broader context with numbered recommendations when preparing a briefing. A single unscoped rule must either average these preferences or let one overwrite the other.

This study asks whether making the scope explicit helps. CAPL attaches to every rule the context in which it applies, the history entries that support it, and whether opposing evidence was seen. Rules are extracted by prompting a small model, so neither model is trained. The contributions are:

- A structured, context-scoped rule representation for preference agents, with evidence and conflict fields that make each rule traceable to history.
- A two-model pipeline (context detection, prompted rule extraction, frozen generation) that runs in 4-bit precision on one 24 GB GPU.
- A live comparison of five prompt-construction methods on synthetic users with known style preferences, reported with context-level and matched per-query breakdowns.

This is an exploratory implementation study. It does not train a preference agent, run a human evaluation, or claim a general advantage over retrieval.

## 2. Related Work

**Preference agents.** Shashidhar et al. [1] train a small model to generate natural-language preference rules from the difference between a large model's zero-shot output and the user's actual output, conditioned on user intent and task metadata, and pass these rules to a frozen large model. CAPL keeps this split between a small rule writer and a frozen generator, but obtains rules through prompting rather than training and makes applicability, evidence, and conflict explicit fields of the rule.

**Retrieval-augmented personalization.** The LaMP benchmark [2] personalizes outputs by retrieving items from a user's profile into the prompt. Retrieval is the strongest baseline in this study, and its success is a central part of the results.

**Profiles and memory.** Generative Agents [3] and MemGPT [4] maintain summarized memory about a user or environment across interactions. A static profile baseline in this study plays a similar role: one global summary of preferences applied to every query.

## 3. Method

### 3.1 Rule representation

Each rule is a JSON object with five fields: the preference itself, the aspect of the response it targets, the context in which it applies, identifiers of the supporting history entries, and a flag for conflicting evidence. An illustrative example:

```json
{
  "preference": "Use concise bullet points for technical debugging.",
  "target_aspect": "structure",
  "context_condition": "technical_engineering",
  "supporting_evidence_ids": ["interaction_07"],
  "conflict_detected": true
}
```

The context condition gives the rule a boundary, the evidence identifiers let it be traced back to history, and the conflict flag gives the extractor a place to record opposing patterns. When a response uses the wrong style, the rules that produced it can be inspected directly.

### 3.2 Pipeline

The pipeline has three stages.

**Context identification.** A keyword classifier assigns the query to a domain such as technical engineering, executive briefing, or casual chat.

**Rule extraction.** Qwen3.5-2B receives the query, the detected context, and the user's recent history, and is prompted to return applicable rules, note conflicting evidence, and weight recent behavior more heavily. The history view contains the last eight entries, with each historical response truncated to 220 characters. If extraction fails or the output cannot be parsed, an algorithmic fallback derives style features from history using explicit exponential recency weights. The two paths therefore implement recency differently: by instruction in the model path and by weighting in the fallback.

**Generation.** Parsed rules are rendered as steering instructions and placed in the prompt of a frozen Qwen3.5-9B model together with the current query.

### 3.3 Implementation

Both models run with 4-bit NF4 quantization on one NVIDIA L4 (24 GB). The preference agent samples at temperature 0.1 and the generator decodes greedily. Extraction results are cached, and no weights are updated at any point.

## 4. Experimental Setup

### 4.1 Methods compared

Five methods differ only in what personalization input reaches the generator's prompt:

| ID | Method | Personalization input |
|---|---|---|
| B0 | No personalization | Current query only |
| B1 | Static profile | Globally applicable inferred rules |
| B2 | Retrieval | Up to three similar past interactions |
| B3 | Task-only extraction | Rules inferred for the current task, without a context condition |
| B4 | CAPL | Rules scoped to the detected context |

Retrieval ranks past interactions by TF-IDF cosine similarity over historical queries and context labels, and inserts the top three as demonstrations under an approximate word-based budget.

### 4.2 Data

Ten synthetic users each have 20 historical interactions and two evaluation queries. Personas are built from templates with contrasting styles across contexts, so the intended preferences are known. With five methods per query, the pilot produced 100 generations.

### 4.3 Metric

A deterministic scorer checks the aspects each persona specifies: word-count limits, bullet or numbered formatting, required salutation phrases, and the presence of code. A response's score is the fraction of applicable checks it passes, and a method's score is the mean over its 20 queries. Means and matched comparisons were recomputed from the complete 100-record results file.

## 5. Results

### 5.1 Main results

![Mean automated rule satisfaction for all five methods: no personalization 48.75%, static profile 58.33%, retrieval 88.33%, task-only rules 56.67%, and CAPL 71.25%.](/assets/img/writing/capl/live-rule-satisfaction.svg)

*Figure 1. Mean automated style-rule satisfaction on the live pilot (20 queries per method).*

| Method | Mean rule satisfaction |
|---|---:|
| B0 No personalization | 48.75% |
| B1 Static profile | 58.33% |
| B2 TF-IDF retrieval | **88.33%** |
| B3 Task-only rules | 56.67% |
| B4 CAPL | 71.25% |

CAPL scores 12.92 percentage points above the static profile and 14.58 points above task-only extraction, the two rule-based methods without a context condition. Retrieval scores 17.08 points above CAPL.

### 5.2 Results by context

![Automated rule satisfaction by method and context, with ten technical queries and five executive and casual queries per method.](/assets/img/writing/capl/context-breakdown.svg)

*Figure 2. Context-level means. Subgroups contain ten technical and five executive and casual queries per method.*

| Context | Queries per method | Static profile | Retrieval | CAPL |
|---|---:|---:|---:|---:|
| Technical engineering | 10 | 46.7% | 76.7% | 47.5% |
| Executive briefing | 5 | 80.0% | 100.0% | 90.0% |
| Casual chat | 5 | 60.0% | 100.0% | 100.0% |

CAPL's advantage over the static profile comes from the executive and casual subsets. On technical queries the two are nearly equal, and task-only extraction (58.3%) scores higher than CAPL.

### 5.3 Matched comparisons

![Matched comparisons of CAPL's automated score against each other method.](/assets/img/writing/capl/paired-comparisons.svg)

*Figure 3. Number of the 20 matched queries on which CAPL scores higher, equal, or lower than each other method.*

| CAPL compared with | Higher | Equal | Lower |
|---|---:|---:|---:|
| No personalization | 11 | 5 | 4 |
| Static profile | 8 | 8 | 4 |
| TF-IDF retrieval | 1 | 9 | 10 |
| Task-only rules | 9 | 7 | 4 |

Ties are common. Against the static profile, CAPL is higher on eight queries, equal on eight, and lower on four. These comparisons use the same deterministic scores as the means.

### 5.4 Cost

| Method | Mean latency (s) | Mean estimated prompt + output tokens |
|---|---:|---:|
| No personalization | 51.01 | 429.60 |
| Static profile | 51.67 | 477.40 |
| TF-IDF retrieval | 17.45 | 429.15 |
| Task-only rules | 47.69 | 475.10 |
| CAPL | 51.04 | 463.60 |

Peak PyTorch allocation across methods was about 7,503 MiB. Latency includes extraction or prompt preparation and generation, and is affected by lazy model loading and caching in this run. Token counts are word-based estimates for the generator only and exclude preference-agent tokens, so they are not matched end-to-end budgets.

## 6. Discussion

**Scoping helps rules, but rules lose information.** Among the three rule-based methods, the one with a context condition scores highest, which is consistent with the motivation for scoping. Retrieval, however, beats all of them. Showing the generator past responses directly preserves details such as exact salutations, lengths, and formatting that a short extracted rule can drop or paraphrase. The question this pilot raises is which preference information survives extraction, and which is better passed as examples.

**Technical queries are the weak spot.** CAPL matches the static profile on technical engineering, the largest subset. Technical preferences in these personas combine several checks (length, structure, code), and a compressed rule set may capture only some of them.

**Inspectability has value beyond the score.** Because each rule names its context and evidence, a wrong response can be traced to a specific rule and history entry. Retrieval does not offer that directly. This property was not measured here.

## 7. Limitations

- **Synthetic users and automated scoring.** The results measure compliance with deterministic style checks on template personas. They do not establish human preference, factual correctness, or task usefulness.
- **Small sample.** Twenty queries per method, with five or ten per context. The means are descriptive, and no significance test is reported.
- **Overlapping queries.** Histories and evaluation queries share a small template pool, and underlying query text is not deduplicated, so the pilot does not test generalization to unseen topics.
- **No ablations.** Context scoping and recency were not isolated in live runs. The separate 50-user mock run and mock ablations use simulated responses and serve only as implementation checks.
- **Limited records.** The saved results omit inferred rules, full responses, retrieved examples, and extraction-path identifiers, which limits diagnosis of individual outcomes.
- **Resume bug.** When resumed, the runner restores saved means but computes pairwise counts only for newly generated responses. All numbers here were recomputed from the full records; the code has not yet been fixed.

## 8. Conclusion

CAPL gives preference rules an explicit applicability condition and evidence trail, and uses a prompted small model to extract them for a frozen generator on a single GPU. In a synthetic pilot it outperforms unscoped rule methods and no personalization, but retrieval of past examples remains clearly stronger. Natural next steps are live ablations of scoping and recency, logging of the extracted rules, deduplicated and more varied histories, a hybrid that passes both rules and retrieved examples, and evaluation with human judgments.

## Code Availability

The code and evaluation records are not yet public.

## References

1. Sumuk Shashidhar, Abhinav Chinta, Vaibhav Sahai, and Dilek Hakkani-Tür. 2024. [Unsupervised Human Preference Learning](https://aclanthology.org/2024.emnlp-main.200/). *Proceedings of EMNLP 2024*. [arXiv:2410.03731](https://arxiv.org/abs/2410.03731).
2. Alireza Salemi, Sheshera Mysore, Michael Bendersky, and Hamed Zamani. 2023. [LaMP: When Large Language Models Meet Personalization](https://arxiv.org/abs/2304.11406). arXiv:2304.11406.
3. Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein. 2023. [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442). *Proceedings of UIST 2023*.
4. Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion Stoica, and Joseph E. Gonzalez. 2023. [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560). arXiv:2310.08560.
