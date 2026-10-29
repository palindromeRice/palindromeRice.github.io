---
layout: splash
title: "Naman Chawla | Writing"
permalink: /writing/
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

<div class="posts-layout">
  <nav class="posts-filter" aria-label="Post type filter">
    <button class="filter-item is-active" data-filter="blog">Papers</button>
    <button class="filter-item" data-filter="articles">Articles</button>
    <button class="filter-item" data-filter="thoughts">Posts</button>
  </nav>

  <div class="blog-list" data-section="01">
  <div class="blog-entry" data-type="blog">
    <a href="https://arxiv.org/abs/2604.07272" class="blog-title" target="_blank" rel="noopener">ClickGuard: A Trustworthy Adaptive Fusion Framework for Clickbait Detection</a>
    <span class="blog-date">arXiv · 2026</span>
  </div>

  <div class="blog-entry" data-type="blog">
    <a href="https://doi.org/10.1007/978-3-032-30438-4_45" class="blog-title" target="_blank" rel="noopener">Skin Cancer Classification Using Ensemble of Deep Learning Models and Transfer Learning</a>
    <span class="blog-date">LNNS · 2026</span>
  </div>

  <div class="blog-entry" data-type="blog">
    <a href="https://doi.org/10.1201/9781003654483-68" class="blog-title" target="_blank" rel="noopener">Decoding Sarcasm: Leveraging GloVe and LSTM for Accurate Sentiment Understanding</a>
    <span class="blog-date">CRC Press · 2025</span>
  </div>

  <div class="blog-entry" data-type="blog">
    <a href="https://link.springer.com/chapter/10.1007/978-981-99-6547-2_49" class="blog-title" target="_blank" rel="noopener">Hybrid Model Approaches Toward Movie Recommendation Systems and Their Comparisons</a>
    <span class="blog-date">LNNS · 2024</span>
  </div>

  <div class="blog-entry" data-type="blog">
    <a href="https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5023927" class="blog-title" target="_blank" rel="noopener">Unveiling Emotions Through Sentiment Analysis</a>
    <span class="blog-date">SSRN · 2024</span>
  </div>

  <div class="blog-entry" data-type="blog">
    <a href="https://dergi.neu.edu.tr/index.php/aiit/article/view/799" class="blog-title" target="_blank" rel="noopener">Gesture-Based Touchless Operations: MediaPipe, OpenCV</a>
    <span class="blog-date">AIIT · 2023</span>
  </div>

  <div class="blog-entry" data-type="articles">
    <a href="/writing/quake2-reconstruction/" class="blog-title">Agent-Assisted Reconstruction of Quake II Gameplay Code</a>
    <span class="blog-date">23/10/2026</span>
  </div>

  <div class="blog-entry" data-type="articles">
    <a href="/writing/browser-agent-sft-grpo/" class="blog-title">Small Models as Agents: SFT and GRPO on a Browser Task</a>
    <span class="blog-date">23/10/2026</span>
  </div>

  <div class="blog-entry" data-type="articles">
    <a href="/writing/capl/" class="blog-title">CAPL: Context-Aware Preference Rules for Steering a Frozen LLM</a>
    <span class="blog-date">23/10/2026</span>
  </div>

  <div class="blog-entry" data-type="thoughts">
    <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_ai-machinelearning-quantization-share-7329875085433991169-ebNV/" class="blog-title" target="_blank" rel="noopener">Single-Bit Quantization in NLP: An Experiment with BitNet b1.58</a>
    <span class="blog-date">05.18.25</span>
  </div>

  <div class="blog-entry" data-type="thoughts">
    <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_github-palindromericeknowledgedistillationt5-share-7320404661834825728-HQG4/" class="blog-title" target="_blank" rel="noopener">NVIDIA’s Llama 3.1 Nemotron Ultra and Knowledge Distillation</a>
    <span class="blog-date">04.22.25</span>
  </div>

  <div class="blog-entry" data-type="thoughts">
    <a href="https://www.linkedin.com/posts/naman-chawla-0563032b5_github-palindromericemoedemo-share-7317525096191979520-FJM6/" class="blog-title" target="_blank" rel="noopener">From Dense to Decisive: How Meta’s Llama 4 Reinvents AI with Experts</a>
    <span class="blog-date">04.14.25</span>
  </div>

  <p class="blog-more">All papers on <a href="https://scholar.google.com/citations?user=Hq_k3_wAAAAJ&hl=en" target="_blank" rel="noopener">Google Scholar</a>.</p>
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function () {
  var buttons = document.querySelectorAll('.posts-filter .filter-item');
  var entries = document.querySelectorAll('.blog-entry');
  var list = document.querySelector('.blog-list');
  var sectionNumbers = { blog: '01', articles: '02', thoughts: '03' };
  function apply(f) {
    var firstVisibleSet = false;
    entries.forEach(function (e) {
      var visible = e.getAttribute('data-type') === f;
      e.style.display = visible ? '' : 'none';
      // Tag the first *visible* entry so its top spacing can be trimmed to hug
      // the tabs. :first-child can't do this because filtered-out entries are
      // still in the DOM (display:none), so it targets a hidden element.
      e.classList.remove('is-first-visible');
      if (visible && !firstVisibleSet) {
        e.classList.add('is-first-visible');
        firstVisibleSet = true;
      }
    });
    if (list) list.setAttribute('data-section', sectionNumbers[f] || '01');
    // Scholar link only makes sense where papers are listed
    var more = document.querySelector('.blog-more');
    if (more) more.style.display = (f === 'blog') ? '' : 'none';
  }
  buttons.forEach(function (btn) {
    btn.addEventListener('click', function () {
      buttons.forEach(function (b) { b.classList.remove('is-active'); });
      btn.classList.add('is-active');
      apply(btn.getAttribute('data-filter'));
    });
  });
  apply('blog');   // Blog is the default view
});
</script>

<style>
/* Header + nav come from the shared site CSS (main.css) via the splash layout,
   so this page matches Home and Papers exactly (background, zoom, header). */

/* Pin the footer to the viewport bottom when the (filtered) list is short:
   stretch the page's wrapper chain and let the motto take the slack. */
body.sticky-bottom-footer {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

body.sticky-bottom-footer > .container.mt-5 {
  display: flex;
  flex-direction: column;
  flex: 1 0 auto;
}

body.sticky-bottom-footer > .container.mt-5 > .container-fluid {
  display: flex;
  flex-direction: column;
  flex: 1 0 auto;
}

/* Motto Section */
.motto-section {
  /* a true full-width footer: the top rule spans the page, echoing the
     header's line so the two frame the content */
  max-width: none;
  margin: 6rem 0 0;
  margin-top: auto;             /* absorb leftover viewport height — line sits at the very bottom */
  padding: 0.5rem 1rem 0.6rem;   /* a flat band — line, breath, text, done */
  border-top: 1px solid #e7e9ee;
  text-align: center;
}

.motto-container {
  position: relative;
  padding: 0;
}

.motto-quote {
  display: inline;   /* quote + attribution on one line */
  font-size: 0.8rem;
  font-weight: 300;
  color: #6b7280;   /* muted — footer furniture, not content */
  font-style: italic;
  line-height: 1.5;
  margin-bottom: 0;
  font-family: Georgia, serif;
  letter-spacing: 0;
}

.motto-attribution {
  display: inline;
  margin-left: 0.6rem;   /* small gap after the quote */
  font-style: italic;    /* fully uniform with the quote — no hierarchy */
  font-family: Georgia, serif;
  font-size: 0.8rem;     /* same size as the quote */
  color: #6b7280;        /* same color as the quote */
  margin-bottom: 0;
  font-weight: 300;
}



@media (max-width: 768px) {
  .motto-quote,
  .motto-attribution {
    font-size: 0.95rem;   /* keep quote + name the same size on mobile too */
  }

  /* on phones, drop the "— Haruki Murakami" to its own second line */
  .motto-attribution {
    display: block;
    margin-left: 0;
    margin-top: 0.35rem;
  }
  
  .motto-section {
    margin: 4rem 0 0;   /* full-width footer on mobile too, flush to the page end */
  }
}

/* ---- Posts layout: vertical filter sidebar + hairline divider (Cognition-style) ---- */
.posts-layout {
  display: flex;
  align-items: flex-start;
  gap: 0;
  max-width: 860px;
  width: 100%;
  margin: 1.5rem auto 14rem;   /* footer sits far below the content, at the very end of the scroll */
  padding: 0 1rem;
  position: relative;
}

/* Column rails: each starts with a bold tick flush against the header line
   (70px above this box) and runs a 1px hairline to the bottom of the section. */
.posts-layout::before,
.posts-layout::after {
  content: '';
  position: absolute;
  top: -3.5rem;    /* reach up to the header rule (2rem margin + 1.5rem gap); rem so it scales with the root font on mobile */
  bottom: 0;
  width: 2px;
  background:
    linear-gradient(#b3b8c2, #b3b8c2) 0 0 / 2px 14px no-repeat,
    linear-gradient(#e7e9ee, #e7e9ee) 0.5px 14px / 1px calc(100% - 14px) no-repeat;
}

.posts-layout::before {
  left: calc(1rem - 1px);              /* menu column rail */
}

.posts-layout::after {
  left: calc(4rem + 110px - 1px);      /* content column rail (menu 110px + 3rem air) */
}

.posts-filter {
  flex: 0 0 110px;
  margin-right: 3rem;    /* clear air between the menu column and the content rail */
  position: sticky;
  top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
  padding-top: 0;         /* menu starts level with the 01 marker */
  padding-left: 0.15rem;  /* text hugs the rail, evenly for every item */
}

.filter-item {
  appearance: none;
  background: none;
  border: none;
  padding: 0;
  margin: 0;
  position: relative;
  text-align: left;
  font-size: 0.95rem;
  font-weight: 400;
  color: #1a202c;
  cursor: pointer;
  line-height: 1.5;
  transition: color 0.15s ease;
}

.filter-item:hover {
  color: #1e3a8a;
}

/* kill the browser's default focus ring/box on click; keep it for keyboard nav */
.filter-item:focus {
  outline: none;
}

.filter-item:focus-visible {
  outline: 2px solid #2437e7;
  outline-offset: 2px;
}

.filter-item.is-active {
  color: #2437e7;   /* Cognition-style blue */
}

/* the "|" indicator floats just outside the rail, like Cognition's */
.filter-item.is-active::before {
  content: '';
  position: absolute;
  left: -0.5rem;   /* just off the rail */
  top: 0.32em;
  width: 3px;
  height: 0.78em;   /* matches the text's cap height, no taller */
  background: #2437e7;
}

.blog-list {
  /* wider than the title's ~600px so the reserved date column doesn't force
     titles to wrap early — line 1 stays long like the original */
  flex: 1 1 auto;
  min-width: 0;
  max-width: 700px;
  margin: 0;
  padding: 0 0 0 2rem;
  position: relative;   /* rail itself is drawn by .posts-layout::after */
}

/* a tiny mono section number hugging the rule, Cognition-style */
.blog-list::after {
  content: attr(data-section);
  position: absolute;
  top: 0;
  left: 0.35rem;
  font-family: 'SFMono-Regular', Menlo, Consolas, monospace;
  font-size: 0.5rem;   /* very small, Cognition-style */
  color: #b3b8c2;
  letter-spacing: 0.04em;
}

@media (max-width: 768px) {
  .posts-layout {
    flex-direction: column;
    gap: 1rem;
  }

  .posts-filter {
    position: static;
    flex-direction: row;
    align-items: baseline;   /* don't stretch buttons — keep the underline snug */
    justify-content: center; /* center the filter tabs on phone */
    width: 100%;             /* fill the row so justify-content can center */
    flex-basis: auto;        /* drop the 110px sidebar width */
    gap: 1.25rem;
    padding-top: 0;
    padding-left: 0;
    border-left: none;
    margin-right: 0;
  }

  /* column furniture is a desktop-only device */
  .blog-list::before,
  .blog-list::after,
  .posts-layout::before,
  .posts-layout::after,
  .filter-item.is-active::before {
    display: none;
  }

  .filter-item {
    border-left: none;
    border-bottom: 2px solid transparent;
    padding: 0 0 0.25rem 0;
  }

  .filter-item.is-active {
    border-left: none;
    border-bottom: 2px solid #2437e7;
  }

  .blog-list {
    border-left: none;
    padding-left: 0;
  }

  /* first *visible* entry hugs the tabs, matching the papers page spacing.
     Keyed off .is-first-visible (set in JS) rather than :first-child so the
     spacing is consistent across every filter, not just the Blog tab. */
  .blog-entry.is-first-visible {
    padding-top: 0;
  }
}

.blog-entry {
  display: flex;
  align-items: baseline;   /* date sits top-right, level with the title's first line */
  gap: 1.5rem;
  padding: 0.9rem 0;
  border-bottom: 1px solid #eef0f3;
}

.blog-entry .blog-title {
  flex: 1 1 auto;
  min-width: 0;   /* let the title wrap; it keeps ~600px so line 1 stays full */
}

.blog-entry:last-child {
  border-bottom: none;
}

.blog-date {
  order: -1;              /* date leads the row, Cognition-style */
  flex: 0 0 4.6rem;       /* fixed column so every title starts on the same rail */
  white-space: nowrap;
  font-family: 'SFMono-Regular', Menlo, Consolas, monospace !important;   /* same face as the 01 marker; beats override.css's span rule */
  font-size: 10px;
  color: #9aa4b2;
  letter-spacing: 0.02em;
}

.blog-date.blog-soon {
  font-style: italic;
  color: #b8c0cc;
}

.blog-title {
  font-size: 1.08rem;
  font-weight: 500;
  color: #043976;   /* site dark blue, matches the Publications links */
  text-decoration: none !important;
  border-bottom: none !important;
  outline: none !important;
  line-height: 1.4;
  position: relative;
  display: inline-block;
  transition: color 0.2s ease;
}

/* Hover: no underline (the entry divider already draws a line) —
   light the whole row up in the accent blue, Cognition-style.
   Row-level hover so title and date change together. */
.blog-entry:hover .blog-title,
.blog-entry:hover .blog-date {
  color: #2437e7 !important;
}

.blog-date {
  transition: color 0.2s ease;
}

/* Remove underline from all link states */
.blog-title:link,
.blog-title:visited,
.blog-title:focus,
.blog-title:active {
  text-decoration: none !important;
  border-bottom: none !important;
  outline: none !important;
}

    /* Brand-blue text selection */
    ::selection { background: #2437e7; color: #fff; }
    ::-moz-selection { background: #2437e7; color: #fff; }
</style>

