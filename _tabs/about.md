---
layout: page
title: About
icon: fas fa-user
order: 5
---

<div style="max-width:560px">

  <!-- Avatar + Name -->
  <div style="display:flex;align-items:center;gap:1.25rem;margin-bottom:2rem">
    <img src="{{ '/assets/img/avatar.png' | relative_url }}" alt="avatar"
         style="width:64px;height:64px;border-radius:50%;border:1px solid #151515;object-fit:cover"
         onerror="this.style.display='none'">
    <div>
      <h2 style="font-size:1.15rem;font-weight:600;color:#ededed;margin:0">Gyu Hwan</h2>
      <p style="font-size:0.78rem;color:#555;margin:0.15rem 0 0">Building things that automate things</p>
    </div>
  </div>

  <!-- Bio -->
  <div style="font-size:0.88rem;color:#666;line-height:1.8;margin-bottom:2.5rem">
    <strong style="color:#999;font-weight:500">백엔드와 인프라를 경험한 엔지니어</strong>입니다.
    코드 한 줄보다 <em>왜 이 결정을 했는가</em>에 집중합니다.
    <br><br>
    이 블로그는 직접 만든 파이프라인으로 운영됩니다.
    22개 소스에서 기술 기사를 수집하고, AI가 평가·생성한 후 자동 발행합니다.
  </div>

  <!-- Now Section -->
  <div class="section-label">Now — 지금 하고 있는 것</div>

  {% for project in site.data.projects %}
    {% assign status_label = project.status %}
    {% assign status_class = 'status-idea' %}
    {% if project.status == 'live' %}
      {% assign status_label = '운영 중' %}
      {% assign status_class = 'status-live' %}
    {% elsif project.status == 'dev' %}
      {% assign status_label = '진행 중' %}
      {% assign status_class = 'status-dev' %}
    {% elsif project.status == 'plan' %}
      {% assign status_label = '기획 완료' %}
      {% assign status_class = 'status-plan' %}
    {% elsif project.status == 'idea' %}
      {% assign status_label = '아이디어' %}
      {% assign status_class = 'status-idea' %}
    {% endif %}

    <div class="now-item">
      <div style="flex-shrink:0;margin-top:0.15rem">
        <span class="status-badge {{ status_class }}">{{ status_label }}</span>
      </div>
      <div class="now-info">
        <h3>{{ project.name }}</h3>
        <p>{{ project.desc }}</p>
        {% if project.tech %}
        <div>
          {% for t in project.tech %}
            <span class="now-tech-tag">{{ t }}</span>
          {% endfor %}
        </div>
        {% endif %}
      </div>
    </div>
  {% endfor %}

  <!-- Pipeline Flow -->
  <div class="flow-card" style="margin-top:2rem">
    <h3>How This Blog Works</h3>
    <div class="flow-steps">
      <span class="flow-step">RSS 22개</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">Naver API</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">Tavily</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">기계 필터</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">Claude AI 평가</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">본문 생성</span>
      <span class="flow-arrow">→</span>
      <span class="flow-step">Jekyll 발행</span>
    </div>
  </div>

  <!-- Live Stats -->
  {% assign dash = site.data.dashboard %}
  {% if dash %}
  <div class="live-stats">
    <div class="live-stat">
      <div class="num">{{ dash.summary.total_posts | default: site.posts.size }}</div>
      <div class="lbl">Published Posts</div>
    </div>
    <div class="live-stat">
      <div class="num">22</div>
      <div class="lbl">RSS Sources</div>
    </div>
    <div class="live-stat">
      <div class="num">${{ dash.summary.monthly_cost_usd | default: '0.00' }}</div>
      <div class="lbl">Monthly Cost</div>
    </div>
  </div>
  {% endif %}

  <!-- Social -->
  <div class="social-row">
    <a href="https://github.com/sghman" class="social-link" target="_blank">→ GitHub</a>
  </div>

</div>
