---
layout: article
title: "Building CAPL: context-aware preference rules for a frozen LLM"
description: "An implementation study inspired by preference agents, exploring how response preferences can be tied to the situations in which they apply."
permalink: /writing/capl/
date: 2026-10-23
---
# Building CAPL: context-aware preference rules for a frozen LLM

I came across [*Unsupervised Human Preference Learning*](https://huggingface.co/papers/2410.03731) on Hugging Face. Its central idea gave me a useful way to think about personalization: a small preference agent can produce natural-language rules that guide a larger model.

That separation interested me. The larger model supplies its general capabilities; the rules describe how its response should be adapted for a particular user and task. Personalization becomes something explicit that can be passed into a prompt.

I wanted to explore the representation of those rules. A preference such as “keep answers concise” leaves a question unanswered: **when should it apply?** Someone might want a short debugging answer with code and a detailed executive briefing with numbered recommendations. Both preferences can belong to the same person.

CAPL (**Context-Aware Unsupervised Human Preference Learning**) is my implementation study of that question. It extracts structured, context-scoped rules from interaction history and uses them to steer a frozen response model. This article covers the design, the implementation, and what an initial live pilot showed.

## The starting point: preference agents

Shashidhar et al.'s [EMNLP 2024 paper](https://aclanthology.org/2024.emnlp-main.200/) trains a smaller preference agent to generate natural-language rules, while leaving the larger response model frozen. The agent conditions on user intent and task metadata.

CAPL uses that preference-agent architecture as its starting point. CAPL uses prompted extraction without training either model, and makes applicability, supporting evidence, and conflict information explicit in a JSON rule schema. Those are implementation choices I wanted to examine in a controlled setting.

| Design choice | CAPL implementation |
|---|---|
| Preference representation | Structured rules with an explicit context condition |
| Personalization mechanism | Rules inserted into the response model's prompt |
| Preference agent | Qwen3.5-2B, used through prompted extraction |
| Response generator | Frozen Qwen3.5-9B |
| Weight updates | None during personalization |
| Initial evaluation | Synthetic histories with known style criteria |

## Giving a rule an applicability condition

Consider two situations:

| Situation | Example preference |
|---|---|
| Technical debugging | Concise bullets, a small code example, direct wording |
| Executive briefing | Broader context, numbered actions, business trade-offs |

The rule representation needs a place to express that distinction. In CAPL, a rule includes a preference, its target aspect, the context in which it applies, and identifiers for supporting history entries.

An illustrative schema example is:

```json
{
  "preference": "Use concise bullet points for technical debugging.",
  "target_aspect": "structure",
  "context_condition": "technical_engineering",
  "supporting_evidence_ids": ["interaction_07"],
  "conflict_detected": true
}
```

The context condition gives the instruction a boundary. Supporting evidence provides a way to trace it back to history, and conflict information gives the extractor a place to record opposing patterns.

This makes the personalization layer inspectable: the rules themselves can be examined when a response uses the wrong style. Whether that representation improves the response is what the evaluation needs to test.

## How CAPL works

The pipeline has three main operations: identify the query context, infer applicable rules from history, and generate a response using those rules.

**Context identification.** The current classifier uses keyword heuristics to assign a query to a domain such as technical engineering or executive briefing.

**Preference extraction.** Qwen3.5-2B receives the query, the detected context, and recent interaction history. Its prompt asks for applicable rules, conflicting evidence, and greater emphasis on recent behavior. The model-based history view contains the last eight entries, with each historical response truncated to 220 characters.

**Response generation.** Parsed rules are rendered into steering instructions for Qwen3.5-9B. The model receives the current query and those instructions, with its weights unchanged.

An algorithmic fallback handles extraction failures or unparseable rules. It derives style features from history and applies explicit exponential recency weights. The model path instead expresses recency through instructions in the extraction prompt. These are separate mechanisms in the implementation.

Both models ran with 4-bit NF4 quantization on an NVIDIA L4 for the live pilot. The preference agent samples at temperature 0.1, while the response generator uses greedy decoding.

## Testing the representation

I compared five ways of constructing the response prompt:

| ID | Method | Personalization input |
|---|---|---|
| B0 | No personalization | Current query |
| B1 | Static profile | Globally applicable inferred preference rules |
| B2 | Retrieval | Up to three similar historical examples |
| B3 | Task-only extraction | Rules inferred for the current task |
| B4 | CAPL | Rules scoped to the detected context |

The retrieval baseline uses TF-IDF cosine similarity over historical queries and context labels. It includes selected past interactions as demonstrations, with an approximate word-based example budget.

The live pilot contained **ten synthetic users**, each with 20 historical interactions and two evaluation queries. Running five methods for each query produced **100 generations**.

Synthetic histories let me define contrasting response styles and evaluate specific aspects of compliance. The scorer checks word-count limits, bullets or numbered formatting, specified salutation phrases, and code presence. A response's score is the fraction of applicable checks it passes; a baseline's mean averages its 20 query scores.

## Initial live results

![Mean automated rule satisfaction for all five methods: no personalization 48.75%, static profile 58.33%, retrieval 88.33%, task-only rules 56.67%, and CAPL 71.25%.](/assets/img/writing/capl/live-rule-satisfaction.svg)

*Figure 1. Mean automated style-rule satisfaction on the synthetic live pilot.*

| Method | Mean rule satisfaction |
|---|---:|
| No personalization | 48.75% |
| Static profile | 58.33% |
| TF-IDF retrieval | **88.33%** |
| Task-only rules | 56.67% |
| CAPL | **71.25%** |

CAPL's mean was 12.92 percentage points above the static profile and 14.58 points above task-only extraction. Retrieval scored highest, 17.08 points above CAPL.

The comparison leaves two useful questions for the next iteration. Which preference information survives extraction into rules? And which information is better preserved by showing the generator historical examples directly? The pilot gives those questions a concrete implementation and baseline comparison.

## Differences across contexts

The context breakdown helps locate the aggregate differences:

![Automated rule satisfaction by method and context, with ten technical queries and five executive and casual queries per method.](/assets/img/writing/capl/context-breakdown.svg)

*Figure 2. Context-level means; the query count is shown for each small subgroup.*

| Context | Queries per method | Static profile | Retrieval | CAPL |
|---|---:|---:|---:|---:|
| Technical engineering | 10 | 46.7% | 76.7% | 47.5% |
| Executive briefing | 5 | 80.0% | 100.0% | 90.0% |
| Casual chat | 5 | 60.0% | 100.0% | 100.0% |

CAPL's difference from the static profile is larger in the executive and casual subsets. On technical queries, the two scores are close, and task-only extraction scores higher than CAPL at 58.3%. That variation makes per-context analysis useful for deciding what to investigate next.

I also compared scores for the same user and query:

![Matched comparisons of CAPL's automated score against each other method.](/assets/img/writing/capl/paired-comparisons.svg)

*Figure 3. Higher, equal, and lower CAPL scores on 20 matched queries per opponent.*

| CAPL compared with | Higher | Equal | Lower |
|---|---:|---:|---:|
| No personalization | 11 | 5 | 4 |
| Static profile | 8 | 8 | 4 |
| TF-IDF retrieval | 1 | 9 | 10 |
| Task-only rules | 9 | 7 | 4 |

Keeping ties visible is useful here: against static profiles, eight queries improve, eight stay equal, and four score lower. These comparisons use the same deterministic style scores as the means.

## Operational measurements

| Method | Mean recorded latency (seconds) | Mean estimated prompt + output tokens |
|---|---:|---:|
| No personalization | 51.01 | 429.60 |
| Static profile | 51.67 | 477.40 |
| TF-IDF retrieval | 17.45 | 429.15 |
| Task-only rules | 47.69 | 475.10 |
| CAPL | 51.04 | 463.60 |

The maximum saved PyTorch peak allocation across methods was approximately 7,503 MiB. The latency figures reflect this execution, including extraction or prompt preparation and generation, with lazy loading and caching affecting the measurements. Token counts are word-based estimates of the generator prompt and output, excluding separate preference-agent tokens.

## Limitations

The results above describe automated style compliance in a small synthetic pilot. They do not establish human preference, factual correctness, task usefulness, statistical significance, or a general advantage over retrieval.

A few details define the evaluation's limits:

- Histories and evaluation queries share a small template pool, with no enforced deduplication of underlying query text. The experiment does not establish unseen-topic generalization.
- Two queries share each synthetic user, and the context subsets contain only five or ten queries per method. The means are descriptive.
- Saved token estimates do not establish matched end-to-end inference budgets. Peak allocator measurements do not give total GPU memory usage or a universal deployment bound.
- Context scoping and recency have not been isolated through live ablations. The CSV also omits inferred rules, full responses, retrieved examples, and extraction-path identifiers, limiting diagnosis of individual outcomes.
- The separate 50-user mock run and mock ablations exercise the pipeline with simulated responses. They are implementation checks, separate from the live evidence used here.

The runner also has a resume-aggregation limitation: it restores saved records for mean scores, while pairwise counts cover only newly generated responses. I reconstructed the article's means and matched comparisons from the complete 100-record CSV. The code limitation remains to be fixed before relying on future resumed summaries.

## Reference

Sumuk Shashidhar, Abhinav Chinta, Vaibhav Sahai, and Dilek Hakkani-Tür. 2024. [*Unsupervised Human Preference Learning*](https://aclanthology.org/2024.emnlp-main.200/). EMNLP 2024. [Hugging Face paper page](https://huggingface.co/papers/2410.03731).
