---
layout: page
title: Projects
icon: fas fa-code
order: 7
---

<style>
.proj-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 0.6rem; margin-bottom: 2rem; }
.proj-grid-full { display: grid; grid-template-columns: 1fr; gap: 0.6rem; margin-bottom: 2rem; }
.pc-head { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 0.5rem; }
@media (max-width: 768px) { .proj-grid { grid-template-columns: 1fr; } }
</style>

<p style="font-size:0.82rem;color:#555;margin-bottom:2rem">자동화, AI 파이프라인, 개인 도구 — 만들고 있는 것들</p>

{% assign categories = "automation:자동화 / 데이터,personal:개인 도구,learning:학습" | split: "," %}

{% for cat_entry in categories %}
  {% assign cat_parts = cat_entry | split: ":" %}
  {% assign cat_key = cat_parts[0] %}
  {% assign cat_label = cat_parts[1] %}
  {% assign cat_projects = site.data.projects | where: "category", cat_key %}

  {% if cat_projects.size > 0 %}
  <div class="section-label">{{ cat_label }}</div>
  <div class="{% if cat_projects.size == 1 %}proj-grid-full{% else %}proj-grid{% endif %}">
    {% for project in cat_projects %}
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

      <div class="proj-card" {% if project.url %}onclick="window.open('{{ project.url }}','_blank')"{% endif %}>
        <div class="pc-head">
          <h3>{{ project.name }}</h3>
          <span class="status-badge {{ status_class }}">{{ status_label }}</span>
        </div>
        <p>{{ project.desc }}</p>
        {% if project.tech %}
        <div>
          {% for t in project.tech %}
            <span class="now-tech-tag">{{ t }}</span>
          {% endfor %}
        </div>
        {% endif %}
      </div>
    {% endfor %}
  </div>
  {% endif %}
{% endfor %}

<!-- Recent Activity Timeline -->
{% if site.data.activity.size > 0 %}
<div style="margin-top:2rem">
  <div class="section-label">최근 활동</div>
  {% for item in site.data.activity limit:5 %}
    {% assign dot_color = '#222' %}
    {% if item.color == 'green' %}{% assign dot_color = '#81c995' %}
    {% elsif item.color == 'accent' %}{% assign dot_color = '#8ab4f8' %}
    {% elsif item.color == 'yellow' %}{% assign dot_color = '#fdd663' %}
    {% endif %}

    <div class="tl-item">
      <span class="tl-date">{{ item.date | date: "%b %d" }}</span>
      <span class="tl-dot" style="background:{{ dot_color }}"></span>
      <span class="tl-text"><strong>{{ item.project }}</strong> — {{ item.text }}</span>
    </div>
  {% endfor %}
</div>
{% endif %}
