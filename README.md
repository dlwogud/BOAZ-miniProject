# BOAZ-miniProject
# Steam Games ETL Project (2025)

### 개요
2025년 3월까지 수집된 Steam 게임 데이터를 기반으로  
**ETL 파이프라인 구축 → 데이터 검증(QA) → 기본 SQL 분석**까지 수행한  
데이터 엔지니어링 미니 프로젝트입니다.

## 1. 프로젝트 구성

### Extract
- Kaggle dataset 불러오기
- 원본 데이터 확인 (89,618 rows × 47 columns)

### Transform
- 필요한 14개 컬럼으로 축소
- 출시일(datetime) 변환
- 전체 긍정률(`pct_pos_total`)을 `positive_rate`로 적용
- 수익 추정(`est_revenue = price × num_reviews_total`)
- 연도(`release_year`) 생성

### Load
- CSV 저장
- SQLite DB 저장 (`steam_games.db`)

### QA (Data Quality Check)
- 결측치 검사
- 중복 검사
- describe() 요약

### Basic Visualization (검증용)
- 연도별 출시 게임 수
- 리뷰 수 vs 긍정률 로그 스케일 분포

### SQL 분석
```sql
SELECT release_year, COUNT(*) AS game_count, AVG(price) AS avg_price
FROM steam_games
GROUP BY release_year
ORDER BY release_year;
