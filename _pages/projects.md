---
layout: splash
title: "Naman Chawla | Projects"
permalink: /projects/
---

<div class="header-container">
  <div class="name-container">
    <h1 class="author-name">Naman Chawla</h1>
  </div>
  <div class="navigation-container">
    <a href="/" class="nav-link">Home</a>
    <a href="/writing/" class="nav-link">Writing</a>
    <a href="/projects" class="nav-link">Projects</a>
  </div>
</div>

<div class="proj-page">
  <div class="proj-grid">
    <div class="proj-card">
      <a href="https://github.com/palindromeRice/capl" target="_blank" rel="noopener">
        <img src="/assets/img/projects/capl.jpg" alt="CAPL: Context-Aware Preference Rules for Steering a Frozen LLM" loading="lazy">
        <span class="proj-title">CAPL: Context-Aware Preference Rules for Steering a Frozen LLM</span>
      </a>
      <div class="proj-desc">A small model writes context-scoped preference rules that steer a frozen 9B generator; 71.25% style-rule satisfaction vs 58.33% for a static profile · 2026</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/capl" target="_blank" rel="noopener">Code</a></div>
    </div>
    <div class="proj-card">
      <a href="https://github.com/palindromeRice/quake2-llm-decompile" target="_blank" rel="noopener">
        <img src="/assets/img/projects/quake2.jpg" alt="Agent-Assisted Reconstruction of Quake II Gameplay Code" loading="lazy">
        <span class="proj-title">Agent-Assisted Reconstruction of Quake II Gameplay Code</span>
      </a>
      <div class="proj-desc">Coding agents rewrite decompiled Quake II gameplay functions in C; 99.0% pass a differential harness and the rebuilt module is playable · 2026</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/quake2-llm-decompile" target="_blank" rel="noopener">Code</a></div>
    </div>
    <div class="proj-card">
      <a href="https://github.com/palindromeRice/openEnvRL" target="_blank" rel="noopener">
        <img src="/assets/img/projects/browser-agent.jpg" alt="Small Models as Agents: SFT and GRPO on a Browser Task" loading="lazy">
        <span class="proj-title">Small Models as Agents: SFT and GRPO on a Browser Task</span>
      </a>
      <div class="proj-desc">SFT then online GRPO in a live browser takes Qwen3.5-0.8B from 16% to 84% exact-match success on MiniWoB++ click-checkboxes · 2026</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/openEnvRL" target="_blank" rel="noopener">Code</a></div>
    </div>

    <div class="proj-card">
      <a href="https://github.com/palindromeRice/SingleBitLLMs" target="_blank" rel="noopener">
        <img src="/assets/img/projects/singlebit.jpg" alt="Single-Bit Quantization of Transformers/BERT" loading="lazy">
        <span class="proj-title">Single-Bit Quantization of Transformers/BERT</span>
      </a>
      <div class="proj-desc">Benchmarked eight single-bit/ternary quantization strategies inspired by BitNet b1.58; 85.7% accuracy/F1 with STE-quantized BERT on SST-2 · 2025</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/SingleBitLLMs" target="_blank" rel="noopener">Code</a> <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_ai-machinelearning-quantization-share-7329875085433991169-ebNV/" target="_blank" rel="noopener">LinkedIn post</a></div>
    </div>
    <div class="proj-card">
      <a href="https://github.com/palindromeRice/Knowledge_Distillation_T5" target="_blank" rel="noopener">
        <img src="/assets/img/projects/distillation.jpg" alt="T5 Knowledge Distillation for Summarization" loading="lazy">
        <span class="proj-title">T5 Knowledge Distillation for Summarization</span>
      </a>
      <div class="proj-desc">Distilled a fine-tuned T5-Base summarizer into T5-Small, retaining ~90% of teacher ROUGE and 84.7% of teacher BERTScore · 2025</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/Knowledge_Distillation_T5" target="_blank" rel="noopener">Code</a> <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_github-palindromericeknowledgedistillationt5-share-7320404661834825728-HQG4/" target="_blank" rel="noopener">LinkedIn post</a></div>
    </div>
    <div class="proj-card">
      <a href="https://github.com/palindromeRice/Moe_demo" target="_blank" rel="noopener">
        <img src="/assets/img/projects/moe.jpg" alt="Mixture-of-Experts Prompt Router" loading="lazy">
        <span class="proj-title">Mixture-of-Experts Prompt Router</span>
      </a>
      <div class="proj-desc">Semantic MoE routing that dispatches prompts to specialized expert models via sentence-embedding cosine similarity · 2025</div>
      <div class="proj-links"><a href="https://github.com/palindromeRice/Moe_demo" target="_blank" rel="noopener">Code</a> <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_github-palindromericemoedemo-share-7317525096191979520-FJM6/" target="_blank" rel="noopener">LinkedIn post</a></div>
    </div>

  </div>
</div>

<script>
// touch devices: reveal color as cards scroll into view
document.addEventListener('DOMContentLoaded', function () {
  if (window.matchMedia('(hover: none)').matches && 'IntersectionObserver' in window) {
    var io = new IntersectionObserver(function (entries) {
      entries.forEach(function (e) { e.target.classList.toggle('in-view', e.intersectionRatio >= 0.6); });
    }, { threshold: [0, 0.6, 1] });
    document.querySelectorAll('.proj-card').forEach(function (c) { io.observe(c); });
  }
});
</script>

<style>
.proj-page {
  max-width: 1080px;
  margin: 0 auto;
  padding: 1.5rem 1.5rem 6rem;
}

/* topic checkboxes: unchecked = show all; shared style with the Talks page */
.proj-topics {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 2.2rem;
  margin: 1.5rem 0 2.5rem;
}
.proj-topic {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  cursor: pointer;
  color: #1a1a1a;
}
.proj-topic input { accent-color: #043976; width: 15px; height: 15px; cursor: pointer; }

.proj-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  column-gap: 3rem;
  row-gap: 3rem;
}
.proj-card a {
  text-decoration: none !important;
  border-bottom: none !important;
  display: block;
}
.proj-card img {
  width: 100%;
  aspect-ratio: 16 / 10;
  object-fit: cover;
  border-radius: 4px;
  filter: grayscale(1);
  transition: filter 0.5s ease;
  margin-bottom: 0.9rem;
}
.proj-card:hover img {
  filter: grayscale(0);
}
.proj-card img.fit-contain {
  object-fit: contain;
  background: #fff;
}
.proj-title {
  font-size: 1.12rem;
  font-weight: 600;
  color: #1a1a1a;
  line-height: 1.35;
  padding: 0 2px;
  transition: background-color 0.15s ease;
}
.proj-card:hover .proj-title {
  background-color: #dbe7fb;
}
.proj-links { margin-top: 3px; font-size: 0.83rem; }
.proj-links a { display: inline-block !important; color: #1a1a1a; text-decoration: underline !important; margin-right: 0.6rem; border-bottom: none !important; }
.proj-links a:hover { color: #2437e7; }
.proj-desc {
  font-style: italic;
  color: #8a8f98;
  font-size: 0.95rem;
  line-height: 1.5;
  margin-top: 0.3rem;
}

@media (max-width: 767px) {
  .proj-grid { grid-template-columns: 1fr; }
}

/* touch devices have no hover: grayscale by default, color when scrolled into view */
@media (hover: none) {
  .proj-card img { filter: grayscale(1); }
  .proj-card.in-view img { filter: grayscale(0); }
}
</style>
