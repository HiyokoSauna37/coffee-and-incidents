---
layout: default
list_title: "記事一覧"
---

<style>
@import url('https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,500;0,9..144,600;1,9..144,400;1,9..144,500&family=Zen+Old+Mincho:wght@400;500;700;900&family=JetBrains+Mono:wght@400;500&display=swap');

.ci-home {
  --bg: #0e0a06;
  --bg-soft: #14100a;
  --bg-elev: #1c1610;
  --ink: #ece2cc;
  --ink-soft: #c4b89c;
  --ink-mute: #8a7d65;
  --ink-faint: #4d4232;
  --accent: #f5c518;
  --accent-warm: #e9b450;
  --accent-deep: #6B3410;
  --crema: #c49b6c;
  --line: rgba(236, 226, 204, 0.12);
  --line-strong: rgba(236, 226, 204, 0.25);

  --font-display: 'Fraunces', 'Zen Old Mincho', serif;
  --font-jp: 'Zen Old Mincho', 'YuMincho', 'Hiragino Mincho ProN', serif;
  --font-mono: 'JetBrains Mono', 'Menlo', monospace;

  color: var(--ink);
  font-family: var(--font-jp);
  line-height: 1.85;
  letter-spacing: 0.02em;
  position: relative;
}

/* Atmospheric backdrop: desk-lamp glow + grain */
.ci-home::before {
  content: '';
  position: fixed;
  inset: 0;
  background:
    radial-gradient(ellipse 60% 40% at 18% 8%, rgba(245, 197, 24, 0.08), transparent 70%),
    radial-gradient(ellipse 70% 50% at 88% 95%, rgba(107, 52, 16, 0.20), transparent 70%),
    #0e0a06;
  z-index: -2;
  pointer-events: none;
}
.ci-home::after {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='240'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/%3E%3CfeColorMatrix values='0 0 0 0 0.92  0 0 0 0 0.85  0 0 0 0 0.7  0 0 0 0.55 0'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
  opacity: 0.07;
  z-index: -1;
  pointer-events: none;
  mix-blend-mode: overlay;
}

.ci-home a { color: inherit; text-decoration: none; }

/* HERO ============================================== */
.ci-hero {
  padding: 3rem 0 5rem;
  display: grid;
  grid-template-columns: 1.15fr 1fr;
  gap: 4rem;
  align-items: end;
  border-bottom: 1px solid var(--line);
}
.ci-hero__masthead { display: flex; flex-direction: column; gap: 1.6rem; }

.ci-issue-line {
  display: inline-flex;
  align-items: center;
  gap: 0.9em;
  font-family: var(--font-mono);
  font-size: 0.7rem;
  letter-spacing: 0.32em;
  text-transform: uppercase;
  color: var(--accent);
}
.ci-issue-line::before {
  content: '';
  width: 32px;
  height: 1px;
  background: var(--accent);
}

.ci-hero__logo {
  width: clamp(280px, 36vw, 420px);
  max-width: 100%;
  height: auto;
  filter: drop-shadow(0 4px 28px rgba(245, 197, 24, 0.10));
}

.ci-hero__tagline {
  font-family: var(--font-jp);
  font-size: 1.02rem;
  color: var(--ink-soft);
  line-height: 1.85;
  margin: 0;
  max-width: 28em;
  border-left: 1px solid var(--line-strong);
  padding-left: 1.2em;
}

.ci-hero__quote {
  margin: 0;
  padding: 2rem 0 1.2rem 2.2rem;
  border-left: 1px solid var(--accent);
  position: relative;
}
.ci-hero__quote::before {
  content: '"';
  position: absolute;
  top: -0.55em;
  left: 0.7rem;
  font-family: var(--font-display);
  font-style: italic;
  font-weight: 400;
  font-size: 7rem;
  color: var(--accent);
  opacity: 0.32;
  line-height: 1;
}
.ci-hero__quote-text {
  font-family: var(--font-jp);
  font-size: 1.08rem;
  line-height: 2.05;
  color: var(--ink);
  margin: 0 0 1.2rem;
  font-style: italic;
}
.ci-hero__quote-attr {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  letter-spacing: 0.32em;
  text-transform: uppercase;
  color: var(--ink-mute);
}
.ci-hero__quote-attr::before { content: '— '; color: var(--accent); }

/* ABOUT ============================================= */
.ci-about {
  padding: 5rem 0;
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 4rem;
  align-items: start;
  border-bottom: 1px solid var(--line);
}
.ci-about__avatar-wrap {
  position: relative;
  width: 200px;
  height: 200px;
}
.ci-about__avatar-wrap::before {
  content: '';
  position: absolute;
  inset: -12px;
  border: 1px solid var(--accent);
  border-radius: 50%;
  opacity: 0.4;
}
.ci-about__avatar-wrap::after {
  content: 'EST · 2026';
  position: absolute;
  top: 50%;
  right: -3.4em;
  transform: translateY(-50%);
  writing-mode: vertical-rl;
  font-family: var(--font-mono);
  font-size: 0.62rem;
  letter-spacing: 0.5em;
  color: var(--ink-mute);
  text-transform: uppercase;
}
.ci-about__avatar {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid var(--accent);
  box-shadow: 0 0 0 1px rgba(245, 197, 24, 0.10), 0 10px 36px rgba(0, 0, 0, 0.55);
  display: block;
}

.ci-section-label {
  font-family: var(--font-mono);
  font-size: 0.68rem;
  letter-spacing: 0.42em;
  text-transform: uppercase;
  color: var(--accent);
  margin: 0 0 1.6rem;
  display: flex;
  align-items: center;
  gap: 0.9em;
  font-weight: 500;
}
.ci-section-label::after {
  content: '';
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, var(--accent) 0%, transparent 100%);
  opacity: 0.45;
}

.ci-about__name {
  font-family: var(--font-display);
  font-size: 2.3rem;
  font-weight: 500;
  color: var(--ink);
  margin: 0 0 0.3em;
  letter-spacing: 0.01em;
  line-height: 1.2;
}
.ci-about__name em {
  font-style: italic;
  color: var(--accent);
  font-weight: 400;
}
.ci-about__role {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  letter-spacing: 0.36em;
  text-transform: uppercase;
  color: var(--ink-mute);
  margin: 0 0 1.8em;
}
.ci-about__bio {
  font-size: 1rem;
  color: var(--ink-soft);
  line-height: 2;
  margin: 0 0 1.4em;
}
.ci-about__list {
  list-style: none;
  padding: 0;
  margin: 0 0 2em;
  display: grid;
  gap: 0.7em;
}
.ci-about__list li {
  font-size: 0.95rem;
  color: var(--ink-soft);
  padding-left: 1.5em;
  position: relative;
  line-height: 1.85;
}
.ci-about__list li::before {
  content: '◇';
  position: absolute;
  left: 0;
  color: var(--accent);
  font-size: 0.65em;
  top: 0.55em;
}
.ci-about__list a {
  color: var(--accent-warm);
  border-bottom: 1px solid rgba(245, 197, 24, 0.3);
  transition: border-color 0.3s ease;
}
.ci-about__list a:hover { border-bottom-color: var(--accent); }

.ci-about__pull {
  font-family: var(--font-jp);
  font-size: 1.05rem;
  font-style: italic;
  color: var(--ink);
  line-height: 1.9;
  border-left: 2px solid var(--accent);
  padding: 0.3em 0 0.3em 1.4em;
  margin: 0;
}

/* ARCHIVE =========================================== */
.ci-archive { padding: 5rem 0 5.5rem; }
.ci-archive__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 2.5rem;
  gap: 1em;
  flex-wrap: wrap;
}
.ci-archive__title {
  font-family: var(--font-display);
  font-size: 2.6rem;
  font-weight: 500;
  color: var(--ink);
  margin: 0;
  letter-spacing: 0.01em;
  line-height: 1.1;
}
.ci-archive__title em { font-style: italic; color: var(--accent); }
.ci-archive__count {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  letter-spacing: 0.32em;
  color: var(--ink-mute);
  text-transform: uppercase;
}

.ci-posts {
  list-style: none;
  padding: 0;
  margin: 0;
  border-top: 1px solid var(--line);
}
.ci-post {
  border-bottom: 1px solid var(--line);
  position: relative;
  transition: background 0.4s ease;
}
.ci-post::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 2px;
  background: var(--accent);
  transform: scaleY(0);
  transform-origin: top;
  transition: transform 0.5s cubic-bezier(0.2, 0.8, 0.2, 1);
}
.ci-post:hover { background: rgba(245, 197, 24, 0.025); }
.ci-post:hover::before { transform: scaleY(1); }

.ci-post a {
  display: grid;
  grid-template-columns: 130px 1fr auto;
  gap: 2.5rem;
  align-items: start;
  padding: 1.8rem 1.5rem;
}
.ci-post__num {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  letter-spacing: 0.32em;
  color: var(--accent);
  text-transform: uppercase;
  line-height: 1.4;
  padding-top: 0.25em;
}
.ci-post__date {
  display: block;
  color: var(--ink-mute);
  margin-top: 0.5em;
  font-size: 0.7rem;
  letter-spacing: 0.18em;
}
.ci-post__body { min-width: 0; }
.ci-post__title {
  font-family: var(--font-display), var(--font-jp);
  font-size: 1.4rem;
  font-weight: 500;
  line-height: 1.45;
  color: var(--ink);
  margin: 0 0 0.7em;
  transition: color 0.3s ease;
  letter-spacing: 0.005em;
}
.ci-post:hover .ci-post__title { color: var(--accent-warm); }
.ci-post__excerpt {
  font-size: 0.92rem;
  color: var(--ink-mute);
  line-height: 1.85;
  margin: 0;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.ci-post__cta {
  font-family: var(--font-mono);
  font-size: 0.7rem;
  letter-spacing: 0.32em;
  color: var(--ink-mute);
  text-transform: uppercase;
  align-self: center;
  white-space: nowrap;
  transition: color 0.3s ease, transform 0.3s ease;
}
.ci-post:hover .ci-post__cta {
  color: var(--accent);
  transform: translateX(6px);
}

/* Featured = newest dispatch */
.ci-post--featured a {
  padding: 2.5rem 1.5rem 3rem;
  background: linear-gradient(180deg, rgba(245, 197, 24, 0.04), transparent 75%);
}
.ci-post--featured .ci-post__title {
  font-size: 1.95rem;
  line-height: 1.32;
  margin-bottom: 0.9em;
}
.ci-post--featured .ci-post__excerpt {
  -webkit-line-clamp: 3;
  font-size: 0.98rem;
  max-width: 50em;
}
.ci-post--featured::after {
  content: 'CURRENT ISSUE';
  position: absolute;
  top: 1.6rem;
  right: 1.5rem;
  font-family: var(--font-mono);
  font-size: 0.62rem;
  letter-spacing: 0.42em;
  color: var(--accent);
  border: 1px solid var(--accent);
  padding: 0.45em 0.95em;
  border-radius: 999px;
  pointer-events: none;
  background: rgba(14, 10, 6, 0.6);
  backdrop-filter: blur(2px);
}

/* COLOPHON ========================================== */
.ci-colophon {
  text-align: center;
  padding: 3rem 0 1rem;
  border-top: 1px dashed var(--line);
  margin-top: 1rem;
}
.ci-colophon__icon {
  width: 36px;
  height: 36px;
  opacity: 0.45;
}
.ci-colophon p {
  font-family: var(--font-jp);
  font-style: italic;
  font-size: 0.92rem;
  color: var(--ink-mute);
  margin: 0.9em 0 0;
  letter-spacing: 0.04em;
}

/* MOBILE ============================================ */
@media (max-width: 800px) {
  .ci-hero {
    grid-template-columns: 1fr;
    gap: 2.5rem;
    padding: 1.5rem 0 3rem;
    align-items: start;
  }
  .ci-hero__quote { padding-left: 1.6rem; }
  .ci-hero__quote::before { font-size: 5rem; left: 0.4rem; }
  .ci-about {
    grid-template-columns: 1fr;
    gap: 2.5rem;
    padding: 3rem 0;
  }
  .ci-about__avatar-wrap { margin: 0 auto; }
  .ci-about__avatar-wrap::after { display: none; }
  .ci-about__name { font-size: 1.7rem; }
  .ci-archive__title { font-size: 1.9rem; }
  .ci-post a {
    grid-template-columns: 1fr;
    gap: 0.9rem;
    padding: 1.6rem 0.4rem;
  }
  .ci-post__num {
    display: flex;
    gap: 1.4em;
    align-items: baseline;
    padding-top: 0;
  }
  .ci-post__date { margin: 0; }
  .ci-post__cta { align-self: start; }
  .ci-post--featured a { padding: 2rem 0.4rem 2.5rem; }
  .ci-post--featured .ci-post__title { font-size: 1.35rem; }
  .ci-post--featured::after {
    top: 0.8rem;
    right: 0.4rem;
    font-size: 0.55rem;
    padding: 0.35em 0.7em;
  }
}

/* Page-load reveal */
@keyframes ci-rise {
  from { opacity: 0; transform: translateY(14px); }
  to { opacity: 1; transform: translateY(0); }
}
.ci-home > section,
.ci-home > .ci-colophon {
  animation: ci-rise 1s cubic-bezier(0.2, 0.8, 0.2, 1) both;
}
.ci-hero { animation-delay: 0.05s; }
.ci-about { animation-delay: 0.25s; }
.ci-archive { animation-delay: 0.45s; }
.ci-colophon { animation-delay: 0.65s; }
@media (prefers-reduced-motion: reduce) {
  .ci-home > section,
  .ci-home > .ci-colophon { animation: none; }
}
</style>

<div class="ci-home">

<section class="ci-hero">
  <div class="ci-hero__masthead">
    <div class="ci-issue-line">
      <span>Vol. 01</span>
      <span>·</span>
      <span>{{ site.time | date: "%Y — %B" | upcase }}</span>
    </div>
    <img class="ci-hero__logo"
         src="{{ '/assets/images/logo-stacked.svg' | relative_url }}"
         alt="Coffee and Incidents" />
    <p class="ci-hero__tagline">
      バグバウンティとデジタルフォレンジックを、<br>
      コーヒーが冷めるまでに読める長さで。
    </p>
  </div>

  <blockquote class="ci-hero__quote">
    <p class="ci-hero__quote-text">
      椅子を引いて座ってほしい。<br>
      お茶でもコーヒーでも、何か飲み物を手元に用意してほしい。<br>
      アルコールでも構わない。<br>
      むしろアルコールの方がいいかもしれない。
    </p>
    <div class="ci-hero__quote-attr">editor's note</div>
  </blockquote>
</section>

<section class="ci-about">
  <div class="ci-about__avatar-wrap">
    <img class="ci-about__avatar"
         src="{{ '/assets/images/avatar.jpg' | relative_url }}"
         alt="ひよっこサウナ" />
  </div>
  <div class="ci-about__body">
    <h2 class="ci-section-label"><span>About the Author</span></h2>
    <h3 class="ci-about__name">ひよっこサウナ <em>/ HiyokoSauna</em></h3>
    <p class="ci-about__role">SOC Analyst · Tokyo</p>
    <p class="ci-about__bio">
      日々の調査とフォレンジックの現場から、印象に残ったインシデントを書き留めている。
    </p>
    <ul class="ci-about__list">
      <li>SOCアナリスト</li>
      <li>バグバウンティとデジタルフォレンジックに興味あり</li>
      <li>文体は <a href="https://labs.watchtowr.com/">watchTowr Labs</a> のオマージュ ── 皮肉・ユーモア・ドラマ性を交えつつ、技術的事実は一次ソースで裏取り済み</li>
    </ul>
    <p class="ci-about__pull">
      セキュリティのインシデントを、コーヒーの温度が下がるくらいの時間で読めるように、ナラティブで書く場所です。
    </p>
  </div>
</section>

<section class="ci-archive">
  <div class="ci-archive__head">
    <h2 class="ci-archive__title">Latest <em>Dispatches</em></h2>
    <span class="ci-archive__count">{{ site.posts.size }} ENTRIES</span>
  </div>

  <ul class="ci-posts">
    {%- for post in site.posts -%}
      {%- assign n = forloop.rindex -%}
      {%- capture issue_num -%}{%- if n < 10 -%}0{{ n }}{%- else -%}{{ n }}{%- endif -%}{%- endcapture -%}
    <li class="ci-post {% if forloop.first %}ci-post--featured{% endif %}">
      <a href="{{ post.url | relative_url }}">
        <div class="ci-post__num">
          <span>No. {{ issue_num }}</span>
          <span class="ci-post__date">{{ post.date | date: "%Y.%m.%d" }}</span>
        </div>
        <div class="ci-post__body">
          <h3 class="ci-post__title">{{ post.title | escape }}</h3>
          {%- if post.excerpt -%}
          <p class="ci-post__excerpt">{{ post.excerpt | strip_html | strip_newlines }}</p>
          {%- endif -%}
        </div>
        <span class="ci-post__cta">
          {%- if forloop.first -%}この号を読む →{%- else -%}続きを読む →{%- endif -%}
        </span>
      </a>
    </li>
    {%- endfor -%}
  </ul>
</section>

<div class="ci-colophon">
  <img class="ci-colophon__icon"
       src="{{ '/assets/images/logo-icon.svg' | relative_url }}"
       alt="" />
  <p>― コーヒーが冷めたら、また来てください。</p>
</div>

</div>
