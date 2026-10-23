---
layout: article
title: "Small Models as Agents: SFT and GRPO on a Browser Task"
description: "On a simple browser task, a 0.8B model went from 16% to 84% exact success with self-generated demonstrations and 20 GRPO updates, on a single GPU."
permalink: /writing/browser-agent-sft-grpo/
date: 2026-10-23
math: true
---
# Small Models as Agents: SFT and GRPO on a Browser Task

## Abstract

We study how far a sub-1B language model can be taken on a browser task using a standard two-stage post-training recipe: supervised fine-tuning (SFT) on self-generated demonstrations, followed by online reinforcement learning with Group Relative Policy Optimization (GRPO). Starting from Qwen3.5-0.8B, we collect 50 episodes that the environment scores as successful, fine-tune a LoRA adapter on them, and then run 20 GRPO updates in a live browser with a binary environment reward. On the MiniWoB++ `click-checkboxes` task, chosen as a simple first browser task, exact-match success rises from 16% (base) to 42% (SFT) to 84% (GRPO) on 50 fresh randomized episodes per stage, with the full pipeline running on a single L4 GPU and the GRPO stage taking about ten minutes. Per-episode trace analysis shows that SFT mainly teaches the output format and task structure, while GRPO eliminates a repeated-click failure loop and resolves most one- and two-box puzzles. The remaining failures are dominated by submitting one box short, a behavior consistent with the partial-credit-tolerant reward used in training. We report results under both exact-match and the environment's default partial-credit success criterion, and discuss the limits of a single-task, single-run study.

## 1. Introduction

Web agents are typically built on large language models, with capability attributed largely to scale. Smaller models are attractive for cost and latency but are often assumed to be incapable of reliable tool use in an interactive environment. This work asks a narrower question: on a simple browser task, how much can a 0.8B model gain from a cheap, standard post-training pipeline, and what does each stage contribute?

We use MiniWoB++ `click-checkboxes`, in which an agent must check exactly the requested boxes on a web page and submit. The task is simple, but it exercises the core loop of a browser agent: parse a page representation, select controls by identifier, act, track state across steps (clicking a checkbox twice unchecks it), and decide when to finish. Its small scale also allows the full loop, including a live browser inside the reinforcement-learning loop, to run on one GPU.

Our contributions are:

1. An end-to-end pipeline (collection, SFT, online GRPO, evaluation) for a 0.8B model on a live browser task, fitting on a single L4 GPU.
2. Evidence that SFT on noisy self-generated demonstrations followed by 20 GRPO updates raises exact-match success from 16% to 84%.
3. A per-episode behavioral analysis across all three stages, identifying a repeated-click loop that GRPO removes and a stop-one-short failure that persists.
4. A discussion of how the success criterion changes reported numbers: the environment's default partial-credit count yields 30%, 64%, and 94% for the same episodes.

## 2. Related Work

**Web-interaction benchmarks.** MiniWoB (Shi et al., 2017) introduced a suite of small synthetic web tasks, later extended as MiniWoB++ (Liu et al., 2018). BrowserGym (Le Sellier De Chezelles et al., 2024) provides a unified gym-style interface to MiniWoB++ and other web benchmarks, including accessibility-tree observations. Prior work reached high MiniWoB++ performance with large-scale behavioral cloning and reinforcement learning (Humphreys et al., 2022); our focus is instead a minimal compute budget and a small pretrained language model.

**Learning from self-generated data.** Fine-tuning a model on its own successful outputs, filtered by a verifier, has been studied as rejection-sampling fine-tuning (Yuan et al., 2023) and self-taught reasoning (Zelikman et al., 2022). Our collection stage is an instance of this idea in an interactive environment, with the environment's reward serving as the filter.

**Group-relative policy optimization.** GRPO (Shao et al., 2024) replaces a learned value baseline with the mean reward of a group of sampled completions for the same prompt, and was later used at scale for reasoning models (DeepSeek-AI, 2025). We apply it with very small groups (four completions) and a very short horizon (20 updates).

**Parameter-efficient fine-tuning.** We use LoRA (Hu et al., 2021) for the SFT stage.

## 3. Task and Environment

### 3.1 Task

Each `click-checkboxes` episode presents a page with a randomized set of labeled checkboxes, a Submit button, and an instruction such as "Select MRW, 628 and click Submit." Requested sets range from zero boxes ("Select nothing and click Submit") to four or five. Labels and element identifiers are randomized per episode, so answers cannot be memorized.

### 3.2 Observation and action spaces

The agent observes an accessibility tree serialized as text, in which each control has a numeric identifier. Figure 1 shows the observation for the example instruction above.

```
[18] checkbox 'MRW'
[21] checkbox '628'
[24] checkbox 'w0e'
[27] button 'Submit'
```

*Figure 1. Accessibility-tree observation for the instruction "Select MRW, 628 and click Submit."*

The agent acts through a single tool, `click(target)`, emitted as a Qwen-style tool call. An episode ends when the agent clicks Submit or exhausts a budget of ten actions.

### 3.3 Reward and success criteria

MiniWoB++ scores each submission with a raw reward in $$[-1, 1]$$ that grants partial credit: for instance, checking two of three requested boxes yields approximately 0.67, and checking unrequested boxes can make the score negative. The environment wrapper used in this work converts this to a binary reward, assigning 1 to any positive raw score.

We therefore distinguish two criteria:

- **Exact-match success:** raw reward equal to 1.0, i.e. precisely the requested boxes were checked at submission. This is our primary metric.
- **Default success:** raw reward greater than 0, the environment wrapper's criterion.

Demonstration collection and GRPO training both used the default (lenient) criterion; all headline results are reported under exact match.

Table 1 summarizes the setup.

| Component | Value |
| --- | --- |
| Model | `Qwen/Qwen3.5-0.8B` |
| Environment | MiniWoB++ `click-checkboxes` via BrowserGym |
| Observation | Serialized accessibility tree |
| Action | `click(target)` tool call |
| Action budget | 10 per episode |
| Training reward | 1 if raw reward > 0, else 0 |
| Primary metric | Exact-match success (raw reward = 1.0) |
| Hardware | One NVIDIA L4 GPU (Google Colab) |

*Table 1. Experimental setup.*

## 4. Method

The pipeline has four stages: (i) collect demonstrations with the base model, (ii) fine-tune on them, (iii) improve the fine-tuned model with online GRPO, and (iv) evaluate all three models under one protocol. All datasets, checkpoints, logs, and per-episode records are persisted to cloud storage.

### 4.1 Demonstration collection

We run the base model on freshly sampled puzzles and retain the first 50 episodes that the environment scores as successful under the default criterion. No human or scripted demonstrations are used. Because the filter accepts partial credit, the resulting set is a mixture of exact solutions and partial selections, and is skewed toward easy puzzles (Table 2).

| Boxes requested | Exact solutions | Partial selections | Total |
| ---: | ---: | ---: | ---: |
| 0 | 21 | 0 | 21 |
| 1 | 4 | 11 | 15 |
| 2 | 1 | 12 | 13 |
| 3 | 0 | 1 | 1 |
| **All** | **26** | **24** | **50** |

*Table 2. Composition of the 50 self-generated demonstrations, by requested set size and exactness.*

Only 26 of the 50 demonstrations are exact, 21 of which are zero-box puzzles. The set contains a single exact two-box solution and no exact solutions with three or more boxes.

### 4.2 Supervised fine-tuning

Each episode is decomposed into one training row per executed action, yielding 65 rows. A LoRA adapter is trained on these rows and merged into the base weights (Table 3). The 60 updates with four examples each correspond to roughly 3.7 passes over the data.

| Setting | Value |
| --- | --- |
| Training rows | 65 (from 50 episodes) |
| Method | LoRA, rank 16, alpha 16 |
| Optimizer updates | 60 |
| Batch size × gradient accumulation | 1 × 4 |
| Learning rate | 2e-4 |
| Sequence length | 1024 |

*Table 3. SFT hyperparameters.*

### 4.3 Online GRPO

GRPO starts from the merged SFT model. For each prompt, the policy samples a group of $$G = 4$$ completions. Each completion is executed in its own live browser instance, and the environment returns a binary reward $$r_i$$. The advantage of completion $$i$$ is its reward relative to the group:

$$
A_i = \frac{r_i - \operatorname{mean}(r_1, \ldots, r_G)}{\operatorname{std}(r_1, \ldots, r_G)}
$$

Completions that outperform their group are reinforced and those that underperform are suppressed. A group in which all completions receive the same reward contributes no gradient signal. A KL penalty with coefficient $$\beta$$ regularizes the policy toward the SFT reference. Hyperparameters are given in Table 4.

| Setting | Value |
| --- | --- |
| Initialization | Merged SFT model |
| Optimizer updates | 20 |
| Group size $$G$$ | 4 |
| Total rollouts | About 80 |
| Batch size × gradient accumulation | 1 × 4 |
| Learning rate | 1e-6 |
| KL coefficient $$\beta$$ | 0.04 |
| Prompt / completion limit | 1024 / 128 tokens |
| Optimizer | 8-bit AdamW |
| Reward | 1 if raw reward > 0, else 0 |
| Wall-clock time | About 10 minutes |

*Table 4. GRPO hyperparameters.*

### 4.4 Evaluation protocol

All three models (base, SFT, GRPO checkpoint after 20 updates) are evaluated with the same tool-call evaluator, in the same software environment, on 50 freshly randomized episodes each. The evaluator parses every tool call in a completion in order and executes one per browser turn, resolving each target against the current page state. A model may therefore emit a complete multi-action plan in a single generation. Figure 2 shows such an output from the GRPO model.

```
<tool_call>
<function=click>
<parameter=target>18</parameter>
</function>
</tool_call>
<tool_call>
<function=click>
<parameter=target>21</parameter>
</function>
</tool_call>
<tool_call>
<function=click>
<parameter=target>27</parameter>
</function>
</tool_call>
```

*Figure 2. GRPO model output for "Select MRW, 628 and click Submit": two checkbox clicks followed by Submit, in one generation.*

We report 95% Wilson score intervals (Wilson, 1927) for success rates. Because each stage is evaluated on an independently sampled set of puzzles, cross-stage comparisons are descriptive rather than paired.

## 5. Results

### 5.1 Main results

Table 5 reports success under both criteria. Exact-match success increases from 16% to 42% with SFT and to 84% with GRPO. The SFT and GRPO intervals do not overlap.

| Model | Exact successes | Exact success | 95% interval | Default success (raw > 0) |
| --- | ---: | ---: | ---: | ---: |
| Base | 8 / 50 | 16% | 8% to 29% | 30% |
| + SFT | 21 / 50 | 42% | 29% to 56% | 64% |
| + GRPO | 42 / 50 | 84% | 71% to 92% | 94% |

*Table 5. Success rates on 50 fresh episodes per model. Intervals are 95% Wilson intervals on exact-match success.*

![Exact-match success by training stage](/assets/img/writing/browser-agent/figure1_stage_progression.png)

*Figure 3. Exact-match success by training stage.*

The gap between the two criteria narrows with training. For the base model, 7 of the 15 episodes counted as successful by default are partial selections; for the GRPO model, 5 of 47 are.

### 5.2 Episode length

Table 6 reports action counts. The base model exhausts its action budget in 30 of 50 episodes, the SFT model in 8, and the GRPO model in none. The base model's exact successes average 1.1 actions, nearly all single Submit clicks on zero-box puzzles.

| Model | Avg actions | Budget exhausted | Avg actions, successes | Avg actions, failures |
| --- | ---: | ---: | ---: | ---: |
| Base | 6.58 | 30 | 1.12 | 7.62 |
| SFT | 3.76 | 8 | 2.81 | 4.45 |
| GRPO | 2.88 | 0 | 2.76 | 3.50 |

*Table 6. Action counts per episode. Success and failure use the exact-match criterion.*

### 5.3 Training dynamics

Figure 4 shows the GRPO training log. Mean training reward per update rises from near zero to around 0.75, while the mean number of tool calls per completion rises from near zero to approximately three by the fifth update. The latter indicates a shift toward emitting complete multi-action plans in a single generation. Each point aggregates only four rollouts, so per-update values are noisy, and the training reward uses the default criterion.

![GRPO training dynamics](/assets/img/writing/browser-agent/figure_grpo_training.png)

*Figure 4. GRPO training: mean reward per update with a 5-update moving average (left) and mean tool calls per completion (right).*

## 6. Analysis

### 6.1 Success by puzzle size

Table 7 and Figure 5 break exact-match success down by the number of requested boxes.

| Boxes requested | Base | SFT | GRPO |
| --- | ---: | ---: | ---: |
| 0 | 7 / 8 | 3 / 4 | 4 / 4 |
| 1 | 1 / 11 | 4 / 12 | 13 / 13 |
| 2 | 0 / 14 | 10 / 22 | 17 / 19 |
| 3 or more | 0 / 17 | 4 / 12 | 8 / 14 |

*Table 7. Exact-match successes by requested set size.*

![Exact-match success by puzzle size](/assets/img/writing/browser-agent/figure_success_by_puzzle_size.png)

*Figure 5. Exact-match success by number of requested checkboxes, per stage.*

Three patterns emerge. First, the base model's 16% comes almost entirely from zero-box puzzles (7 of 8), with one exact success in the 42 puzzles that require at least one box. Second, SFT yields non-trivial success at every set size from one box upward (33% to 45%), including two-box and larger puzzles for which its demonstrations contain one exact example or none. This suggests SFT transfers the task's structure (locate requested labels, click their identifiers, submit) rather than memorized solutions. Third, GRPO solves all zero- and one-box puzzles in the sample and 17 of 19 two-box puzzles; its residual errors are concentrated in puzzles with three or more boxes. Bucket sizes range from 4 to 22 episodes and samples differ across stages, so these per-bucket rates are indicative only.

### 6.2 Failure modes across stages

We assign each failed episode to one of three categories using deterministic rules over the requested labels, the visible controls, and the executed actions (Table 8).

| Failure type | Base | SFT | GRPO |
| --- | ---: | ---: | ---: |
| Budget exhausted without submitting | 30 | 8 | 0 |
| Premature Submit (subset of requested boxes) | 12 | 13 | 6 |
| Unrequested box selected, then Submit | 0 | 8 | 2 |
| **Total** | **42** | **29** | **8** |

*Table 8. Failure categories per stage, under exact match.*

The budget-exhausted category is dominated by a repeated-click loop. In 24 of the base model's 30 such episodes, the model clicks the same checkbox on all ten steps, toggling it on and off without ever submitting; in 5 further episodes it emits no executable action. All 8 budget-exhausted SFT episodes follow the same repeated-click pattern. GRPO exhibits no such episodes. This is consistent with the group-relative objective: a looping completion always receives zero reward and is suppressed whenever another completion in its group scores. SFT on demonstrations alone does not provide an explicit signal against such behavior.

### 6.3 Residual failures of the GRPO model

The GRPO model fails on 8 of 50 episodes (Figures 6 and 7). Six are premature submissions, five of which select all but one of the requested boxes before submitting within a single, otherwise well-formed plan. The remaining two select an unrequested box. All eight occur on puzzles with two or more requested boxes.

![GRPO evaluation outcomes](/assets/img/writing/browser-agent/figure5_final_episode_summary.png)

*Figure 6. Outcomes of the 50 GRPO evaluation episodes under exact match: 42 successes, 8 failures.*

![GRPO failure categories](/assets/img/writing/browser-agent/figure6_final_episode_failure_modes.png)

*Figure 7. Categories of the 8 GRPO failures.*

The dominance of stop-one-short failures aligns with the training reward. Because GRPO was trained with the default criterion, a completion selecting two of three requested boxes receives the same reward as an exact solution, so the objective provides no pressure to recover the final box. Training with an exact-match reward is a direct test of this explanation, which we leave to future work.

These categories describe observed actions, not model intent; this is behavioral trace analysis rather than mechanistic interpretability.

## 7. Discussion

**A short RL stage on a small model.** Twenty GRPO updates, about ten minutes of training, doubled exact-match success over SFT (42% to 84%). On this task, small groups and a short horizon were sufficient for a substantial gain.

**SFT tolerates noisy demonstrations.** Although only about half of the demonstrations were exact and nearly all exact ones were trivial, SFT produced a model that solved a meaningful fraction of multi-box puzzles and gave GRPO a starting point with non-zero reward. Collecting these demonstrations required only running the base model.

**Complementary roles.** In our traces, SFT primarily instilled the output format and the select-then-submit structure, while GRPO removed degenerate looping and improved box selection. This division is consistent with the common view of SFT as initialization and RL as refinement, though a single task cannot establish it in general.

**The success criterion matters.** The environment's default criterion reported the base model at 30%; exact match reported 16%, almost all from zero-box puzzles. The same leniency in the training signal plausibly explains the GRPO model's dominant residual failure. Reporting both criteria, and stating which one is used for training, avoids overstating results.

## 8. Limitations

- **Single, simple task.** All results concern randomized `click-checkboxes` episodes. We make no claim about other browser tasks, real websites, or long-horizon workflows.
- **Single training run.** Variance across GRPO seeds was not measured.
- **Small samples.** Fifty episodes per stage yield wide intervals, and per-size buckets are smaller still.
- **Unpaired evaluation.** Each stage was evaluated on an independently sampled set of puzzles rather than a fixed test set.
- **Lenient training signal.** Demonstration filtering and GRPO training used a partial-credit-tolerant reward, while results are reported under exact match.

## 9. Conclusion

On a simple browser task, a 0.8B model trained with self-generated demonstrations and 20 GRPO updates reached 84% exact-match success, up from 16% for the base model, on a single GPU. Trace-level analysis attributes the gains to distinct effects of each stage and links the remaining failures to the leniency of the training reward. Natural next steps are an exact-match training reward, a fixed seeded test set shared across stages, multiple GRPO seeds, and extension to MiniWoB++ tasks involving text entry, menus, and multi-page interaction.

## Code and Data

Code, configurations, and per-episode evaluation records are available at [github.com/palindromeRice/openEnvRL](https://github.com/palindromeRice/openEnvRL).

## References

- DeepSeek-AI. (2025). DeepSeek-R1: Incentivizing reasoning capability in LLMs via reinforcement learning. arXiv:2501.12948.
- Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. (2021). LoRA: Low-rank adaptation of large language models. arXiv:2106.09685.
- Humphreys, P. C., Raposo, D., Pohlen, T., et al. (2022). A data-driven approach for learning to control computers. ICML 2022.
- Le Sellier De Chezelles, T., Gasse, M., Drouin, A., et al. (2024). The BrowserGym ecosystem for web agent research. arXiv:2412.05467.
- Liu, E. Z., Guu, K., Pasupat, P., Shi, T., & Liang, P. (2018). Reinforcement learning on web interfaces using workflow-guided exploration. ICLR 2018.
- Shao, Z., Wang, P., Zhu, Q., et al. (2024). DeepSeekMath: Pushing the limits of mathematical reasoning in open language models. arXiv:2402.03300.
- Shi, T., Karpathy, A., Fan, L., Hernandez, J., & Liang, P. (2017). World of Bits: An open-domain platform for web-based agents. ICML 2017.
- Wilson, E. B. (1927). Probable inference, the law of succession, and statistical inference. Journal of the American Statistical Association, 22(158), 209-212.
- Yuan, Z., Yuan, H., Li, C., Dong, G., Tan, C., & Zhou, C. (2023). Scaling relationship on learning mathematical reasoning with large language models. arXiv:2308.01825.
- Zelikman, E., Wu, Y., Mu, J., & Goodman, N. D. (2022). STaR: Bootstrapping reasoning with reasoning. NeurIPS 2022.
