# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

이 프로젝트는 `uv`로 의존성을 관리합니다 (Python >= 3.12).

```bash
# 의존성 설치
uv sync

# Jupyter 노트북 실행
uv run jupyter notebook
```

가상환경(`.venv/`)은 상위 디렉토리(`pro/.venv/`)에 위치합니다. Windows에서 활성화:
```
..\\.venv\\Scripts\\activate
```

---

## Project Goal

**[금융] 젠트리피케이션 위험도 분석 시스템 구축 및 상생 체계 제안**

성남시 원도심(수정구 41133 + 중원구 41131) 재개발 지역을 대상으로, 상권 붕괴·소상공인 이탈·임대료 상승·유동인구 변화를 종합해 **젠트리피케이션 위험을 조기 진단하고 정책 우선순위를 제시**하는 시스템입니다.

### 배경 및 필요성

- 성남시 수정·중원구는 1970~80년대 조성 원도심으로 재개발 사업이 활발하게 진행 중
- 대규모 이주(약 5만 가구)로 소비 기반 약화 → 상권 붕괴 → 투자 위축의 악순환
- 최근 5년간 외식업체 45.1% 폐업, 창업 대비 폐업 비율 96%
- 분당구(41135)는 비교 대조군 (신도심 vs 원도심 양극화 분석 목적)
- 미시적 빅데이터 기반 실증 분석이 부족한 기존 연구의 한계를 극복하는 것이 목표

### 분석 주제

1. **상권 취약성 사전 진단** — 업종별 폐업률, 빈 점포 증가율 → 상권 침체 지수 개발
2. **상권 변화 및 붕괴 패턴 분석** — 업종 구성 변화, 공실 증가 흐름, 상권 쇠퇴 초기 신호 탐지
3. **젠트리피케이션 위험도 분석** — 임대료 상승·상권 고급화·기존 상인 이탈 지표 정량화
4. **재개발 파급 효과 시뮬레이션** — 완료 구역 전후 데이터로 유동인구 감소 반경·임대료 전이 속도 정량화
5. **지원 정책 우선순위화** — 구역별 지원 필요도 점수 산출, 고·중·저 위험 등급 분류

### 위험도 예측 지표

| 지표 | 설명 | 예측 시점 |
|---|---|---|
| 매출액 | 업종별 매출액 변화 | 초기 |
| 임대료 | 면적당 임대료 변화 | 초기 |
| 건축물 거래량 | 지역 거래건수 | 초/중기 |
| 소형건축물 거래량 | 소형 건축물 거래건수 | 초기 |
| 수익추구형 소유자 매입 | 법인·외지인 매입량 | 초기 |
| 권리금 | 점포당 권리금 변화 | 중기 |
| 수익추구형 소유자 증가 | 법인·외지인 소유 비율 | 중기 |
| 점포의 변화 | 주거→상업 전환, 업종 획일화 | 중기 |

**위험 단계 분류**: 초기 / 주의 / 경계 / 위험 (4단계)

### 분석 파이프라인

```
1차 전처리 (각 팀원)
    ↓
2차 전처리 → 기준월 × 행정동 마스터 테이블
    ↓
핵심 지표 10개 내외 확정
    ↓
K-Means / DBSCAN  →  지역 유형화 (고위험형 / 변화진행형 / 안정형)
Isolation Forest  →  급격한 변화 지역 조기 탐지
    ↓
규칙 기반 위험 라벨 초안 생성
    ↓
XGBoost / Random Forest  →  위험도 예측 (초기 / 주의 / 경계 / 위험)
Ridge / Lasso              →  지표 변화량 예측 및 핵심 변수 선별
    ↓
SHAP  →  "왜 이 지역이 위험인가" 기여도 시각화
    ↓
Tableau 대시보드 (정책기관용 + 소상공인용)
```

### 머신러닝 기법 요약

| 기법 | 용도 |
|---|---|
| K-Means | 유사 특성 지역 군집화 (지역 유형 분류) |
| DBSCAN | 불규칙 군집·이상 지역 탐지 |
| Random Forest | 위험도 예측 (앙상블, 해석 용이) |
| XGBoost | 위험도 예측 (비선형 복합 관계 반영, 고성능) |
| Linear / Ridge / Lasso | 임대료·공실률 등 지표 변화량 예측, 다중공선성 처리 |
| Isolation Forest | 단기간 급격한 변화 지역 조기 경고 |
| SHAP | 변수 중요도 설명, 정책 투입 우선순위 근거 |

### 대시보드 사용자

- **정책(경제)기관용**: 재개발 가능성 높은 지역 사전 식별 + 상권 충격·소상공인 피해 정량화
- **소상공인용**: 본인 구역 재개발 가능성·상권 변화 흐름·업종별 위험 수준 직관적 확인

### 설계 시 주의사항

- **원도심 범위**: 수정구(41133) + 중원구(41131). 분당구(41135)는 비교 대조군
- **블록 단위 활용**: 가맹점 데이터에 `blk_cd`(5,442개) 존재 → 행정동보다 세밀한 분석 가능
- **매출구간 E 재검토**: EDA에서 제외했으나 대형 프랜차이즈 침투 지표로 활용 가능성 있음
- **상권 타입**: 직접 컬럼 없음 → `card_tpbuz_nm_1`(9개 대분류) 비율로 파생 필요

---

## Project Architecture

팀원별 디렉토리에서 각자 담당 데이터를 전처리하며, 모든 작업은 Jupyter 노트북(`.ipynb`)으로 진행합니다. `main.py`는 플레이스홀더입니다.

```
final_project/
├── Data/                      # 2차 전처리 완료 통합 데이터 (공용)
│   ├── 2차전처리_공시지가데이터.csv
│   ├── 2차전처리_대민개방데이터.csv
│   ├── 2차전처리_버스데이터.csv
│   ├── 2차전처리_전입데이터.csv
│   ├── 2차전처리_전출데이터.csv
│   ├── 2차전처리_지하철데이터.csv
│   ├── transactions_total.csv
│   ├── 신용정보.csv
│   └── ...
├── Geunsu/                    # 가맹점·매출·공시지가 담당
├── Jiryun/                    # 통신 데이터 담당
├── eunbi/                     # 거래량·인구/기업·통신 Parquet 변환 담당
├── sungju/                    # 거래량·공시지가·교통·신용 담당
└── main.py                    # 플레이스홀더
```

**팀원별 담당 데이터:**
- `Geunsu/` — 가맹점(franchise), 매출(sales), 공시지가(land price). 전처리 결과 → `Geunsu/data/`
- `Jiryun/` — 통신 데이터(telecom, T20–T27 파일군). CSV → Parquet 변환 포함
- `eunbi/` — 거래량(real estate transactions), 인구/기업(population/company), 통신 Parquet 변환
- `sungju/` — 거래량, 공시지가, 교통(지하철/버스), 신용(credit)

**전처리 단계:**
- `1차 전처리` — 결측 처리, 이상치 탐지/제거, 데이터 타입 변환, 인코딩 정규화
- `2차 전처리` — 연도별 파일 병합, 행정동 코드 매핑, 파생 변수 생성, 통합 파일 출력

---

## Data Dictionary

총 **1,061개 데이터셋** 제공 (2023-01 ~ 2025-12, 성남시 기준)

| 데이터 (Korean) | 영문 | 담당 | 건수 | 형식 |
|---|---|---|---|---|
| 가맹점 / 매출 | Franchise & Sales | Geunsu | 72건 | CSV |
| 공시지가 | Official Land Price | Geunsu, sungju | — | CSV / Parquet |
| 통신 (T4–T27) | Telecom Movement | Jiryun | 756건 | CSV → Parquet |
| 인구 / 기업 | Population & Company | eunbi | 107건 | CSV |
| 신용 (전입·전출·전이·신용정보·이동통계) | Credit | sungju | 126건 | CSV / Parquet |
| 거래량 | Real Estate Transactions | eunbi, sungju | — | CSV |
| 교통 (지하철/버스) | Subway & Bus Ridership | sungju | — | CSV |

**데이터셋 상세 구성:**
- **기업**: 법인기업(cnt), 신규기업(new), 기업이전통계(nps_move_cnt)
- **신용**: 전입(IN_STAT), 전출(OUT_STAT), 전이(CHANGE), 전이경계(CHANGE_L), 신용정보(DM_STAT), 이동통계(WORK_STAT)
- **통신**: T4~T27 파일군 (시간·지역·좌표·사용자·이동정보·연령별 분포)
- **카드**: 가맹점정보(mer_s), 매출(day)

**비교 도시 데이터 존재**: 광명시·수원시·시흥시·안산시·안양시·용인시·포천시·하남시·화성시 (동일 형식)

**지역 코드 체계:**
- 시군구코드: 5자리 문자열 (예: `"41135"` = 분당구, `"41133"` = 수정구, `"41131"` = 중원구)
- 행정동코드: 8자리 문자열 (예: `"41135510"`) — 공통 조인 키
- 코드는 앞 자리 0 보존을 위해 반드시 **문자열**로 저장

---

## Common Coding Patterns

```python
# 한국어 CSV 읽기 (기본 인코딩)
df = pd.read_csv(path, encoding='cp949')  # 일부는 'utf-8-sig'

# 날짜 컬럼 (YYYYMM 형식)
df['date'] = pd.to_datetime(df['year_month'].astype(str), format='%Y%m')

# 지역 코드 문자열 변환
df['행정동코드'] = df['행정동코드'].astype(str).str.zfill(8)

# 한글 폰트 설정 (matplotlib)
plt.rcParams['font.family'] = 'Malgun Gothic'
plt.rcParams['axes.unicode_minus'] = False

# 대용량 Parquet 집계 (DuckDB 사용 권장)
import duckdb
result = duckdb.query("SELECT * FROM 'data/*.parquet'").df()

# 원도심 필터링 (분석 대상)
df_원도심 = df[df['cty_rgn_no'].isin(['41133', '41131'])]
# 비교 대조군 (분당구)
df_신도심 = df[df['cty_rgn_no'] == '41135']
```

---

## Known Data Quality Issues

- **가맹점**: 판매등급 컬럼 결측 약 30%, 신규/폐업 카운트가 총 가맹점 수를 초과하는 이상 레코드 소수 존재
- **버스 데이터**: 정류장 ID가 두 행정동에 중복 매핑되는 케이스, 연도별 정류장명 변경, 6만5천여 건 행정동 미매핑 → 수동 매핑 테이블 적용
- **통신 데이터 (t26)**: 연도별 컬럼 스키마 불일치, 컬럼명에 공백 포함 → `.str.strip()` 처리 필요
- **대용량 주의**: 통신 데이터는 연도당 1억+ 행 — 전체 병합 시 OOM 발생 가능. DuckDB 또는 분할 처리 사용

---

## Key Dependencies

현재 설치된 패키지 (`pyproject.toml` 기준):

- `pandas` — 데이터 조작
- `numpy`, `scipy` — 수치 연산
- `matplotlib`, `seaborn` — 시각화 (한글 폰트 설정 필요)
- `scikit-learn` — K-Means / DBSCAN / Isolation Forest / Ridge / Lasso / 전처리
- `requests` — HTTP 데이터 수집

모델링 단계 추가 필요:
- `xgboost` — 위험도 예측 모델
- `shap` — 변수 중요도 설명 (XGBoost / Random Forest 결과 해석)
- `duckdb` — 대용량 Parquet 집계
- `pyarrow` — Parquet 입출력 (ZSTD 압축)
