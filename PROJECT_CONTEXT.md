# 프로젝트 컨텍스트 — 사장님신용대출 금리전략 대시보드

> 은행 PC로 옮긴 뒤, 작업을 이어가기 전에 이 문서를 먼저 읽어주세요.
> (작성일: 2026-06-11, 집 PC에서 작성)

## 1. 이 프로젝트는 무엇인가

`standalone.html` 단일 파일로 동작하는 React 대시보드입니다.
- 빌드 과정 없음 — 브라우저에서 importmap + Babel 인라인 변환으로 바로 실행 (`react`, `recharts`, `lucide-react`를 esm.sh CDN에서 로드)
- 그냥 `standalone.html`을 더블클릭해서 브라우저로 열면 됨
- git 저장소 아님 (버전 관리 안 됨 — 수정 전에 항상 백업 권장)

대상 업무: Corporate여신팀의 "사장님신용대출" 상품의 **전략등급별 금리 조정 시뮬레이션** + **금리전략 변경(3건) 시행 실적 분석**

## 2. 대시보드 구조 (탭 2개)

1. **금리 조정 시뮬레이터** (`Simulator` 컴포넌트)
   - 등급(1~7등급)별 bp 조정 슬라이더 → 잔액/건수/이자수익/공헌이익/영업이익 변화 시뮬레이션
   - "최적금리 찾기", "초기화" 버튼
   - 기준값 CSV 업로드 (`template_2_simbase.csv`)
   - 하단 "사장님신용대출 요약" 바차트, 간접효과(카드/예금/MAU) 카드

2. **시행 실적** (`ResultPage` 컴포넌트)
   - 3개 금리전략 변경 이벤트(`EVENTS` 배열, [standalone.html:115-119](standalone.html#L115-L119))를 탭으로 전환:
     - `ev331`: 3/31 본부조정금리변경
     - `ev423`: 4/23 본부조정금리변경
     - `ev529`: 5/29 토스/현카 우대금리 도입
   - 섹션: 주간 영업 지표 → 수익성 지표(고신용/중저신용 분리) → 퍼널 전환율

## 3. CSV 업로드 규칙 (★ 가장 헷갈리는 부분)

| 데이터 종류 | 이벤트별 분리 여부 | 필요 파일 수 | 비고 |
|---|---|---|---|
| 주간 실적 (`template_1_weekly_*.csv`) | **이벤트별로 따로** | 3개 (ev331/ev423/ev529) | 각 이벤트 탭에서 개별 업로드 |
| 퍼널 (`template_4_funnel_*.csv`) | **이벤트별로 따로** | 3개 (ev331/ev423/ev529) | 각 이벤트 탭에서 개별 업로드 |
| 수익성 월간 (`template_3_monthly_high/low.csv`) | **이벤트 구분 없음** (1~6월 전체 1세트) | 2개 (고신용/중저신용) | 헤더 상단 업로드 버튼, 전체 기간 공용 |
| 시뮬레이터 기준값 (`template_2_simbase.csv`) | 해당 없음 | 1개 | 시뮬레이터 탭 "기준값 CSV" |

현재 폴더에 있는 9개 템플릿 파일은 전부 **데모 데이터 그대로** 옮겨 놓은 것 — 실제 쿼리 결과 값으로 교체해서 업로드하면 됨.

CSV 헤더 형식:
- 주간: `week,phase,newLoan,netLoan,balance,count,newCust`
- 퍼널: `grade,platformInflow_before,platformInflow_after,preApprovalExec_before,preApprovalExec_after`
- 수익성 월간: `month,interest,contribution,operatingProfit,roa`
- 시뮬레이터 기준값: `grade,balance,count,newWk,netWk,churn,margin,legalCostRate,contractCreditRate,contractOpRate,creditGapRate,opGapRate,nonIntRate,elasticity`

## 4. 오늘(2026-06-11) `standalone.html`에 적용한 수정 사항

1. **요약 바차트에 "대출 건수(주)" 추가** ([standalone.html:386](standalone.html#L386))
   - 시뮬레이터 탭 "사장님신용대출 요약 — 현 상태 vs 조정 후" 차트에 추가됨

2. **바차트 막대 위에 값 라벨 표시** ([standalone.html:391-400, 507-512](standalone.html#L391))
   - 실무진 회의에서 "마우스오버 팝업이 불편하다"는 의견 → 툴팁(마우스오버)은 그대로 두고, 막대 위에 숫자를 직접 표시
   - 건수 항목은 정수, 나머지는 소수1자리로 자동 포맷 (`fmtSummaryLabel`)
   - `LabelList`를 recharts에서 추가 import

3. **COF → "자금원가율" 용어 변경** (화면 표시 텍스트만 변경, 내부 변수명 `cof`/`setCof`/`COF_DEFAULT`/`cofInline`/`cofInput`은 그대로 유지 — 화면에 안 보이므로)
   - 공헌이익 공식 텍스트, footer 등 ([standalone.html:432](standalone.html#L432), [standalone.html:813](standalone.html#L813))

4. **자금원가율 입력란에 "(사장님 신용대출 3개월 평균)" 라벨 추가** ([standalone.html:436-438](standalone.html#L436-L438))
   - 작은 글씨(10.5px, 회색)로 괄호 표기 → "(사장님 신용대출 3개월 평균) 자금원가율 1.7%" 형태로 한눈에 의미 파악 가능

## 5. 다음에 이어서 할 만한 작업 (아직 미정 / TODO 아님, 참고용)

- 9개 템플릿 CSV에 데모 데이터 대신 실제 쿼리 결과 채워서 업로드 테스트
- ADW 데이터 구축 완료 시, 수익성 지표(고신용/중저신용)를 실측치로 교체 (현재 footer/공지 문구에 "ADW 데이터 구축 중"이라고 명시되어 있음)
- 탄력성 계수(ε), 원가율 6종(legalCostRate 등)도 현재는 가정값 — 실측치 확보 시 `template_2_simbase.csv`로 교체 안내됨
