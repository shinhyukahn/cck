# CLAUDE.md — 모노레포 프로젝트

## 프로젝트 개요
[TODO: 모노레포에 포함된 패키지/서비스 요약]

## 기술 스택
- 모노레포 도구: [TODO: Turborepo/Nx/Lerna/pnpm workspace]
- 공통 언어: [TODO: TypeScript/JavaScript/Python/Go]
- 패키지 매니저: [TODO: pnpm/yarn/npm]

## 패키지 구조
[TODO: 아래는 예시]
├── apps/
│   ├── web/           # [TODO: 프론트엔드]
│   ├── api/           # [TODO: 백엔드]
│   └── admin/         # [TODO: 어드민]
├── packages/
│   ├── ui/            # [TODO: 공유 UI]
│   ├── config/        # [TODO: 공유 설정]
│   ├── utils/         # [TODO: 공유 유틸]
│   └── types/         # [TODO: 공유 타입]
└── tools/

## 패키지 간 의존성
[TODO: 어떤 패키지가 어떤 패키지에 의존하는지]

## 코딩 컨벤션
- [TODO: 전체 공통 규칙]
- [TODO: 공유 패키지 수정 시 영향 범위 확인 규칙]
- [TODO: import 경로 규칙 (@app/ui 같은 alias)]

## 빌드 & 실행
- 전체 빌드: [TODO: turbo build]
- 특정 패키지만: [TODO: turbo build --filter=web]
- 개발 서버: [TODO: turbo dev]
- 테스트: [TODO: turbo test]

## 주의사항
- [TODO: 공유 패키지 수정 시 모든 의존 패키지 테스트 필수]
- [TODO: 루트 package.json에 의존성 추가 금지 등]
- [TODO: 배포 순서]
