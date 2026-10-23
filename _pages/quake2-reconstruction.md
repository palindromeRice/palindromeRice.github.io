---
layout: article
title: "Agent-Assisted Reconstruction of Quake II Gameplay Code"
description: "Coding agents rebuilt Quake II's gameplay module from stripped binaries: 99.0% of testable functions match the original, and the module is playable."
permalink: /writing/quake2-reconstruction/
date: 2026-10-23
---

# Agent-Assisted Reconstruction of Quake II Gameplay Code

## Abstract

Decompilers recover program structure from binaries, but their output rarely compiles or behaves like the original without repair. This study shows that coding agents can close that gap for a complete, stateful game module: Quake II's gameplay code. The module was compiled, stripped of its symbols, and analyzed with the Ghidra decompiler; agents then rebuilt each function in C without seeing the source, and a test harness compared every result against the original on generated game worlds. Raw decompiler output matches the original for fewer than 9% of functions. After agent reconstruction and repair, 99.0% of testable functions match in an unoptimized build (981 of 991) and 98.5% in an optimized build (964 of 979). The rebuilt functions also work together as a real game: the installed module uses reconstructed code for 1,018 of its 1,019 functions and is playable in the open-source Yamagi Quake II engine on Linux and Mac. The results point to a practical path from decompiler output to playable code, with behavioral testing and real play each catching problems the other misses.

<video controls playsinline preload="metadata" poster="/assets/video/writing/quake2-gameplay.jpg">
  <source src="/assets/video/writing/quake2-gameplay.mp4" type="video/mp4">
</video>

*Figure 1. The reconstructed module running in Yamagi Quake II on an Apple Silicon Mac.*

## 1. Introduction

Game decompilation has grown quickly in recent years. Fan projects have rebuilt the source of console classics such as Super Mario 64 and The Legend of Zelda: Ocarina of Time and turned them into native PC ports, and shared tools now let such ports be built for whole families of games [6]. Larger PC games have long-running community efforts too: gta-reversed has been rewriting Grand Theft Auto: San Andreas function by function, swapping each rewritten function into the running game, and estimates it is a little over half done [15].

In 2026, coding agents changed the pace of this work. Lewis used them to help finish a complete decompilation of the Nintendo 64 game Snowboard Kids 2 [5, 7]. Heumann ran fleets of agents for three months on Call of Duty: Modern Warfare 2 (2009), reaching 99% of functions with 83% matching the original byte for byte [8, 12]. AI-assisted ports have multiplied, including an unofficial browser version of Grand Theft Auto V that was taken down within hours of going live [13]. The speed has also caused friction: publishers have pushed back with takedowns, and decomp.dev, a hub for human-led decompilation, shut down in October 2026, saying AI had made it impossible to tell human-led projects from machine-generated ones [14].

Most of these projects aim for *matching* decompilation: C code that compiles back to exactly the same machine code as the original. That gives a clear, automatic check, but it requires the original compiler and a large amount of effort per function. A different question is whether agents can produce code that simply *behaves* like the original, which is what matters for porting, modding, and preservation.

Research on decompilation with language models points the same way. Models can repair decompiler output so it compiles [1, 2], and newer benchmarks check whether the repaired code actually runs correctly [3, 4]. Most of that work tests functions one at a time. A game is a harder test: functions share global state, call each other, and depend on exact data layouts. A function can pass on its own and still break the game.

This study uses Quake II, the first-person shooter id Software released in 1997. Its gameplay code (monsters, weapons, items, doors, and level logic) lives in a separate module of about a thousand C functions, and id Software later released that code as open source. This makes it a useful test case: large and stateful enough to be realistic, with the original available to compare against. This study reconstructs the full gameplay module from stripped binaries, checks each function against the original, and then runs the reconstructed functions together in a playable build. Its contributions are:

- An end-to-end pipeline from decompiler output to agent-written C, with repair driven by test feedback.
- A test harness that compares game state, return values, and engine calls between original and reconstructed functions.
- Results on unoptimized and optimized builds, with an audit that removes passes where no comparison actually took place.
- A record of the failures that only appeared once the game was played.

This is an engineering study. It does not propose a new decompilation technique or aim for byte-identical output.

## 2. Related Work

**Language models for decompilation.** LLM4Decompile [1] studies both direct decompilation with language models and refinement of Ghidra output. Wong et al. [2] repair decompiled C so that it recompiles. This study follows the refinement approach.

**Checking behavior.** DecompileBench [3] adds runtime checks to decompiler evaluation, and PCodeTrans [4] swaps reconstructed functions into real binaries and compares how they execute. Both show that compiling is not enough, which motivates the behavioral comparison used here.

**Agents in game decompilation.** Lewis [5, 7] uses coding agents as assistants in a human-led, byte-matching decompilation of Snowboard Kids 2. Heumann [8] reports a largely autonomous multi-agent decompilation of Modern Warfare 2, again verified by byte matching, and found that agents needed a strict, machine-checkable pass/fail signal to stay on track. This study shares that lesson but uses behavioral tests instead of byte matching, and evaluates the result by playing the game.

## 3. Method

### 3.1 Binaries

The input is id Software's open-source release of the Quake II gameplay code (version 3.19, 48 source files) [9], compiled with GCC on 64-bit Intel Linux and stripped of symbols before analysis with Ghidra [11]. These are builds made for this study, not the original commercial release. Two versions were built:

- **Unoptimized:** compiler optimizations off, so the machine code stays close to the source structure.
- **Optimized:** standard optimizations on, so the compiler inlines functions and removes code, making the original structure harder to recover.

The two runs used different agent configurations and function sets, so the gap between them is not purely an optimization effect.

To play the reconstructed code, the module is loaded by Yamagi Quake II [10], an open-source port of the Quake II engine, with the free demo data id Software released for the game. The demo data contains the first three levels: Outer Base, Installation, and Comm Center. The full game code is reconstructed, but other levels need the data files from a purchased copy of the game.

### 3.2 Reconstruction

For each function, Ghidra provides its decompiled pseudocode, disassembly, a guessed signature, the functions it calls and is called by, and the constants it uses. The agent receives these and writes a C implementation. It never sees the original source: each call runs in an isolated temporary directory with restricted tools. The evaluation and integration tooling does use the source, for function signatures, the reference implementation, declarations, data, and symbol names.

Three stages are compared:

1. **Ghidra baseline:** the raw pseudocode, with standard headers and basic types added.
2. **First agent response:** the agent's first C implementation.
3. **After repair:** up to two rounds of fixes driven by compiler errors or failing tests.

### 3.3 Evaluation

The harness runs the original function and the candidate in separate processes on identical generated game worlds, then compares the results. It checks the state of entities, players, the game, and the level, as well as return values, random-number use, and every call the function makes into the engine. The engine is replaced by a stand-in that always gives the same answers, so any difference in behavior shows up.

Each function is tested on up to 1,500 worlds in a development set and 1,500 in a separate held-out set. Repair sees only development failures. If the original function crashes or hangs on a world, that world is skipped; if only the candidate fails, it counts as a mismatch. The number of usable worlds therefore varies by function.

A function passes only if it has no mismatches **and** at least one usable world in both sets.

## 4. Results

### 4.1 Behavioral agreement

The unoptimized build has 1,019 functions, 991 of which the harness can test. The optimized build has 1,015; Ghidra misses eight, leaving 979 testable. In each build, 28 functions are outside the harness's reach. Functions with no usable worlds still count in the totals.

*Table 1. Functions that behave like the original.*

| Stage | Unoptimized (991 functions) | Optimized (979 functions) |
|---|---:|---:|
| Ghidra baseline | 86 (8.7%) | 76 (7.8%) |
| First agent response | 949 (95.8%) | 918 (93.8%) |
| After repair | 981 (99.0%) | 964 (98.5%) |

Raw Ghidra output works for fewer than 9% of functions. The agents' first attempts work for about 94-96%, and repair closes most of the remaining gap. "After repair" counts a function if any saved attempt passed, so it measures how far the process got, not the success of a single attempt. Measured against all functions in each binary, the after-repair rates are 96.3% and 95.0%.

**Audit.** The harness marks a function as passing when the original crashes or hangs on every world and nothing is actually compared. Table 1 excludes these false passes, which removes ten from the unoptimized results and nine from the optimized ones. The harness bug itself has not been fixed or rerun. Having at least one usable world is a minimum requirement, not proof of thorough testing.

An early restart of the unoptimized run overwrote four failed first attempts with new ones, so the second row reports first *saved* responses.

### 4.2 Playing the game

Running the reconstructed functions together exposed failures the isolated tests missed:

- **Hardcoded addresses:** save/load code kept memory addresses from the test binary. The build script now replaces them with proper references.
- **Wrong record size:** an early weapon-switching function assumed each item record was 120 bytes instead of 136 and crashed. More tests and repair fixed it.
- **Platform assumptions:** some functions relied on how 64-bit Intel code passes structures between functions, which broke on Apple Silicon. The Mac build therefore runs the Intel version through Apple's Rosetta 2 translation layer.
- **Memory layout:** the function that moves doors and platforms assumed certain global variables sat next to each other in memory. This failed on macOS, so the installed build keeps the original version of that one function.

The installed module includes 28 reconstructed functions the harness never tested, so 1,018 reconstructed does not mean 1,018 verified. Original data, declarations, and glue code also remain.

## 5. Limitations and Future Work

Each limitation below comes with what this study did about it and what would address it fully.

**Possible memorization.** Quake II's source has been public for decades, so the agents may have seen it during training. Some reconstructions use original function names that were not in the prompt, which hints at this. To reduce the risk, the agents never received the source, and a small set of newly written control functions, which no model could have seen before, also passed. Those controls were themselves agent-written, though, so the cleanest test is to repeat the study on a game whose source has never been released.

**Test coverage.** Passing the harness shows agreement on the worlds that were tested, not on every possible input. A check that injected deliberate bugs into 100 sampled functions caught 269 of 329 (82%), so most errors are detected, but some rarely reached code paths get little testing. The harness also uses synthetic single-player worlds and a stand-in engine, so multiplayer and many real-engine interactions are not covered. Playing the real game (Section 4.2) helps fill this gap, and targeted inputs for hard-to-reach code paths and multiplayer scenarios are the natural next step.

**Changes to the test harness.** The harness was improved during the study, for example in how it compares pointers and generates inputs. Repair never saw held-out failures, which keeps the reported results honest at the function level. Because I reviewed results and refined the tests along the way, a fully independent check would mean freezing the harness and running it once on fresh held-out worlds.

## 6. Conclusion

Coding agents, given Ghidra output and test feedback, reconstructed nearly all of Quake II's gameplay functions to the point of matching the original on generated test worlds, and the reconstructed module runs as a playable game. The main lesson is that function-level agreement, working integration, and portability are separate claims: each needed its own evidence, and playing the game found bugs that function tests did not. Testing on a game whose source is not public, with stronger test coverage, is the natural next step.

## Code Availability

The pipeline, test harness, saved results, and installer are available at [https://github.com/palindromeRice/quake2-llm-decompile](https://github.com/palindromeRice/quake2-llm-decompile).

## References

1. Hanzhuo Tan, Qi Luo, Jing Li, and Yuqun Zhang. 2024. [LLM4Decompile: Decompiling Binary Code with Large Language Models](https://arxiv.org/abs/2403.05286). *Proceedings of EMNLP 2024*.
2. Wai Kin Wong et al. 2023. [Refining Decompiled C Code with Large Language Models](https://arxiv.org/abs/2310.06530). arXiv:2310.06530.
3. Zeyu Gao et al. 2025. [DecompileBench: A Comprehensive Benchmark for Evaluating Decompilers in Real-World Scenarios](https://arxiv.org/abs/2505.11340). arXiv:2505.11340.
4. Yuxin Cui, Zeyu Gao, Shuxian He, Siliang Qin, and Chao Zhang. 2026. [PCodeTrans: Translate Decompiled Pseudocode to Compilable and Executable Equivalent](https://arxiv.org/abs/2603.14855). arXiv:2603.14855.
5. Chris Lewis. 2025. [Using Coding Agents to Decompile Nintendo 64 Games](https://blog.chrislewis.au/using-coding-agents-to-decompile-nintendo-64-games/). Chris' Blog, 4 November.
6. Wes Fenlon. 2026. [Decompilation projects and N64 Recompiled PC ports list](https://readonlymemo.com/decompilation-projects-and-n64-recompiled-list/). Read Only Memo, updated October 2026.
7. Chris Lewis. 2026. [Snowboard Kids 2 is 100% Decompiled](https://blog.chrislewis.au/snowboard-kids-2-is-100-decompiled/). Chris' Blog, 17 May.
8. Maurice Heumann. 2026. [500+ Billion Tokens Later: Letting AI Agents Decompile a First-Person Shooter](https://momo5502.com/posts/2026-10-09-game-decompilation/). Maurice's Blog, 9 October.
9. id Software. 2001. [Quake II source code](https://github.com/id-Software/Quake-2). GitHub, GPL-2.0.
10. Yamagi Quake II contributors. [Yamagi Quake II](https://github.com/yquake2/yquake2). GitHub.
11. National Security Agency. [Ghidra](https://github.com/NationalSecurityAgency/ghidra). GitHub.
12. Tom's Hardware. 2026. [Dev used AI agents and more than 500 billion tokens to rebuild Call of Duty: Modern Warfare 2's source code](https://www.tomshardware.com/tech-industry/artificial-intelligence/dev-used-ai-agents-and-more-than-500-billion-tokens-to-rebuild-call-of-duty-modern-warfare-2s-source-code-83-percent-of-the-2009-shooters-functions-now-match-the-original-byte-for-byte). October.
13. Tom's Hardware. 2026. [GTA V playable in browser immediately nuked: unofficial WebAssembly port built with AI gets taken down within hours of going live](https://www.tomshardware.com/video-games/pc-gaming/gta-v-playable-in-browser-immediately-nuked-unofficial-webassembly-port-built-with-ai-gets-taken-down-within-hours-of-going-live). October.
14. Jason Koebler. 2026. [Video Game Reverse Engineering Community Shuts Down Because AI Has Made It Too Easy](https://www.404media.co/video-game-reverse-engineering-community-shuts-down-because-ai-has-made-it-too-easy/). 404 Media, 15 October.
15. gta-reversed contributors. [gta-reversed: Reimplementation of GTA:SA 1.0 US](https://github.com/gta-reversed/gta-reversed). GitHub.
