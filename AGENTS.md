# AGENTS.md — Codex 작업 지침

이 파일은 Codex(및 자율 코딩 에이전트)를 위한 작업 프로토콜이다.  
사람 리뷰어: John (대표). 기술 검토: Claude Code.

---

## 1. 역할

Codex가 담당하는 작업:

- `index.html` HTML/JS/CSS 구현 및 버그 수정
- 신규 차트 추가 (Chart.js 4.x)
- Google Sheets Apps Script 연동 로직 수정
- 데이터 파싱 로직 (`parseJSONtoData`) 최적화
- `data/` 폴더 샘플 데이터 생성/수정

---

## 2. 건드리지 말 것

다음 파일은 **Claude Code가 관리**한다. Codex가 직접 수정하지 말 것:

- `business-context.md` — 사업 맥락 및 의사결정 기준
- `CLAUDE.md` — Claude 작업 지침

`PROJECT.md`는 작업 완료 후 Handoff Log 항목만 추가 가능 (기존 항목 수정 금지).

---

## 3. 코드 표준

| 항목 | 기준 |
|------|------|
| 언어 | 바닐라 JavaScript (프레임워크 금지) |
| 차트 라이브러리 | Chart.js 4.x (CDN: `chart.umd.min.js`) |
| 폰트 | Noto Sans KR (Google Fonts) |
| 테마 | 다크 테마, CSS 변수 기반 |
| 외부 의존성 | Chart.js + Google Fonts 외 추가 금지 |
| 모듈 시스템 | 없음 (단일 HTML 파일, `<script>` 인라인) |

### CSS 변수 (변경 금지)

```css
--bg: #0a0e17
--card: #111827
--card-border: #1e293b
--text: #e2e8f0
--text-muted: #94a3b8
--accent: #3b82f6
--green: #10b981
--red: #ef4444
--orange: #f59e0b
```

---

## 4. Google Sheets 연동 형식

Apps Script Web App → JSON 배열 반환:

```javascript
// 반환 형식: 배열의 배열 (헤더 없음)
[
  ["스마트스토어", "2025-03-15", "딸기 2kg", 12, 384000],
  ["쿠팡",         "2025-03-15", "감귤 5kg",  8, 240000],
  // ...
]
// [채널, 날짜, 상품명(SKU), 수량, 매출(원)]
```

`parseJSONtoData(rows)` 함수가 이 배열을 파싱. 컬럼 순서 변경 시 이 함수도 함께 수정.

`SHEET_JSON_URL` 변수는 실제 Apps Script 배포 URL로 설정해야 함. 테스트 시 로컬 JSON 파일 또는 `data/sample.json`으로 대체 가능.

---

## 5. 마진 계산 변수

전역 변수 3개가 마진 추정에 사용됨:

```javascript
let costRatio = 0.78;        // 공급가 비율 (78%)
let channelFeeRate = 0.12;   // 채널수수료율 (12%)
let deliveryPerOrder = 3500; // 택배비/건 (₩3,500)
```

**반드시 `updateMarginDisplay()` 함수를 통해서만 업데이트할 것.**  
직접 변수 할당 금지 (UI 입력값과 동기화가 깨짐).

```javascript
// 올바른 방법
document.getElementById('inputCostRatio').value = 80;
updateMarginDisplay(); // 이 함수가 변수 업데이트 + KPI 재렌더

// 금지
costRatio = 0.80; // 직접 할당 금지
```

---

## 6. 차트 추가 패턴

신규 차트 추가 시 반드시 이 패턴을 따를 것:

```javascript
// 1. 전역 변수 선언 (다른 let 변수들 옆에)
let newChart;

// 2. 렌더 함수
function renderNewChart() {
  // 기존 차트 destroy (메모리 누수 방지)
  if (newChart) newChart.destroy();

  // 데이터 준비
  const labels = [...];
  const datasets = [...];

  // 차트 생성
  newChart = new Chart(
    document.getElementById('newChart').getContext('2d'),
    { type: 'bar', data: { labels, datasets }, options: chartOpts() }
  );
}

// 3. updateAll()에 추가
function updateAll() {
  renderKPIs(); updateCharts(); renderPieChart();
  renderSpikes(); renderYoyChart(); renderGauges();
  renderNewChart(); // 여기에 추가
}
```

---

## 7. PR 제출 시 확인 사항

PR을 올리기 전 반드시 체크:

- [ ] `SHEET_JSON_URL`이 실제 URL인지 또는 명시적 플레이스홀더인지 확인
- [ ] 마진 기본값 (78, 12, 3500) 변경 여부 확인 — 변경 시 `business-context.md`도 함께 업데이트 필요 (Claude 담당이므로 PR 본문에 명시)
- [ ] 콘솔 에러 없음
- [ ] `data/` 이외 경로에 개인정보/공급사 민감 정보 없음
- [ ] 신규 외부 라이브러리/CDN 추가 없음 (추가 시 John 승인 필요)
- [ ] PR 본문에 변경된 차트/기능 목록 명시
