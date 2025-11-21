# BOAZ-miniProject  
## Steam Games ETL Project (2025)

이 프로젝트는 **2025년 3월까지 수집된 Steam 게임 데이터**를 활용해  
**ETL 파이프라인 → 데이터 품질 검증(QA) → 구조화 테이블 생성 → SQL 분석**까지 수행한  
데이터 엔지니어링 미니 프로젝트입니다.

데이터 엔지니어링 입문자로서  
- 데이터를 구조화하고  
- 품질을 점검하며  
- 분석 가능한 형태로 변환하는 작업을  
직접 구현하는 것을 목표로 진행했습니다.

---

#  1. Project Overview

| 단계 | 주요 내용 |
|------|-----------|
| **Extract** | CSV 로컬 파일 로딩 (89,618 rows × 47 columns) |
| **Transform** | 날짜·리뷰·장르 등 핵심 컬럼 정제, 품질 검증, 수익 추정 |
| **Load** | CSV 저장 + SQLite DB 적재 |
| **Genre Normalization** | 리스트형 장르 explode → 별도 테이블 생성 |
| **QA (Data Quality Check)** | 논리적 무결성 검증(2가지 rule) 및 간단 시각화 |
| **SQL Analysis** | 연도별 평균 가격 변화 (LAG 윈도우 함수 사용) |

---

#  2. Extract

- Kaggle Steam dataset 로딩  
- 총 89,618개의 게임 레코드  
- dtype 검사 및 원본 컬럼 구조 파악
```python
df = pd.read_csv(RAW_PATH, encoding='utf-8')

#  3. Transform
✔ 핵심 14개 컬럼으로 정제

필요 컬럼만 선택하여 목적 중심의 구조 생성.

✔ 날짜(Date) 처리
release_date → datetime64  
release_year → int

✔ 긍정률 생성

positive_rate = pct_pos_total

✔ 수익 추정치 생성
est_revenue = price × num_reviews_total

# 4. Data Quality Check (QA)

ETL 신뢰성을 위해 2가지 논리적 검증 규칙 적용.

✔ Rule 1 — 리뷰 수 검증

positive + negative > num_reviews_total → 비정상 flag

✔ Rule 2 — 미래 출시일 검증

release_date > 오늘 날짜 → 데이터 오염 가능성 flag

출력 예:

invalid_review    12
future_release     3

# 5. Genre Normalization (explode)

Steam의 genres 는 "['Action','Adventure']" 형태 문자열이므로
바로 분석이 어렵다.
따라서 다음과 같은 정규화 수행:

✔ 문자열 → 리스트 변환 (ast.literal_eval)
✔ explode()로 appid–장르 매핑 테이블 생성
✔ Steam 공식 장르(valid_genres) 기준 필터링

생성 예시:

appid	genre
730	Action
570	Strategy
359550	Action
# 6. Load
✔ CSV 저장
df_main.to_csv(CLEAN_PATH, index=False)

✔ SQLite DB 저장

두 개의 테이블 구성:

테이블	설명
steam_games	정제된 게임 기본 정보
game_genres	appid–genre explode 결과

SQLite 특성상 리스트/불린은 문자열 or INTEGER로 변환 후 저장.

# 7. Visualization (QA 목적)
• 연도별 출시 게임 수 (bar chart)
• 리뷰 수(log scale) vs 긍정률 (scatter chart)

QA 목적의 간단한 검증용 시각화.

# 8. SQL Analysis

SQLite DB를 기반으로
연도별 평균 게임 가격 변화를 LAG 윈도우 함수로 계산.

WITH base AS (
    SELECT release_year, AVG(price) AS avg_price
    FROM steam_games
    GROUP BY release_year
)
SELECT
    release_year,
    avg_price,
    avg_price - LAG(avg_price) OVER (ORDER BY release_year) AS diff_from_prev
FROM base
ORDER BY release_year;


출력 예:

release_year	avg_price	diff_from_prev
2013	9.52	NULL
2014	10.17	0.65
…	…	…
# 9. Tech Stack

Python 3.10

Pandas, NumPy

Matplotlib

SQLite (sqlite3)

AST (literal_eval for list parsing)

# 10. Project Significance

이 프로젝트를 통해:

구조화된 ETL 파이프라인 설계

데이터 품질 검증(QA) 로직 직접 구현

리스트 형태 장르 데이터를 정규화하여 별도 테이블로 분리

SQL 윈도우 함수 적용하여 데이터 적재 검증

등 데이터 엔지니어링의 기본기를 실습했습니다.

간단한 미니 프로젝트지만
실제 데이터 파이프라인 구성의 핵심을 충실히 경험했습니다.

