---
name: setup
description: >
  프로젝트에 맞는 CLAUDE.md 한국어 템플릿을 생성합니다.
  웹앱, API 서버, 데이터 파이프라인, 모노레포, 사이드 프로젝트 유형을 지원합니다.
  Use when the user wants to create or set up CLAUDE.md in Korean.
argument-hint: "[프로젝트 유형]"
---

# CLAUDE.md 한국어 템플릿 생성

사용자의 프로젝트에 맞는 CLAUDE.md 템플릿을 생성해줘.

## 프로젝트 유형

인자가 없으면 먼저 물어봐:

"어떤 프로젝트야?
1. **웹앱** (React/Next.js/Vue 등)
2. **API 서버** (Spring/Express/FastAPI 등)
3. **데이터 파이프라인** (ETL, 배치 처리)
4. **모노레포** (여러 패키지 관리)
5. **사이드 프로젝트** (빠른 시작, 최소 설정)"

## 템플릿 위치

각 유형별 템플릿 파일:
- 웹앱: [templates/web-app.md](templates/web-app.md)
- API 서버: [templates/api-server.md](templates/api-server.md)
- 데이터 파이프라인: [templates/data-pipeline.md](templates/data-pipeline.md)
- 모노레포: [templates/monorepo.md](templates/monorepo.md)
- 사이드 프로젝트: [templates/side-project.md](templates/side-project.md)

## 동작

1. 해당 템플릿 파일을 읽어서 내용을 확인
2. 현재 프로젝트 디렉토리를 탐색해서 기술 스택을 자동 감지 (package.json, pom.xml, requirements.txt 등)
3. 감지된 정보로 [TODO] 항목을 최대한 자동으로 채움
4. 자동 감지 못한 부분은 [TODO]로 남겨두고 사용자에게 안내
5. ./CLAUDE.md 파일로 저장

사용자 입력: $ARGUMENTS
