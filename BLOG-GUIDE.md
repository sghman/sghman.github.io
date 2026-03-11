# 블로그 관리 가이드

## 일상 운영 (자동)

파이프라인이 매일 자동으로:
- 기술 기사 수집 → AI 평가 → 포스트 발행
- 대시보드 데이터 갱신 (`_data/dashboard.json`)

**손 댈 것 없음.**

---

## 프로젝트 변동 시

### 프로젝트 추가

`_data/projects.yml`에 항목 추가:

```yaml
- name: 새 프로젝트
  desc: 한 줄 설명
  status: idea        # live | dev | plan | idea
  category: personal  # automation | personal | learning
  tech: [Python]
```

### 프로젝트 상태 변경

`_data/projects.yml`에서 해당 항목의 `status` 값만 수정.

### 프로젝트 중단/삭제

해당 항목을 삭제하거나 주석 처리 (`#`).

---

## 활동 기록 추가 (선택)

`_data/activity.yml`에 항목 추가:

```yaml
- date: "2026-04-01"
  project: 프로젝트명
  text: 간단한 한 줄 설명
  color: green  # green | accent | yellow
```

최근 5~10개만 유지 권장. 오래된 항목은 삭제.

---

## 자기소개 수정

`_tabs/about.md` 직접 편집.

- 상단 바이오 텍스트: HTML 안의 텍스트 수정
- "Building things that automate things" 변경: `_config.yml`의 `tagline` + about.md 내 문구

---

## 디자인 수정

| 변경 대상 | 파일 |
|---|---|
| 색상/폰트 | `_sass/variables-hook.scss` |
| 다크모드 색상 | `_sass/colors/typography-dark.scss` |
| 커스텀 CSS | `assets/css/jekyll-theme-chirpy.scss` 하단 |
| 랜딩 페이지 | `_layouts/landing.html` |
| 대시보드 | `_tabs/dashboard.md` |

---

## 로컬 확인

```bash
cd "E:\신규 개발\sghman.github.io"
bundle exec jekyll serve
# → http://localhost:4000/
```

## 배포

```bash
git add -A && git commit -m "update: 변경 내용" && git push origin main
# GitHub Pages가 자동 빌드/배포
```

---

## 파일 구조 요약

```
_data/
├── dashboard.json    ← 파이프라인 자동 생성 (수정 금지)
├── projects.yml      ← 프로젝트 목록 (About + Projects 공용)
├── activity.yml      ← 최근 활동 타임라인
├── authors.yml       ← 작성자 정보
└── contact.yml       ← 소셜 링크

_tabs/
├── posts.md          ← 포스트 목록 + 필터
├── categories.md     ← 카테고리 아카이브
├── tags.md           ← 태그 아카이브
├── archives.md       ← 날짜별 아카이브
├── about.md          ← 자기소개 + Now + 파이프라인
├── dashboard.md      ← 트렌드 대시보드
└── projects.md       ← 프로젝트 카드 + 타임라인

_layouts/
└── landing.html      ← 풀스크린 랜딩 (index.html용)
```
