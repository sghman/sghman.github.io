---
layout: page
title: 기술 트렌드 대시보드
permalink: /dashboard/
icon: fas fa-chart-line
order: 5
---

<style>
.db-grid { display: grid; gap: 1.5rem; }

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
}
.posts-table tr:last-child td { border-bottom: none; }

.quality-badge {
  display: inline-block;
  padding: 0.1rem 0.5rem;
  border-radius: 1rem;
  font-size: 0.75rem;
  font-weight: 600;
}

.db-meta {
  font-size: 0.75rem;
  color: var(--text-muted-color);
  text-align: right;
}

@media (max-width: 768px) {
  .summary-cards { grid-template-columns: repeat(2, 1fr); }
  .chart-row { grid-template-columns: 1fr; }
}
</style>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

<div class="db-grid">
  <div class="summary-cards" id="js-summary-cards"></div>

  <div class="chart-row">
    <div class="chart-box">
      <h3>내 키워드 주간 순위 변동 (Bump Chart)</h3>
      <canvas id="js-bump-chart" height="260"></canvas>
    </div>
    <div class="chart-box">
      <h3>급상승 미지 키워드 (최근 7일)</h3>
      <canvas id="js-emerging-bar" height="260"></canvas>
      <div id="js-drilldown" class="drilldown-panel">
        <strong id="js-drilldown-title"></strong>
        <ul id="js-drilldown-list"></ul>
      </div>
    </div>
  </div>

  <div class="chart-full">
    <h3>내 키워드 멘션 추이 (최근 30일)</h3>
    <canvas id="js-mentions-chart" height="140"></canvas>
  </div>

  <div class="chart-row">
    <div class="chart-box">
      <h3>블로그 품질 트렌드 (최근 60일)</h3>
      <canvas id="js-quality-chart" height="200"></canvas>
    </div>
    <div class="chart-box">
      <h3>월별 AI 비용 (USD, Haiku 4.5)</h3>
      <canvas id="js-cost-chart" height="200"></canvas>
    </div>
  </div>

  <div class="chart-full">
    <h3>최근 발행 포스트</h3>
    <table class="posts-table">
      <thead>
        <tr><th>날짜</th><th>키워드</th><th>품질</th><th>포스트</th></tr>
      </thead>
      <tbody id="js-posts-tbody"></tbody>
    </table>
  </div>

  <p class="db-meta" id="js-meta"></p>
</div>

<script>
(function () {
  /* ── 데이터 주입 (Jekyll Liquid) ───────────────────────────── */
  const DATA = {{ site.data.dashboard | jsonify }};
  if (!DATA) {
    document.querySelector('.db-grid').innerHTML =
      '<p style="color:var(--text-muted-color)">dashboard.json 데이터를 찾을 수 없습니다.</p>';
    return;
  }

  /* ── Chirpy 다크모드 CSS 변수 읽기 ────────────────────────── */
  const cs  = getComputedStyle(document.documentElement);
  const get = (v) => cs.getPropertyValue(v).trim();
  const C = {
    text:   get('--text-color')        || '#afafb1',
    muted:  get('--text-muted-color')  || '#868686',
    border: get('--main-border-color') || '#2c2d2d',
    link:   get('--link-color')        || '#8ab4f8',
  };

  /* 8색 팔레트 */
  const PAL = [
    '#8ab4f8','#f28b82','#81c995','#fdd663',
    '#c58af9','#ff8bcb','#78d9ec','#ff9a45',
  ];

  /* Chart.js 전역 기본값 */
  Chart.defaults.color       = C.text;
  Chart.defaults.borderColor = C.border;

  /* ── 공통 축 옵션 생성 헬퍼 ──────────────────────────────── */
  function xAxis(extra) {
    return Object.assign({ ticks: { color: C.muted, maxRotation: 0 }, grid: { color: C.border } }, extra);
  }
  function yAxis(extra) {
    return Object.assign({ ticks: { color: C.muted }, grid: { color: C.border } }, extra);
  }

  /* ── 1. 요약 카드 ─────────────────────────────────────────── */
  const s = DATA.summary || {};
  const cards = [
    { label: '총 발행 포스트',   value: s.total_posts      ?? '-' },
    { label: '추적 키워드',      value: s.total_keywords   ?? '-' },
    { label: '평균 품질 점수',   value: s.avg_quality_score != null ? s.avg_quality_score.toFixed(1) : '-' },
    { label: '이달 AI 비용',     value: s.monthly_cost_usd  != null ? `$${s.monthly_cost_usd.toFixed(3)}` : '-' },
  ];
  document.getElementById('js-summary-cards').innerHTML = cards.map(c => `
    <div class="summary-card">
      <div class="summary-card__label">${c.label}</div>
      <div class="summary-card__value">${c.value}</div>
    </div>`).join('');

  /* ── 2. Bump Chart (y축 reverse Line Chart) ──────────────── */
  const bump = DATA.tracked_trend?.bump_chart;
  if (bump?.weeks?.length && bump?.series?.length) {
    new Chart(document.getElementById('js-bump-chart'), {
      type: 'line',
      data: {
        labels: bump.weeks,
        datasets: bump.series.map((s, i) => ({
          label:           s.topic,
          data:            s.ranks,
          borderColor:     PAL[i % PAL.length],
          backgroundColor: PAL[i % PAL.length] + '22',
          pointRadius:     6,
          pointHoverRadius:8,
          tension:         0.3,
        })),
      },
      options: {
        responsive: true,
        scales: {
          y: yAxis({
            reverse: true,
            min: 1,
            max: bump.series.length,
            ticks: { stepSize: 1, callback: v => `#${v}`, color: C.muted },
          }),
          x: xAxis(),
        },
        plugins: {
          legend: { labels: { color: C.text, boxWidth: 12 } },
          tooltip: { callbacks: { label: ctx => ` ${ctx.dataset.label}: #${ctx.parsed.y}위` } },
        },
      },
    });
  }

  /* ── 3. 급상승 미지 키워드 Horizontal Bar + 드릴다운 ─────── */
  const emerging = DATA.emerging_topics?.topics ?? [];
  let activeDrilldownIdx = -1;  /* 드릴다운 상태 관리 */

  if (emerging.length) {
    const topN = emerging.slice(0, 15);
    new Chart(document.getElementById('js-emerging-bar'), {
      type: 'bar',
      data: {
        labels: topN.map(t => t.topic),
        datasets: [{
          label: '주간 멘션',
          data:  topN.map(t => t.weekly_mentions),
          backgroundColor: topN.map((_, i) => PAL[i % PAL.length] + 'bb'),
          borderColor:     topN.map((_, i) => PAL[i % PAL.length]),
          borderWidth: 1,
        }],
      },
      options: {
        indexAxis: 'y',
        responsive: true,
        onClick: (_evt, elements) => {
          if (!elements.length) return;
          const idx   = elements[0].index;
          const topic = topN[idx];
          const panel = document.getElementById('js-drilldown');
          const titleEl = document.getElementById('js-drilldown-title');
          const listEl  = document.getElementById('js-drilldown-list');

          /* 같은 항목 다시 클릭 → 닫기 */
          if (activeDrilldownIdx === idx && panel.classList.contains('active')) {
            panel.classList.remove('active');
            activeDrilldownIdx = -1;
            return;
          }

          activeDrilldownIdx = idx;
          titleEl.textContent = `${topic.topic}  (${(topic.sources || []).join(', ')})`;

          /* XSS 방지: textContent 사용 */
          listEl.innerHTML = '';
          const titles = topic.sample_titles?.length ? topic.sample_titles : ['샘플 없음'];
          titles.forEach(t => {
            const li = document.createElement('li');
            li.textContent = t;
            listEl.appendChild(li);
          });

          panel.classList.add('active');
        },
        plugins: {
          legend: { display: false },
          tooltip: {
            callbacks: {
              afterLabel: ctx => `소스: ${(topN[ctx.dataIndex].sources || []).join(', ')}`,
            },
          },
        },
        scales: {
          x: xAxis({ beginAtZero: true }),
          y: yAxis({ ticks: { color: C.text, font: { size: 11 } } }),
        },
      },
    });
  }

  /* ── 4. Multi-line 멘션 추이 (30일) ─────────────────────── */
  const trendSeries = DATA.tracked_trend?.series ?? [];
  if (trendSeries.length) {
    const allDates = [...new Set(
      trendSeries.flatMap(s => s.data_points.map(p => p.date))
    )].sort();
    new Chart(document.getElementById('js-mentions-chart'), {
      type: 'line',
      data: {
        labels: allDates,
        datasets: trendSeries.map((s, i) => {
          const m = Object.fromEntries(s.data_points.map(p => [p.date, p.mentions]));
          return {
            label:           s.topic,
            data:            allDates.map(d => m[d] ?? null),
            borderColor:     PAL[i % PAL.length],
            backgroundColor: 'transparent',
            pointRadius:     2,
            tension:         0.3,
            spanGaps:        false,
          };
        }),
      },
      options: {
        responsive: true,
        scales: {
          y: yAxis({ beginAtZero: true }),
          x: xAxis({ ticks: { color: C.muted, maxTicksLimit: 10, maxRotation: 0 } }),
        },
        plugins: { legend: { labels: { color: C.text, boxWidth: 12 } } },
      },
    });
  }

  /* ── 5. 품질 트렌드 Line Chart (60일) ───────────────────── */
  const qualSeries = DATA.knowledge_trend?.series ?? [];
  if (qualSeries.length) {
    const allQDates = [...new Set(
      qualSeries.flatMap(s => s.data_points.map(p => p.date))
    )].sort();
    new Chart(document.getElementById('js-quality-chart'), {
      type: 'line',
      data: {
        labels: allQDates,
        datasets: qualSeries.map((s, i) => {
          const m = Object.fromEntries(s.data_points.map(p => [p.date, p.quality_score]));
          return {
            label:           s.keyword,
            data:            allQDates.map(d => m[d] ?? null),
            borderColor:     PAL[i % PAL.length],
            backgroundColor: 'transparent',
            pointRadius:     3,
            tension:         0.3,
            spanGaps:        false,
          };
        }),
      },
      options: {
        responsive: true,
        scales: {
          y: yAxis({ min: 0, max: 10, ticks: { stepSize: 2, color: C.muted } }),
          x: xAxis({ ticks: { color: C.muted, maxTicksLimit: 8, maxRotation: 0 } }),
        },
        plugins: { legend: { labels: { color: C.text, boxWidth: 12, font: { size: 11 } } } },
      },
    });
  }

  /* ── 6. 월별 AI 비용 Stacked Bar ────────────────────────── */
  const costData = DATA.monthly_cost ?? [];
  if (costData.length) {
    new Chart(document.getElementById('js-cost-chart'), {
      type: 'bar',
      data: {
        labels: costData.map(c => c.month),
        datasets: [
          {
            label:           'Input 비용',
            data:            costData.map(c => +(c.input_tokens  * 0.80 / 1e6).toFixed(4)),
            backgroundColor: PAL[0] + 'bb',
            stack:           'cost',
          },
          {
            label:           'Output 비용',
            data:            costData.map(c => +(c.output_tokens * 4.00 / 1e6).toFixed(4)),
            backgroundColor: PAL[1] + 'bb',
            stack:           'cost',
          },
        ],
      },
      options: {
        responsive: true,
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
  }

  /* ── 7. 최근 발행 포스트 테이블 ─────────────────────────── */
  const posts  = DATA.recent_posts ?? [];
  const tbody  = document.getElementById('js-posts-tbody');
  if (posts.length) {
    tbody.innerHTML = posts.map(p => {
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

  /* ── 8. 메타 정보 ────────────────────────────────────────── */
  if (DATA.generated_at) {
    document.getElementById('js-meta').textContent = `마지막 갱신: ${DATA.generated_at} UTC`;
  }
})();
</script>
