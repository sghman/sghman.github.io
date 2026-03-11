---
layout: page
title: Posts
icon: fas fa-pen-nib
order: 1
---

<style>
/* Today's Pick */
.todays-pick {
  background: #0a0a0a; border: 1px solid #151515;
  border-radius: 10px; padding: 1.25rem 1.5rem; margin-bottom: 2rem;
  cursor: pointer; transition: border-color 0.2s; text-decoration: none; display: block;
}
.todays-pick:hover { border-color: #222; }
.todays-pick .pick-label {
  font-size: 0.65rem; color: #8ab4f8; text-transform: uppercase;
  letter-spacing: 0.08em; margin-bottom: 0.6rem; font-weight: 500;
}
.todays-pick .pick-title {
  font-size: 1.2rem; font-weight: 600; color: #ededed;
  margin-bottom: 0.5rem; line-height: 1.4;
}
.todays-pick .pick-excerpt {
  font-size: 0.88rem; color: #666; line-height: 1.65; margin-bottom: 0.6rem;
}
.todays-pick .pick-meta {
  display: flex; gap: 0.75rem; align-items: center;
  font-size: 0.75rem; color: #444;
}
.todays-pick .pick-score { color: #81c995; font-weight: 600; }

/* Post list */
.post-item {
  display: grid; grid-template-columns: 76px 1fr auto;
  gap: 1.25rem; padding: 1rem 0; border-bottom: 1px solid #0d0d0d;
  align-items: center; transition: all 0.12s; cursor: pointer;
  text-decoration: none;
}
.post-item:hover { padding-left: 0.3rem; }
.post-item .pi-date { font-size: 0.78rem; color: #444; font-variant-numeric: tabular-nums; }
.post-item .pi-title { font-size: 0.95rem; color: #aaa; transition: color 0.12s; line-height: 1.45; }
.post-item:hover .pi-title { color: #fff; }
.post-item .pi-right { display: flex; align-items: center; gap: 0.5rem; flex-shrink: 0; }
</style>

{% assign all_posts = site.posts %}

<!-- Today's Pick: 가장 최근 포스트 -->
{% assign best_post = all_posts | first %}

{% if best_post %}
<div class="todays-pick" onclick="location.href='{{ best_post.url | relative_url }}'" style="cursor:pointer">
  <div class="pick-label">Today's Pick</div>
  <div class="pick-title">{{ best_post.title }}</div>
  <div class="pick-excerpt">{{ best_post.description | default: best_post.excerpt | strip_html | truncate: 120 }}</div>
  <div class="pick-meta">
    <span>{{ best_post.date | date: "%b %d" }}</span>
    {% for tag in best_post.tags limit:1 %}
      {% if tag == 'claude' or tag == 'LLM' %}
        <span class="tag-pill tag-ai">{{ tag }}</span>
      {% elsif tag == 'daily_pulse' %}
        <span class="tag-pill tag-pulse">{{ tag }}</span>
      {% else %}
        <span class="tag-pill tag-infra">{{ tag }}</span>
      {% endif %}
    {% endfor %}
  </div>
</div>
{% endif %}

<!-- Filter chips: 카테고리 기반 하드코딩 -->
<div class="filter-bar" style="display:flex;gap:0.4rem;margin-bottom:1.5rem;flex-wrap:wrap">
  <button class="filter-chip active" onclick="filterPosts('all',this)">All</button>
  <button class="filter-chip" onclick="filterPosts('tech',this)">Tech</button>
  <button class="filter-chip" onclick="filterPosts('daily',this)">Daily</button>
  <button class="filter-chip" onclick="filterPosts('best',this)">Best</button>
</div>

<!-- Post list -->
<div id="post-list">
{% for post in all_posts %}
  {% assign tag_class = 'tag-infra' %}
  {% assign post_cat = 'tech' %}
  {% if post.categories contains 'Daily Bigtech' %}
    {% assign post_cat = 'daily' %}
    {% assign tag_class = 'tag-pulse' %}
  {% endif %}

  <a class="post-item" href="{{ post.url | relative_url }}" data-tag="{{ post_cat }}" data-idx="{{ forloop.index }}">
    <span class="pi-date">{{ post.date | date: "%b %d" }}</span>
    <div><span class="pi-title">{{ post.title | remove: '[Tech] ' | truncate: 80 }}</span></div>
    <div class="pi-right">
      <span class="tag-pill {{ tag_class }}">{{ post_cat }}</span>
    </div>
  </a>
{% endfor %}
</div>

<button class="load-more-btn" id="load-more" onclick="loadMore()" style="display:none">Load more posts →</button>

<script>
(function() {
  const INITIAL = 20;
  const STEP = 20;
  let shown = INITIAL;
  const items = document.querySelectorAll('.post-item');
  const btn = document.getElementById('load-more');

  function applyVisibility(filter) {
    let count = 0;
    items.forEach((el, i) => {
      const tag = el.dataset.tag;
      let match = filter === 'all' || tag === filter;
      if (filter === 'best') match = i < 10;
      if (match) {
        count++;
        el.style.display = count <= shown ? '' : 'none';
      } else {
        el.style.display = 'none';
      }
    });
    btn.style.display = count > shown ? '' : 'none';
  }

  window.filterPosts = function(tag, btnEl) {
    shown = INITIAL;
    document.querySelectorAll('.filter-chip').forEach(b => b.classList.remove('active'));
    btnEl.classList.add('active');
    applyVisibility(tag);
  };

  window.loadMore = function() {
    shown += STEP;
    const active = document.querySelector('.filter-chip.active');
    const tag = active ? active.textContent.trim().toLowerCase() : 'all';
    applyVisibility(tag === 'all' ? 'all' : tag);
  };

  applyVisibility('all');
})();
</script>
