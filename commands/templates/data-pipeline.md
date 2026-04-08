# CLAUDE.md — 데이터 파이프라인 프로젝트

## 프로젝트 개요
[TODO: 어떤 데이터를 어디서 가져와서 어디로 보내는지 한 줄로]

## 기술 스택
- 언어: [TODO: Python/Scala/Java]
- 오케스트레이션: [TODO: Airflow/Prefect/Dagster/Cron]
- 처리 엔진: [TODO: Spark/Pandas/Polars/Dask]
- 스토리지: [TODO: S3/GCS/HDFS/로컬]
- DB: [TODO: BigQuery/Snowflake/Redshift/PostgreSQL]
- 메시징: [TODO: Kafka/RabbitMQ/없음]

## 프로젝트 구조
[TODO: 아래는 예시]
├── dags/              # Airflow DAG 정의
├── pipelines/
│   ├── extract/       # 데이터 추출
│   ├── transform/     # 데이터 변환
│   └── load/          # 데이터 적재
├── models/            # 데이터 모델/스키마
├── tests/
├── config/
└── scripts/

## 데이터 흐름
[TODO: 소스 → 변환 → 목적지 설명]

## 코딩 컨벤션
- [TODO: 함수 네이밍 (extract_*, transform_*, load_*)]
- [TODO: 데이터 검증 규칙]
- [TODO: 멱등성(idempotency) 보장 방법]

## 빌드 & 실행
- 로컬 실행: [TODO: 실행 커맨드]
- 테스트: [TODO: pytest]
- 배포: [TODO: CI/CD 설명]

## 주의사항
- [TODO: 프로덕션 데이터 직접 접근 금지]
- [TODO: 스키마 변경 시 마이그레이션 필수]
- [TODO: PII 처리 규칙]
