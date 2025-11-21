#BOAZ-miniProject
#Steam Games ETL Project (2025)

이 프로젝트는 2025년 3월까지 수집된 Steam 게임 데이터를 활용해
ETL 파이프라인 → 데이터 품질 검증(QA) → 구조화 테이블 생성 → 기본 SQL 분석까지 수행한
데이터 엔지니어링 미니 프로젝트입니다.

데이터 엔지니어링 입문자로서

데이터를 구조화하고
품질을 점검하며
분석 가능한 형태로 변환하는 작업을
직접 구현하는 것을 목표로 진행되었습니다.

1. Project Overview
단계	주요 내용
Extract	CSV 로컬 파일 로딩 (89,618 rows × 47 columns)
Transform	날짜·리뷰·장르 등 핵심 컬럼 정제, 품질 검증, 수익 추정
Load	CSV 저장 + SQLite DB 적재
Genre Normalization	리스트형 장르 explode → 별도 테이블 생성
QA	논리적 무결성 검증(2가지 rule) 및 간단 시각화
SQL Analysis	연도별 평균 게임 가격 변화 (LAG 윈도우 함수)
2. Data Pipeline
1) Extract

Kaggle Steam dataset 로딩

89,618개의 게임 레코드 확인

dtype 검사 및 원본 컬럼 구조 파악

df = pd.read_csv(RAW_PATH, encoding='utf-8')

2) Transform
✔ 핵심 컬럼 14개로 정제

필요한 컬럼만 선택하여 메모리 및 목적에 맞게 구조 축소.

✔ 날짜(Date) 변환
release_date → datetime64
release_year → int

✔ 긍정률(positive_rate) 생성

pct_pos_total 활용하여 positive_rate 재정의.

✔ 추정 수익 생성
est_revenue = price × num_reviews_total

3. Data Quality Check (QA)

ETL 과정에서 데이터 품질을 검증하는 과정이 중요하기 때문에
아래 2가지 논리 검증 규칙 추가.

✔ Rule 1 — 리뷰 수 검증

positive + negative > num_reviews_total이면 비정상 데이터로 간주.

✔ Rule 2 — 미래 출시일 검증

release_date > 오늘 날짜 → 데이터 오염 가능성 flag

출력 예시:

invalid_review    12
future_release     3

4. Genre Normalization (explode)

Steam 데이터의 genres 컬럼은
"['Action','Adventure']"처럼 리스트 형태 문자열로 저장되어 있어
그대로는 분석이 어렵습니다.

따라서:

문자열 → 실제 리스트 변환 (ast.literal_eval)

explode()로 게임(appid)–장르(genre) 매핑 테이블 생성

Steam의 순수 장르(valid_genres) 목록만 필터링하여 정제

생성된 테이블 예시:

appid	genre
730	Action
570	Strategy
359550	Action

이 테이블은 후속 SQL 분석 및 집계에 활용됨.

5. Load
✔ CSV 저장
df_main.to_csv(CLEAN_PATH, index=False)

✔ SQLite DB 적재

두 개의 테이블 생성:

테이블	설명
steam_games	게임 기본 정보(정제된 메인 테이블)
game_genres	appid–genre 매핑 테이블 (explode 결과)

특이 사항

SQLite는 리스트/불린 타입을 지원하지 않아 문자열 및 INT로 변환 후 저장

6. Visualization (QA 용도)
• 연도별 출시 게임 수 (bar chart)
• 리뷰 수(log scale) vs 긍정률 (scatter chart)

시각화 목적:

데이터 분포가 정상적인지, 이상값이 존재하는지 간단히 점검

7. SQL Analysis (Window Function)

정제된 DB를 활용해
연도별 평균 가격 변화를 LAG 윈도우 함수로 계산.

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


출력 예시:

release_year	avg_price	diff_from_prev
2013	9.52	NULL
2014	10.17	0.65
…	…	…
8. 기술 스택

Python 3.10

Pandas, NumPy

Matplotlib

SQLite (sqlite3)

AST (리스트 문자열 파싱용)

9. 프로젝트 의의

이 미니 프로젝트를 통해:

구조화된 ETL 파이프라인을 직접 설계해보고

데이터 품질 검증(QA) 로직을 적용하며

리스트 형태의 장르 데이터를 정규화해 별도 테이블로 분리하고

SQL 기반 집계를 통해 DB 적재를 검증하는

데이터 엔지니어링 기본 역량을 실습하였습니다.

간단한 프로젝트지만,
실제 데이터 파이프라인 구성의 주요 개념들을 체계적으로 경험하는 데 목적이 있습니다.
