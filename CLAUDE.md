# CLAUDE.md — JOHN 신선식품 드롭셔핑 대시보드 작업 지침

---

## 1. 역할

이 레포에서 Claude의 역할:

- **대시보드 유지보수**: `index.html` 수정, 차트/KPI 개선, 버그 수정
- **비즈니스 분석 보조**: 마진 계산, 채널 성과 해석, SKU 우선순위 제안
- **문서화**: `business-context.md`, `PROJECT.md`, `AGENTS.md` 업데이트

John은 개발자가 아님. 마케팅/MD/커머스 운영 관점에서 판단하는 대표. 기술 설명은 최소화하고 비즈니스 임팩트 중심으로 커뮤니케이션.

---

## 2. 레포 구조

```
b2c-dashboard/
├── index.html            # 메인 대시보드 (바닐라 JS + Chart.js)
├── business-context.md   # 사업 맥락 및 의사결정 기준
├── CLAUDE.md             # Claude 작업 지침 (이 파일)
├── AGENTS.md             # Codex 협업 프로토콜
├── PROJECT.md            # 작업 현황 및 결정 로그
└── data/                 # 샘플 데이터 (테스트용 CSV/JSON)
```

---

## 3. 핵심 비즈니스 규칙

1. **마진 목표**: 추정 영업이익률 10~15% — `business-context.md` §6 기준
2. **마진 공식**: `판매가 = (공급가 + 택배비 + 마진) / (1 - 채널수수료율)` — `pricing_margin.md` 기준
3. **의학적 효능 표현 금지**: 신선식품/화장품 모두 해당 ("면역력 강화", "항산화" 등 효능 표현 사용 불가)
4. **구매 전환 추정 금지**: 실데이터 없이 전환율·구매 가능성 수치 제시 불가
5. **공급사 정보 보호**: 공급사 연락처, 계약 조건, 단가 정보는 코드에 하드코딩 금지

---

## 4. 대시보드 수정 시 규칙

### Google Sheets 데이터 형식 (변경 금지)
Apps Script에서 반환하는 배열 형식:
```javascript
[channel, date, product_name, qty, total]  // 5컬럼, 헤더 없음
```

### 마진 설정 기본값 변경 시
`index.html`의 마진 설정 패널 기본값을 바꿀 경우, `business-context.md` §6 테이블도 함께 업데이트.

대응 변수:
- `inputCostRatio` → `costRatio` (공급가 비율, 소수점: 0.78 = 78%)
- `inputFeeRate` → `channelFeeRate` (채널수수료율, 소수점)
- `inputDelivery` → `deliveryPerOrder` (택배비/건, 원 단위 정수)

### 차트 수정 시
- 기존 차트 destroy() 후 재생성 패턴 유지
- Chart.js 4.x API 사용 (3.x 문법 혼용 금지)
- 다크 테마 CSS 변수 사용: `var(--bg)`, `var(--card)`, `var(--text)`, `var(--text-muted)`, `var(--accent)`, `var(--green)`, `var(--orange)`, `var(--red)`

---

## 5. 금지 사항

- 공급사 연락처(이메일, 전화번호), 계약 단가, 마진 협상 내용을 코드나 주석에 포함 금지
- 개인정보(고객 이름, 주소, 연락처) 포함 금지
- `SHEET_JSON_URL` 실제 URL 외 다른 외부 API 엔드포인트 추가 시 John 확인 필수
- `business-context.md` 내 사업 전략 결정(어떤 SKU를 밀지 등)은 Claude가 단독 결정 금지 — 분석 제공 후 John이 결정

---

## 6. 커밋 전 체크리스트

- [ ] `SHEET_JSON_URL`이 실제 Apps Script URL로 설정되어 있는지 확인
- [ ] 마진 기본값(78%, 12%, 3500)이 `business-context.md`와 일치하는지 확인
- [ ] 콘솔 에러 없음 (브라우저에서 직접 확인)
- [ ] 모바일 뷰 확인 (900px 이하 레이아웃)
- [ ] 하드코딩된 개인정보/공급사 정보 없음

---

## 7. John의 포지셔닝

> John은 개발자가 아니다. MD(상품기획)/커머스 운영 리더로서 AI를 생산성 레버리지 도구로 활용한다.

- 기술 구현은 Claude/Codex에게 위임
- 비즈니스 판단(어떤 상품을 밀지, 어떤 채널에 집중할지)은 John이 결정
- 대시보드는 "5분 안에 보고 결정"할 수 있는 밀도로 유지
- 불필요한 기능 추가보다 **기존 기능의 정확도와 신뢰성**을 우선
