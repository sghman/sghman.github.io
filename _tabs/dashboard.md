---
layout: page
title: 기술 트렌드 대시보드
permalink: /dashboard/
icon: fas fa-chart-line
order: 6
---

<style>
/* 오른쪽 패널(인기 태그) 숨기고 main 풀 와이드 */
#panel-wrapper { display: none !important; }
#main-wrapper main.col-xl-9 {
  max-width: 100% !important;
  flex: 0 0 100% !important;
}

.db-grid { display: grid; gap: 1.5rem; }

/* ===== P3: Pipeline Status Bar ===== */
.pipeline-status {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.625rem 1rem;
  background: var(--card-header-bg);
  border-radius: var(--bs-border-radius, 0.625rem);
  font-size: 0.8rem;
  color: var(--text-color);
  flex-wrap: wrap;
}
.pipeline-status .status-dot {
  width: 8px; height: 8px;
  border-radius: 50%;
  animation: pulse 2s infinite;
}
.pipeline-status .status-dot.ok { background: #81c995; }
.pipeline-status .status-dot.warn { background: #fdd663; }
.pipeline-status .status-dot.fail { background: #f28b82; }
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.4} }
.pipeline-status .status-detail {
  margin-left: auto;
  font-size: 0.75rem;
  color: var(--text-muted-color);
}

/* ===== Summary Cards ===== */
.summary-cards {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
}
.summary-card {
  background: var(--card-header-bg);
  border-radius: var(--bs-border-radius, 0.625rem);
  padding: 1rem 1.25rem;
}
.summary-card__label {
  font-size: 0.75rem;
  color: var(--text-muted-color);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.summary-card__value {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--heading-color);
  margin-top: 0.25rem;
}
.summary-card__delta {
  font-size: 0.7rem;
  margin-top: 0.15rem;
}
.delta-up { color: #81c995; }
.delta-down { color: #f28b82; }
.delta-neutral { color: var(--text-muted-color); }

/* ===== P2: Filter Bar ===== */
.filter-bar {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: center;
}
.filter-btn {
  padding: 0.3rem 0.875rem;
  border-radius: 0.375rem;
  border: 1px solid var(--main-border-color);
  background: var(--card-header-bg);
  color: var(--text-muted-color);
  font-size: 0.78rem;
  cursor: pointer;
  transition: all 0.2s;
}
.filter-btn:hover, .filter-btn.active {
  border-color: var(--link-color);
  color: var(--link-color);
  background: transparent;
}
.filter-sep {
  width: 1px; height: 20px;
  background: var(--main-border-color);
}
.kw-filter-btn {
  padding: 0.3rem 0.875rem;
  border-radius: 0.375rem;
  border: 1px solid var(--main-border-color);
  background: var(--card-header-bg);
  color: var(--text-muted-color);
  font-size: 0.78rem;
  cursor: pointer;
  transition: all 0.2s;
  display: flex; align-items: center; gap: 0.25rem;
}
.kw-filter-btn:hover, .kw-filter-btn.active {
  border-color: var(--link-color);
  color: var(--text-color);
}
.kw-filter-btn .dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  display: inline-block;
}

/* ===== P1: Posts Table (improved) ===== */
.posts-table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
.posts-table th {
  text-align: left;
  color: var(--label-color);
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--main-border-color);
  font-weight: 600;
}
.posts-table td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--main-border-color);
  color: var(--text-color);
  vertical-align: top;
}
.posts-table tr:last-child td { border-bottom: none; }
.posts-table tr:hover { background: rgba(255,255,255,0.02); }

/* P1: 요약 말줄임 */
.summary-cell {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 500px;
  transition: all 0.3s;
}
.summary-cell:hover {
  -webkit-line-clamp: unset;
}

/* P1: 품질 뱃지 개선 */
.quality-badge {
  display: inline-block;
  padding: 0.1rem 0.5rem;
  border-radius: 1rem;
  font-size: 0.75rem;
  font-weight: 600;
  min-width: 2.5rem;
  text-align: center;
}
.quality-high { background: rgba(129,201,149,0.15); color: #81c995; }
.quality-mid  { background: rgba(253,214,99,0.15);  color: #fdd663; }
.quality-low  { background: rgba(242,139,130,0.15); color: #f28b82; }

/* Keyword badge in table */
.kw-badge {
  display: inline-block;
  padding: 0.05rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.72rem;
  font-weight: 500;
}
.kw-ai   { background: rgba(138,180,248,0.15); color: #8ab4f8; }
.kw-infra { background: rgba(129,201,149,0.15); color: #81c995; }

/* ===== Charts ===== */
.chart-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}
.chart-box, .chart-full {
  background: var(--card-header-bg);
  border-radius: var(--bs-border-radius, 0.625rem);
  padding: 1.25rem;
}
.chart-box h3, .chart-full h3 {
  font-size: 0.875rem;
  color: var(--label-color);
  margin: 0 0 1rem;
  border: none;
  padding: 0;
}
.chart-wrapper { position: relative; }

/* ===== P3: Emerging Keywords → Action ===== */
.emerging-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 0.5rem;
}
.emerging-chip {
  background: var(--card-header-bg);
  border: 1px solid var(--main-border-color);
  border-radius: 0.5rem;
  padding: 0.625rem 0.75rem;
  font-size: 0.8rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: default;
  transition: all 0.2s;
}
.emerging-chip:hover {
  border-color: var(--link-color);
}
.emerging-chip .name { font-weight: 500; color: var(--text-color); }
.emerging-chip .count {
  font-size: 0.7rem;
  color: var(--text-muted-color);
  background: rgba(255,255,255,0.06);
  padding: 0.1rem 0.4rem;
  border-radius: 0.25rem;
  margin-left: 0.5rem;
  white-space: nowrap;
}
.emerging-chip .sources-tag {
  font-size: 0.6rem;
  color: var(--text-muted-color);
  margin-left: 0.25rem;
}

/* Drilldown panel */
.drilldown-panel {
  display: none;
  background: var(--main-bg);
  border: 1px solid var(--main-border-color);
  border-radius: 0.5rem;
  padding: 0.875rem;
  margin-top: 0.75rem;
  font-size: 0.85rem;
}
.drilldown-panel.active { display: block; }
.drilldown-panel ul { margin: 0.5rem 0 0; padding-left: 1.2rem; }

/* ===== Posts Scroll Wrapper ===== */
.posts-scroll-wrap {
  max-height: 320px;
  overflow-y: auto;
  border-radius: 0.375rem;
  scrollbar-width: thin;
  scrollbar-color: var(--main-border-color) transparent;
}
.posts-scroll-wrap::-webkit-scrollbar { width: 6px; }
.posts-scroll-wrap::-webkit-scrollbar-track { background: transparent; }
.posts-scroll-wrap::-webkit-scrollbar-thumb {
  background: var(--main-border-color);
  border-radius: 3px;
}
.posts-scroll-wrap .posts-table thead th {
  position: sticky;
  top: 0;
  background: var(--card-header-bg);
  z-index: 1;
}

.posts-table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
.posts-table th {
  text-align: left;
}

/* ===== P1: Ops Panel (collapsible) ===== */
.ops-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  padding: 0.625rem 0;
  color: var(--text-muted-color);
  font-size: 0.85rem;
  user-select: none;
  border: none;
  background: none;
  width: 100%;
  text-align: left;
}
.ops-toggle:hover { color: var(--text-color); }
.ops-toggle .arrow {
  display: inline-block;
  transition: transform 0.3s;
  font-size: 0.7rem;
}
.ops-toggle.open .arrow { transform: rotate(90deg); }
.ops-content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.4s ease;
}
.ops-content.open { max-height: 600px; }

/* ===== Meta ===== */
.db-meta {
  font-size: 0.75rem;
  color: var(--text-muted-color);
  text-align: right;
}

@media (max-width: 768px) {
  .summary-cards { grid-template-columns: repeat(2, 1fr); }
  .chart-row { grid-template-columns: 1fr; }
  .summary-cell { max-width: 250px; }
}
</style>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

<div class="db-grid">
  <!-- P3: Pipeline Status -->
  <div class="pipeline-status" id="js-pipeline-status"></div>

  <!-- Summary Cards -->
  <div class="summary-cards" id="js-summary-cards"></div>

  <!-- P2: Filter Bar -->
  <div class="filter-bar" id="js-filter-bar"></div>

  <!-- P1: Posts Table (moved to TOP) -->
  <div class="chart-full">
    <h3>최근 발행 포스트</h3>
    <table class="posts-table">
      <thead>
        <tr><th style="width:70px">날짜</th><th style="width:90px">키워드</th><th style="width:50px">품질</th><th>포스트</th></tr>
      </thead>
      <tbody id="js-posts-tbody"></tbody>
    </table>
  </div>

  <!-- Mentions Chart -->
  <div class="chart-full">
    <h3>키워드 멘션 추이 (최근 30일)</h3>
    <div class="chart-wrapper" style="height:200px">
      <canvas id="js-mentions-chart"></canvas>
    </div>
  </div>

  <!-- Weekly Rank + Emerging Keywords -->
  <div class="chart-row">
    <div class="chart-box">
      <h3>이번 주 키워드 수집 현황</h3>
      <div class="chart-wrapper" id="js-weekly-rank-wrapper">
        <canvas id="js-weekly-rank-chart"></canvas>
      </div>
    </div>
    <div class="chart-box">
      <h3>새로 뜨는 키워드 (최근 7일)</h3>
      <div class="emerging-grid" id="js-emerging-grid"></div>
      <div id="js-drilldown" class="drilldown-panel">
        <strong id="js-drilldown-title"></strong>
        <ul id="js-drilldown-list"></ul>
      </div>
    </div>
  </div>

  <!-- P1: Ops Panel (collapsible — quality + cost charts inside) -->
  <div>
    <button class="ops-toggle" id="js-ops-toggle">
      <span class="arrow">&#9654;</span>
      운영 상세 (품질 트렌드, AI 비용, 수집 통계)
    </button>
    <div class="ops-content" id="js-ops-content">
      <div class="chart-row" style="margin-top:0.75rem">
        <div class="chart-box">
          <h3>블로그 품질 트렌드 (최근 60일)</h3>
          <div class="chart-wrapper" style="height:240px">
            <canvas id="js-quality-chart"></canvas>
          </div>
        </div>
        <div class="chart-box">
          <h3>월별 AI 비용 (USD, Haiku 4.5)</h3>
          <div class="chart-wrapper" style="height:240px">
            <canvas id="js-cost-chart"></canvas>
          </div>
        </div>
      </div>
    </div>

  <div class="chart-full">
    <h3 style="display:flex;justify-content:space-between;align-items:center">
      <span>최근 발행 포스트 <small id="js-posts-count" style="font-size:0.75rem;font-weight:400;color:var(--text-muted-color)"></small></span>
      <a href="/" style="font-size:0.78rem;color:var(--link-color);font-weight:400;text-decoration:none">→ 전체 보기</a>
    </h3>
    <div class="posts-scroll-wrap">
      <table class="posts-table">
        <thead>
          <tr><th>날짜</th><th>키워드</th><th>품질</th><th>포스트</th></tr>
        </thead>
        <tbody id="js-posts-tbody"></tbody>
      </table>
    </div>
  </div>

  <p class="db-meta" id="js-meta"></p>
</div>

<script>
(function () {
  const DATA = {{ site.data.dashboard | jsonify }};
  if (!DATA) {
    document.querySelector('.db-grid').innerHTML =
      '<p style="color:var(--text-muted-color)">dashboard.json 데이터를 찾을 수 없습니다.<\/p>';
    return;
  }

  /* Chirpy CSS 변수 읽기 */
  const cs  = getComputedStyle(document.documentElement);
  const get = (v) => cs.getPropertyValue(v).trim();
  const C = {
    text:   get('--text-color')        || '#afafb1',
    muted:  get('--text-muted-color')  || '#868686',
    border: get('--main-border-color') || '#2c2d2d',
    link:   get('--link-color')        || '#8ab4f8',
  };
  const PAL = ['#8ab4f8','#f28b82','#81c995','#fdd663','#c58af9','#ff8bcb','#78d9ec','#ff9a45'];

  Chart.defaults.color       = C.text;
  Chart.defaults.borderColor = C.border;
  Chart.defaults.layout.padding = 10;

  function xAxis(extra) {
    return Object.assign({ ticks: { color: C.muted, maxRotation: 0 }, grid: { color: C.border } }, extra);
  }
  function yAxis(extra) {
    return Object.assign({ ticks: { color: C.muted }, grid: { color: C.border } }, extra);
  }
  function barHeight(count) {
    return Math.min(350, Math.max(120, count * 40 + 60));
  }

  /* 키워드→카테고리 매핑 */
  const KW_CATEGORIES = {
    'claude': 'AI', 'LLM': 'AI', 'llm': 'AI',
    'kubernetes': 'Infra', 'docker': 'Infra', 'k8s': 'Infra',
  };
  function kwCategory(kw) { return KW_CATEGORIES[kw] || 'etc'; }
  function kwCssClass(kw) {
    const cat = kwCategory(kw);
    if (cat === 'AI') return 'kw-ai';
    if (cat === 'Infra') return 'kw-infra';
    return '';
  }

  /* ============================================================
   * P3: Pipeline Status
   * ============================================================ */
  (function renderPipelineStatus() {
    const el = document.getElementById('js-pipeline-status');
    const posts = DATA.recent_posts || [];
    const genAt = DATA.generated_at || '';

    // 오늘/어제 날짜로 최근 실행 상태 추정
    const today = new Date().toISOString().slice(0, 10);
    const todayPosts = posts.filter(p => p.date === today);
    const allKeywords = [...new Set(posts.map(p => p.keyword))];
    const lowQuality = posts.filter(p => p.quality_score != null && p.quality_score < 5.0).length;

    let statusClass = 'ok';
    let statusText = '파이프라인 정상';
    let detailParts = [];

    if (todayPosts.length === 0) {
      statusClass = 'warn';
      statusText = '오늘 발행된 포스트 없음';
    }
    if (lowQuality > 0) {
      detailParts.push(`품질 5.0 미만: ${lowQuality}건`);
    }
    detailParts.push(`키워드 ${allKeywords.length}개 추적 중`);

    el.innerHTML = `
      <span class="status-dot ${statusClass}"><\/span>
      <span>${statusText} — 마지막 갱신: ${genAt} UTC<\/span>
      <span class="status-detail">${detailParts.join(' · ')}<\/span>
    `;
  })();

  /* ============================================================
   * Summary Cards (with delta)
   * ============================================================ */
  (function renderSummary() {
    const s = DATA.summary || {};
    const posts = DATA.recent_posts || [];

    // 이번 주 발행 수 계산
    const today = new Date();
    const weekAgo = new Date(today); weekAgo.setDate(today.getDate() - 7);
    const weekAgoStr = weekAgo.toISOString().slice(0, 10);
    const twoWeeksAgoStr = new Date(today.getTime() - 14*86400000).toISOString().slice(0, 10);
    const thisWeek = posts.filter(p => p.date >= weekAgoStr).length;
    const prevWeek = posts.filter(p => p.date >= twoWeeksAgoStr && p.date < weekAgoStr).length;
    const weekDelta = thisWeek - prevWeek;

    // 키워드별 카테고리 그룹
    const kwSet = [...new Set(posts.map(p => p.keyword))];
    const catGroups = {};
    kwSet.forEach(k => {
      const cat = kwCategory(k);
      catGroups[cat] = (catGroups[cat] || 0) + 1;
    });
    const catDesc = Object.entries(catGroups).map(([c, n]) => `${c}(${n})`).join(' · ');

    const cards = [
      { label: '이번 주 발행', value: thisWeek,
        delta: weekDelta >= 0 ? `+${weekDelta} vs 지난주` : `${weekDelta} vs 지난주`,
        cls: weekDelta >= 0 ? 'delta-up' : 'delta-down' },
      { label: '추적 키워드', value: s.total_keywords ?? '-',
        delta: catDesc, cls: 'delta-neutral' },
      { label: '평균 품질 점수', value: s.avg_quality_score != null ? s.avg_quality_score.toFixed(1) : '-',
        delta: '', cls: 'delta-neutral' },
      { label: '이달 AI 비용', value: s.monthly_cost_usd != null ? `$${s.monthly_cost_usd.toFixed(3)}` : '-',
        delta: '', cls: 'delta-neutral' },
    ];
    document.getElementById('js-summary-cards').innerHTML = cards.map(c => `
      <div class="summary-card">
        <div class="summary-card__label">${c.label}<\/div>
        <div class="summary-card__value">${c.value}<\/div>
        ${c.delta ? `<div class="summary-card__delta ${c.cls}">${c.delta}<\/div>` : ''}
      <\/div>`).join('');
  })();

  /* ============================================================
   * P2: Filter Bar
   * ============================================================ */
  let activeRange = 30;   // days
  let activeKeywords = null; // null = all

  (function renderFilterBar() {
    const posts = DATA.recent_posts || [];
    const allKw = [...new Set(posts.map(p => p.keyword))];

    // 카테고리별 그룹핑
    const cats = {};
    allKw.forEach(kw => {
      const cat = kwCategory(kw);
      if (!cats[cat]) cats[cat] = [];
      cats[cat].push(kw);
    });

    const bar = document.getElementById('js-filter-bar');
    let html = '';
    // 기간 필터
    [7, 30, 60].forEach(d => {
      html += `<button class="filter-btn${d === 30 ? ' active' : ''}" data-range="${d}" onclick="window.__dbFilter(${d}, null)">${d}일<\/button>`;
    });
    html += '<div class="filter-sep"><\/div>';
    // 카테고리 필터
    Object.entries(cats).forEach(([cat, kws]) => {
      const color = cat === 'AI' ? '#8ab4f8' : cat === 'Infra' ? '#81c995' : '#fdd663';
      html += `<button class="kw-filter-btn active" data-cat="${cat}" onclick="window.__dbFilterKw(this, '${cat}')">
        <span class="dot" style="background:${color}"><\/span>
        ${cat} <span style="font-size:0.65rem;color:var(--text-muted-color)">(${kws.join(', ')})<\/span>
      <\/button>`;
    });
    bar.innerHTML = html;
  })();

  // 활성 카테고리 추적
  const activeCats = new Set(Object.keys({AI:1, Infra:1, etc:1}));

  window.__dbFilter = function(range) {
    activeRange = range;
    document.querySelectorAll('.filter-btn').forEach(b => {
      b.classList.toggle('active', +b.dataset.range === range);
    });
    renderPosts();
    renderMentionsChart();
  };
  window.__dbFilterKw = function(btn, cat) {
    btn.classList.toggle('active');
    if (activeCats.has(cat)) activeCats.delete(cat);
    else activeCats.add(cat);
    renderPosts();
    renderMentionsChart();
  };

  /* ============================================================
   * P1: Posts Table (top, with quality badges + truncation)
   * ============================================================ */
  function renderPosts() {
    const posts = DATA.recent_posts || [];
    const cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - activeRange);
    const cutoffStr = cutoff.toISOString().slice(0, 10);

    const filtered = posts.filter(p => {
      if (p.date < cutoffStr) return false;
      const cat = kwCategory(p.keyword);
      return activeCats.has(cat);
    });

    const tbody = document.getElementById('js-posts-tbody');
    if (!filtered.length) {
      tbody.innerHTML = '<tr><td colspan="4" style="text-align:center;color:var(--text-muted-color)">해당 기간에 데이터 없음<\/td><\/tr>';
      return;
    }

    tbody.innerHTML = filtered.map(p => {
      const q = p.quality_score;
      let qClass = 'quality-mid';
      let warn = '';
      if (q != null) {
        if (q >= 7.0) qClass = 'quality-high';
        else if (q < 5.0) { qClass = 'quality-low'; warn = ' ⚠'; }
      }
      const badge = q != null
        ? `<span class="quality-badge ${qClass}">${q.toFixed(1)}${warn}<\/span>`
        : '-';

      const cat = kwCategory(p.keyword);
      const kwClass = kwCssClass(p.keyword);
      const kwLabel = cat !== 'etc' ? `${cat} · ${p.keyword}` : p.keyword;

      const link = p.post_url
        ? `<a href="${p.post_url}" target="_blank" rel="noopener" style="color:var(--link-color)" class="summary-cell">${p.one_line_summary || '보기'}<\/a>`
        : `<span class="summary-cell">${p.one_line_summary || '-'}<\/span>`;

      return `<tr>
        <td style="white-space:nowrap">${p.date.slice(5)}<\/td>
        <td><span class="kw-badge ${kwClass}">${kwLabel}<\/span><\/td>
        <td>${badge}<\/td>
        <td>${link}<\/td>
      <\/tr>`;
    }).join('');
  }
  renderPosts();

  /* ============================================================
   * Mentions Chart (filtered by range & keywords)
   * ============================================================ */
  let mentionsChartInstance = null;
  function renderMentionsChart() {
    const trendSeries = DATA.tracked_trend?.series ?? [];
    if (!trendSeries.length) return;

    const cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - activeRange);
    const cutoffStr = cutoff.toISOString().slice(0, 10);

    const filteredSeries = trendSeries.filter(s => activeCats.has(kwCategory(s.topic)));

    const allDates = [...new Set(
      filteredSeries.flatMap(s => s.data_points.filter(p => p.date >= cutoffStr).map(p => p.date))
    )].sort();

    if (mentionsChartInstance) mentionsChartInstance.destroy();

    mentionsChartInstance = new Chart(document.getElementById('js-mentions-chart'), {
      type: 'line',
      data: {
        labels: allDates,
        datasets: filteredSeries.map((s, i) => {
          const m = Object.fromEntries(s.data_points.map(p => [p.date, p.mentions]));
          return {
            label: s.topic,
            data: allDates.map(d => m[d] ?? null),
            borderColor: PAL[i % PAL.length],
            backgroundColor: PAL[i % PAL.length] + '18',
            fill: true,
            pointRadius: 3,
            pointHoverRadius: 5,
            tension: 0.4,
            spanGaps: false,
          };
        }),
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: yAxis({ beginAtZero: true }),
          x: xAxis({ ticks: { color: C.muted, maxTicksLimit: 10, maxRotation: 0 } }),
        },
        plugins: { legend: { labels: { color: C.text, boxWidth: 12 } } },
      },
    });
  }
  renderMentionsChart();

  /* ============================================================
   * Weekly Rank Horizontal Bar
   * ============================================================ */
  (function() {
    const weeklyRank = DATA.tracked_trend?.weekly_rank ?? [];
    if (!weeklyRank.length) return;
    const h = barHeight(weeklyRank.length);
    document.getElementById('js-weekly-rank-wrapper').style.height = h + 'px';
    new Chart(document.getElementById('js-weekly-rank-chart'), {
      type: 'bar',
      data: {
        labels: weeklyRank.map(r => r.topic),
        datasets: [{
          label: '이번 주 멘션',
          data: weeklyRank.map(r => r.mentions),
          backgroundColor: weeklyRank.map((_, i) => PAL[i % PAL.length] + 'bb'),
          borderColor: weeklyRank.map((_, i) => PAL[i % PAL.length]),
          borderWidth: 1, borderRadius: 3,
        }],
      },
      options: {
        indexAxis: 'y',
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false },
          tooltip: { callbacks: { label: ctx => ` ${ctx.parsed.x}건` } },
        },
        scales: {
          x: xAxis({ beginAtZero: true }),
          y: yAxis({ ticks: { color: C.text, font: { size: 12 } } }),
        },
      },
    });
  })();

  /* ============================================================
   * P3: Emerging Keywords as chips with drilldown
   * ============================================================ */
  (function() {
    const emerging = DATA.emerging_topics?.topics ?? [];
    const grid = document.getElementById('js-emerging-grid');
    if (!emerging.length) {
      grid.innerHTML = '<span style="color:var(--text-muted-color);font-size:0.8rem">데이터 없음<\/span>';
      return;
    }
    grid.innerHTML = emerging.slice(0, 12).map((t, i) => `
      <div class="emerging-chip" data-idx="${i}">
        <span class="name">${t.topic}<\/span>
        <span class="count">멘션 ${t.weekly_mentions}<\/span>
      <\/div>
    `).join('');

    // Click drilldown
    grid.querySelectorAll('.emerging-chip').forEach(chip => {
      chip.addEventListener('click', () => {
        const idx = +chip.dataset.idx;
        const topic = emerging[idx];
        const panel = document.getElementById('js-drilldown');
        const titleEl = document.getElementById('js-drilldown-title');
        const listEl = document.getElementById('js-drilldown-list');

        if (panel.classList.contains('active') && panel.dataset.idx === String(idx)) {
          panel.classList.remove('active');
          return;
        }
        panel.dataset.idx = idx;
        titleEl.textContent = `${topic.topic}  (${(topic.sources || []).join(', ')})`;
        listEl.innerHTML = '';
        const titles = topic.sample_titles?.length ? topic.sample_titles : ['샘플 없음'];
        titles.forEach(t => {
          const li = document.createElement('li');
          li.textContent = t;
          listEl.appendChild(li);
        });
        panel.classList.add('active');
      });
    });
  })();

  /* ============================================================
   * Quality Trend + Cost Chart (inside ops panel)
   * ============================================================ */
  // Quality
  (function() {
    const qualSeries = DATA.knowledge_trend?.series ?? [];
    if (!qualSeries.length) return;
    const allQDates = [...new Set(
      qualSeries.flatMap(s => s.data_points.map(p => p.date))
    )].sort();
    new Chart(document.getElementById('js-quality-chart'), {
      type: 'line',
      data: {
        labels: allQDates,
        datasets: [
          // 품질 임계값 선 (5.0)
          {
            label: '임계값 5.0',
            data: allQDates.map(() => 5.0),
            borderColor: '#f28b82',
            borderDash: [6, 3],
            borderWidth: 1,
            pointRadius: 0,
            fill: false,
          },
          ...qualSeries.map((s, i) => {
            const m = Object.fromEntries(s.data_points.map(p => [p.date, p.quality_score]));
            return {
              label: s.keyword,
              data: allQDates.map(d => m[d] ?? null),
              borderColor: PAL[i % PAL.length],
              backgroundColor: 'transparent',
              pointRadius: 3,
              pointHoverRadius: 5,
              tension: 0.4,
              spanGaps: false,
            };
          }),
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: yAxis({ min: 0, max: 10, ticks: { stepSize: 2, color: C.muted } }),
          x: xAxis({ ticks: { color: C.muted, maxTicksLimit: 8, maxRotation: 0 } }),
        },
        plugins: { legend: { labels: { color: C.text, boxWidth: 12, font: { size: 11 } } } },
      },
    });
  })();

  // Cost
  (function() {
    const costData = DATA.monthly_cost ?? [];
    if (!costData.length) return;
    new Chart(document.getElementById('js-cost-chart'), {
      type: 'bar',
      data: {
        labels: costData.map(c => c.month),
        datasets: [
          {
            label: 'Input 비용',
            data: costData.map(c => +(c.input_tokens * 0.80 / 1e6).toFixed(4)),
            backgroundColor: PAL[0] + 'bb',
            stack: 'cost',
            borderRadius: { topLeft: 0, topRight: 0, bottomLeft: 3, bottomRight: 3 },
          },
          {
            label: 'Output 비용',
            data: costData.map(c => +(c.output_tokens * 4.00 / 1e6).toFixed(4)),
            backgroundColor: PAL[1] + 'bb',
            stack: 'cost',
            borderRadius: { topLeft: 3, topRight: 3, bottomLeft: 0, bottomRight: 0 },
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          x: xAxis({ stacked: true }),
          y: yAxis({ stacked: true, ticks: { color: C.muted, callback: v => `$${v.toFixed(3)}` } }),
        },
        plugins: {
          legend: { labels: { color: C.text, boxWidth: 12 } },
          tooltip: {
            callbacks: {
              footer: items => `합계: $${items.reduce((a, b) => a + b.parsed.y, 0).toFixed(4)}`,
            },
          },
        },
      },
    });
  })();

  /* ============================================================
   * P1: Ops Toggle
   * ============================================================ */
  document.getElementById('js-ops-toggle').addEventListener('click', function() {
    this.classList.toggle('open');
    document.getElementById('js-ops-content').classList.toggle('open');
  });

  /* ── 7. 최근 발행 포스트 테이블 ─────────────────────────── */
  const posts  = DATA.recent_posts ?? [];
  const tbody  = document.getElementById('js-posts-tbody');
  const MAX_POSTS = 20;
  const displayed = posts.slice(0, MAX_POSTS);
  const countEl = document.getElementById('js-posts-count');
  if (countEl) {
    countEl.textContent = posts.length > MAX_POSTS
      ? `(최신 ${MAX_POSTS}개 / 총 ${posts.length}개)`
      : `(${posts.length}개)`;
  }
  if (displayed.length) {
    tbody.innerHTML = displayed.map(p => {
      const q     = p.quality_score;
      const color = q >= 8 ? '#81c995' : q >= 6 ? '#fdd663' : '#f28b82';
      const badge = q != null
        ? `<span class="quality-badge" style="background:${color}22;color:${color}">${q.toFixed(1)}</span>`
        : '-';
      const link  = p.post_url
        ? `<a href="${p.post_url}" target="_blank" rel="noopener" style="color:var(--link-color)">${p.one_line_summary || '보기'}</a>`
        : (p.one_line_summary || '-');
      return `<tr>
        <td style="white-space:nowrap">${p.date}</td>
        <td>${p.keyword}</td>
        <td>${badge}</td>
        <td>${link}</td>
      </tr>`;
    }).join('');
  } else {
    tbody.innerHTML = '<tr><td colspan="4" style="text-align:center;color:var(--text-muted-color)">데이터 없음</td></tr>';
  }

  /* Meta */
  if (DATA.generated_at) {
    document.getElementById('js-meta').textContent = `마지막 갱신: ${DATA.generated_at} UTC`;
  }
})();
</script>