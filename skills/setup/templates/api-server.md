# CLAUDE.md — API 서버 프로젝트

## 프로젝트 개요
[TODO: 프로젝트가 뭘 하는지 한 줄로]

## 기술 스택
- 언어: [TODO: Java/Python/TypeScript/Go/Rust]
- 프레임워크: [TODO: Spring Boot/FastAPI/Express/NestJS/Gin]
- DB: [TODO: PostgreSQL/MySQL/MongoDB/Redis]
- ORM: [TODO: JPA/SQLAlchemy/Prisma/TypeORM/GORM]
- 인증: [TODO: JWT/OAuth2/세션]
- 기타: [TODO: Redis, Kafka, RabbitMQ, ElasticSearch 등]

## 프로젝트 구조
[TODO: 주요 디렉토리 설명]
src/
├── controllers/    # API 엔드포인트
├── services/       # 비즈니스 로직
├── repositories/   # DB 접근 레이어
├── models/         # 데이터 모델/엔티티
├── middleware/     # 미들웨어
├── config/         # 설정 파일
├── dto/            # 요청/응답 DTO
└── utils/          # 유틸리티

## 코딩 컨벤션
- 네이밍: [TODO: camelCase/snake_case, 클래스명 규칙 등]
- 에러 처리: [TODO: 커스텀 예외 클래스, 글로벌 핸들러 등]
- 로깅: [TODO: 로그 레벨 규칙]
- 트랜잭션: [TODO: 서비스 레이어에서 관리 등]

## 빌드 & 실행
- 빌드: [TODO: mvn clean package / npm run build / go build]
- 실행: [TODO: 실행 커맨드]
- 테스트: [TODO: 테스트 커맨드]
- DB 마이그레이션: [TODO: flyway / alembic / prisma migrate]

## API 설계 원칙
- [TODO: REST 규칙, URL 패턴]
- [TODO: 응답 포맷]
- [TODO: HTTP 상태 코드 규칙]
- [TODO: 페이지네이션 방식]
- [TODO: 버전닝 방식]

## 주의사항
- [TODO: 절대 수정하면 안 되는 파일]
- [TODO: 환경 변수 건드리지 말 것]
- [TODO: 프로덕션 DB 직접 쿼리 금지 등]
