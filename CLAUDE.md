# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

CCK(Claude Code Korean)는 한국어 사용자가 Claude Code를 자연어로 사용할 수 있게 해주는 **투명한 플러그인**이다. 이 저장소는 현재 설계 문서 단계이며, 실제 구현물로 발전시키는 것이 목표다.

- 핵심 설계 문서: `cck-plugin-design-v3.md`
- 참고 선행 버전: V2 → V3으로 개념 전환 (명시적 커맨드 학습 → 투명한 자동 감지)

## 목표 디렉토리 구조 (구현 시 기준)

```
cck/
├── .claude-plugin/
│   └── plugin.json              # 플러그인 매니페스트
├── data/
│   ├── commands.json            # 커맨드 DB (한국어 메타데이터 포함)
│   ├── aliases.json             # 한국어 자연어 → 커맨드 ID 매핑
│   └── recipes.json             # 멀티스텝 워크플로우 레시피
├── skills/
│   ├── korean-autopilot/        # 백그라운드 자동 활성화 스킬 (user-invocable: false)
│   │   └── SKILL.md
│   ├── cheat/                    # 사용자 호출: /cck:cheat
│   │   └── SKILL.md
│   └── setup/                    # 사용자 호출: /cck:setup [프로젝트유형]
│       ├── SKILL.md
│       └── templates/
│           ├── web-app.md
│           ├── api-server.md
│           ├── data-pipeline.md
│           ├── monorepo.md
│           └── side-project.md
├── agents/
│   └── korean-helper.md         # 한국어 도움 서브에이전트
└── README.md
```

## 핵심 설계 원칙

1. **투명성**: 사용자가 CCK의 존재를 몰라도 됨. 한국어로 말하면 Claude가 알아서 처리
2. **사용자 호출 스킬 최소화**: 명시적 호출은 `/cck:cheat`, `/cck:setup` 2개만
3. **korean-autopilot**이 나머지 모든 기능을 흡수 (찾기 + 레시피 + 설명)
4. **실행 가능/불가 구분**: `session_executable: true`면 직접 실행, `false`면 터미널 커맨드로 안내

## 데이터 스키마

### commands.json 핵심 필드
- `session_executable`: 세션 내에서 직접 실행 가능한지 여부 (true = 패턴 A 직접실행, false = 패턴 B 터미널안내)
- `tags`: 한국어 자연어 검색 키워드
- `summary_ko` / `detail_ko`: 한국어 설명

### aliases.json
자연어 키워드 → 커맨드 ID 배열. 예: `"이어서": ["continue-flag", "resume"]`

### recipes.json
step 타입: `"terminal"` (복사 가능 안내), `"claude-command"` (직접 실행), `"claude-prompt"` (사용자 프롬프트)

## korean-autopilot 응답 패턴

- **패턴 A** (session_executable: true): 설명 없이 바로 실행
- **패턴 B** (session_executable: false): 터미널에서 실행할 커맨드를 복사 가능한 코드 블록으로 안내
- **패턴 C** (멀티스텝): recipes.json 참조, 현재 단계 실행 후 다음 단계 안내
- **패턴 D** (문제 감지): 사용자가 요청 안 해도 선제적으로 제안 (예: 느려졌을 때 `/compact`)

## 언어 규칙

- 응답은 항상 한국어. 단, 커맨드/플래그 자체는 영어 원문 그대로 유지
- 기술 용어는 한국 개발자가 실제로 쓰는 표현 사용 ("커밋", "브랜치", "머지", "PR")
- 코드 블록 안의 커맨드는 복사 가능한 형태로 유지
