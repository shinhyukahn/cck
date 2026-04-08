# CCK (Claude Code Korean) - 플러그인 설계서 v3

## 한 줄 요약
> 설치만 하면 한국어로 말하는 것만으로 Claude Code를 완전히 조작할 수 있게 해주는 **투명한 플러그인**

---

## 핵심 철학: "사용자는 CCK의 존재를 몰라도 된다"

V2까지의 문제: `/cck:찾기`, `/cck:레시피` 같은 새 커맨드를 또 배워야 했음.
그건 "Claude Code 커맨드를 모르는 문제"를 "CCK 커맨드를 배워야 하는 문제"로 바꾼 것일 뿐.

V3의 원칙:
- **사용자는 한국어로 하고 싶은 일만 말한다**
- **Claude가 알아서 적절한 커맨드를 실행하거나 안내한다**
- **플러그인은 투명하게 동작한다 — 사용자가 호출할 필요 없음**
- **명시적 호출 스킬은 최소한으로 — 치트시트, CLAUDE.md 생성만**

### 사용자 경험 비교

| 상황 | 플러그인 없을 때 | V2 (CCK 커맨드 학습 필요) | **V3 (투명)** |
|------|-----------------|-------------------------|--------------|
| 이어서 하고 싶어 | 구글에 "claude code resume" 검색 | `/cck:찾기 이어서` | **"이어서 하고 싶어"** → Claude가 `claude -c` 안내 |
| 코드 리뷰 받고 싶어 | `/review`를 알아야 함 | `/cck:레시피 코드리뷰` | **"코드 리뷰해줘"** → Claude가 `/review` 실행 |
| 컨텍스트 부족 | 원인도 모름 | `/cck:찾기 정리` | **"좀 느려진 것 같아"** → Claude가 `/compact` 제안 |
| CLAUDE.md 만들기 | `/init` → 영어 빈 파일 | `/cck:설정 api` | **"CLAUDE.md 만들어줘"** → 자동 감지 + 한국어 템플릿 |

---

## 1. 플러그인 구조

```
cck/
├── .claude-plugin/
│   └── plugin.json              # 플러그인 매니페스트
│
├── data/                         # 핵심 지식 데이터 (모든 스킬이 공유)
│   ├── commands.json            # 전체 커맨드 DB (한국어)
│   ├── aliases.json             # 한국어 자연어 → 커맨드 매핑
│   └── recipes.json             # 워크플로우 레시피 모음
│
├── skills/
│   ├── korean-autopilot/         # 핵심: 백그라운드 자동 활성화 스킬
│   │   └── SKILL.md
│   │
│   ├── 치트/                     # 유일한 사용자 호출 스킬 1: 치트시트
│   │   └── SKILL.md
│   │
│   └── 설정/                     # 유일한 사용자 호출 스킬 2: CLAUDE.md 생성
│       ├── SKILL.md
│       └── templates/
│           ├── web-app.md
│           ├── api-server.md
│           ├── data-pipeline.md
│           ├── monorepo.md
│           └── side-project.md
│
├── agents/
│   └── korean-helper.md         # 한국어 도움 서브에이전트
│
└── README.md
```

V2 대비 변화:
- 6개 스킬 → **3개로 축소** (찾기, 레시피, 설명 삭제 → korean-autopilot에 통합)
- `korean-autopilot`이 찾기 + 레시피 + 설명 기능을 전부 흡수
- 사용자가 직접 호출하는 건 `치트`, `설정` 2개뿐

---

## 2. plugin.json

```json
{
  "name": "cck",
  "description": "Claude Code를 한국어로 자연스럽게 사용할 수 있게 해주는 플러그인. 설치만 하면 한국어 자연어로 커맨드를 찾고, 워크플로우를 따라하고, 프로젝트를 셋업할 수 있습니다. Transparent Korean language plugin for Claude Code — just install and speak Korean naturally.",
  "version": "1.0.0",
  "author": {
    "name": "XSignal",
    "url": "https://github.com/xsignal"
  },
  "repository": "https://github.com/xsignal/cck",
  "license": "MIT",
  "keywords": ["korean", "한국어", "navigator", "autopilot", "transparent"]
}
```

---

## 3. 스킬 상세 설계

### 3.1 `korean-autopilot` — 핵심 백그라운드 스킬

이 스킬이 플러그인의 90%. 사용자가 호출하지 않음. Claude가 자동 활성화.

```markdown
---
name: korean-autopilot
description: >
  한국어로 Claude Code 사용법을 묻거나, 한국어로 작업을 지시하는 사용자를 자동으로 감지하고
  적절한 Claude Code 커맨드를 실행하거나 안내합니다.
  Automatically detects Korean-speaking users and helps them navigate Claude Code
  by executing or suggesting the right commands based on natural language input.
  Triggers when: user speaks Korean and asks about Claude Code commands,
  seems confused about what command to use, asks how to do something in Claude Code,
  mentions wanting to continue/resume/review/deploy/setup, or experiences issues
  like slow responses or context limits.
user-invocable: false
---

# Claude Code 한국어 오토파일럿

당신은 Claude Code 세션 안에서 동작하는 한국어 보조 시스템입니다.
사용자가 한국어로 Claude Code 관련 요청을 하면 자동으로 활성화됩니다.

## 핵심 원칙

1. **사용자가 새로운 커맨드를 배울 필요 없게 하라**
   - "이어서 하고 싶어" → `claude -c`를 안내하되, 가능하면 직접 실행
   - "코드 리뷰해줘" → `/review`를 직접 실행
   - "컨텍스트 정리해" → `/compact`를 직접 실행

2. **실행 가능하면 실행하고, 세션 밖 커맨드면 안내하라**
   - 세션 안 커맨드 (`/compact`, `/diff`, `/review` 등): **직접 실행**
   - 세션 밖 커맨드 (`claude -c`, `claude -w` 등): **복사 가능하게 안내**
   - 설치/설정 (`claude plugin install`, `claude mcp add` 등): **단계별 안내**

3. **최소한의 설명만 하라**
   - 장황한 설명 금지. 실행하고 결과를 보여주면 됨
   - 부가 설명이 필요할 때만 간결하게 추가
   - "이 커맨드는 ~~하는 기능으로..." 같은 교과서식 설명 금지

4. **맥락을 읽어라**
   - "느려졌어" → 컨텍스트 문제일 가능성 → `/compact` 제안
   - "아까 한 거 취소" → `/rewind` 실행
   - "다른 방법으로 해봐" → `/rewind` 후 재시도 제안
   - "PR 올려" → worktree + commit + gh pr create 워크플로우 안내

## 응답 패턴

### 패턴 A: 직접 실행 가능한 경우
사용자가 세션 안에서 실행 가능한 작업을 요청하면, 설명 없이 바로 실행.
실행 후 필요하면 한 줄 설명만 덧붙임.

예시:
- "코드 리뷰해줘" → `/review` 실행
- "변경사항 보여줘" → `/diff` 실행
- "컨텍스트 좀 정리해" → `/compact` 실행
- "이전으로 되돌려" → `/rewind` 실행
- "보안 점검해줘" → `/security-review` 실행
- "지금까지 얼마 썼어?" → `/cost` 실행

### 패턴 B: 세션 밖 커맨드 안내
터미널에서 새로 입력해야 하는 커맨드는 복사 가능하게 제공.

형식:
```
[한 줄 설명]

터미널에서 실행해:
`[커맨드]`

💡 [실전 팁 한 줄 — 있을 때만]
```

예시:
- "어제 하던 거 이어서" →
  "마지막 대화를 이어서 하려면:
   `claude -c`"
   
- "독립된 공간에서 작업하고 싶어" →
  "별도 워크트리에서 시작하려면:
   `claude -w 작업이름`
   💡 --tmux 붙이면 병렬 작업 가능"

### 패턴 C: 멀티스텝 워크플로우 안내
여러 단계가 필요한 작업은 레시피 데이터를 참조해서 단계별로 안내.
한번에 다 보여주지 말고, 현재 단계 실행 → 다음 단계 안내 순서로.

형식:
```
[목표] — [전체 N단계]

**지금 할 것:**
`[커맨드]`
[한 줄 설명]

[실행 후 다음 단계를 안내할게.]
```

예시:
- "PR 만들어서 올리고 싶어" →
  "PR 워크플로우 — 4단계

   **지금 할 것:**
   이 세션에서 바로 작업할 수도 있고,
   독립 워크트리에서 안전하게 하려면:
   `claude -w feature-name`

   작업이 끝나면 다음 단계를 안내할게."

### 패턴 D: 문제 상황 감지
사용자가 직접 도움을 요청하지 않아도, 문제 신호를 감지하면 선제적으로 제안.

감지 신호 → 제안:
- "응답이 느려졌어" / "이상하게 대답해" → "컨텍스트가 가득 찼을 수 있어. `/compact` 실행할까?"
- "아까 한 거 별로야" / "되돌리고 싶어" → "`/rewind`로 이전 시점으로 돌아갈 수 있어. 실행할까?"
- "토큰 많이 쓰고 있어?" / "비용이 걱정돼" → `/cost` 실행
- "이 프로젝트 처음인데" / "셋업 어떻게 해" → `/cck:설정` 안내
- 긴 대화 후 새 주제 시작 → "새 주제네. `/compact`로 이전 내용 정리할까?"

## 데이터 참조

정확한 커맨드 정보는 이 플러그인의 데이터 파일을 참조:
- 커맨드 DB: [commands.json](../../data/commands.json)
- 자연어 매핑: [aliases.json](../../data/aliases.json)
- 워크플로우 레시피: [recipes.json](../../data/recipes.json)

사용자의 자연어 입력이 aliases.json의 키워드와 매칭되면 해당 커맨드를 우선 제안.
매칭되지 않으면 commands.json의 tags, summary_ko 필드에서 의미적으로 가장 가까운 것을 찾음.
멀티스텝 작업이면 recipes.json에서 해당 레시피를 참조.

## 언어 규칙

1. **응답은 항상 한국어** — 단, 커맨드/플래그 자체는 영어 원문 유지
2. **커맨드를 한국어로 번역하지 마** — `/compact`을 "압축"으로 바꾸지 않음
3. **코드 블록 안의 커맨드는 그대로 복사 가능해야 함**
4. **기술 용어는 한국 개발자가 실제로 쓰는 표현** — "커밋", "브랜치", "머지", "PR" 등
```

### 3.2 `/cck:치트` — 치트시트 (사용자 호출)

유일하게 명시적 호출이 자연스러운 스킬. "한 눈에 보고 싶을 때" 쓰는 거라 자동화가 안 맞음.

```markdown
---
name: 치트
description: >
  Claude Code 핵심 커맨드를 한 눈에 볼 수 있는 한국어 치트시트.
  Use when the user asks for a cheatsheet, quick reference, or "전체 명령어 보여줘".
disable-model-invocation: true
---

# Claude Code 한국어 치트시트

아래 치트시트를 그대로 보여줘. 포맷을 바꾸지 마.

---

## ⚡ Claude Code 치트시트

### 시작하기
| 커맨드 | 설명 |
|--------|------|
| `claude` | 대화 시작 |
| `claude "질문"` | 질문과 함께 시작 |
| `claude -c` | 마지막 대화 이어서 |
| `claude -r <이름>` | 특정 세션 복구 |
| `claude -n "이름"` | 세션에 이름 붙이기 |
| `cat file \| claude -p "질문"` | 파일 내용 넘겨서 질문 |

### 대화 중 자주 쓰는 것
| 커맨드 | 설명 |
|--------|------|
| `/compact` | 컨텍스트 정리 (요약) |
| `/clear` | 대화 완전 초기화 |
| `/model` | 모델 변경 |
| `/diff` | 변경사항 보기 |
| `/rewind` | 이전 시점으로 되돌리기 |
| `/cost` | 토큰 사용량 확인 |
| `/copy` | 마지막 응답 복사 |
| `/context` | 컨텍스트 사용량 시각화 |

### 프로젝트 관리
| 커맨드 | 설명 |
|--------|------|
| `/init` | CLAUDE.md 생성 |
| `/memory` | 메모리 파일 관리 |
| `/permissions` | 권한 규칙 관리 |
| `/plugin` | 플러그인 관리 |
| `/skills` | 사용 가능한 스킬 목록 |

### 코드 작업
| 커맨드 | 설명 |
|--------|------|
| `/review` | 코드 리뷰 (플러그인 필요) |
| `/security-review` | 보안 취약점 분석 |
| `/simplify` | 코드 품질 개선 |
| `/batch <지시>` | 대규모 변경 병렬 처리 |
| `/pr-comments` | PR 코멘트 확인 |

### 실행 옵션 (플래그)
| 플래그 | 설명 |
|--------|------|
| `--worktree` (`-w`) | 독립 작업 공간에서 실행 |
| `--effort high` | 더 깊이 생각하게 |
| `--model sonnet` | 모델 지정 |
| `--tmux` | tmux 세션으로 열기 |
| `--chrome` | 브라우저 자동화 활성화 |
| `--dangerously-skip-permissions` | 권한 확인 건너뛰기 |

### 단축키
| 키 | 설명 |
|-----|------|
| `Shift+Tab` | 권한 모드 전환 |
| `Ctrl+C` | 현재 응답 중단 |
| `Ctrl+L` | 화면 정리 |
| `Esc` (2번) | 현재 작업 중단 |
| `Shift+Enter` | 여러 줄 입력 |

---

💡 더 자세한 설명이 필요하면 한국어로 물어보면 돼.
예: "worktree가 뭐야?", "PR 어떻게 올려?"
```

### 3.3 `/cck:설정` — CLAUDE.md 한국어 템플릿 (사용자 호출)

프로젝트 초기 셋업은 명시적 의도가 필요해서 사용자 호출로 유지.
단, `korean-autopilot`이 "CLAUDE.md 만들어줘" 같은 자연어를 감지하면 이 스킬을 자동 안내.

```markdown
---
name: 설정
description: >
  프로젝트에 맞는 CLAUDE.md 한국어 템플릿을 생성합니다.
  웹앱, API 서버, 데이터 파이프라인, 모노레포, 사이드 프로젝트 유형을 지원합니다.
  Use when the user wants to create or set up CLAUDE.md in Korean.
argument-hint: [프로젝트 유형]
disable-model-invocation: true
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
```

---

## 4. 데이터 파일 설계 (`data/` 디렉토리)

> 모든 데이터 파일은 플러그인 루트의 `data/` 디렉토리에 위치.
> `korean-autopilot` 스킬과 `korean-helper` 에이전트가 공유 참조.

### 4.1 data/commands.json

V2와 동일한 스키마. 핵심 10개 예시 + 확장 목록.

```json
{
  "version": "1.0.0",
  "last_updated": "2026-04-08",
  "commands": [
    {
      "id": "compact",
      "type": "slash",
      "name": "/compact",
      "aliases": [],
      "args": "[지시사항]",
      "summary_ko": "대화 내용을 요약해서 컨텍스트를 줄임",
      "detail_ko": "긴 대화를 하다 보면 컨텍스트 윈도우가 차서 성능이 떨어짐. /compact를 쓰면 대화 내용을 요약해서 공간을 확보함.",
      "when_to_use": [
        "대화가 길어져서 Claude 응답이 느려질 때",
        "특정 주제에만 집중하고 싶을 때",
        "컨텍스트 공간 부족 경고가 나올 때"
      ],
      "examples": [
        "/compact",
        "/compact \"테스트 관련 내용만 남겨줘\""
      ],
      "cautions": [
        "요약 과정에서 세부 내용이 손실될 수 있음",
        "중요한 맥락은 CLAUDE.md에 기록해두는 게 안전"
      ],
      "related": ["clear", "context", "cost"],
      "tags": ["정리", "컨텍스트", "요약", "압축", "대화정리", "토큰절약", "느려졌을때"],
      "category": "대화관리",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "continue-flag",
      "type": "flag",
      "name": "-c (--continue)",
      "aliases": [],
      "args": "",
      "summary_ko": "마지막 대화를 이어서 계속함",
      "detail_ko": "현재 디렉토리에서 가장 최근에 했던 대화를 불러와서 이어서 작업.",
      "when_to_use": [
        "어제 하던 작업을 이어서 할 때",
        "실수로 터미널을 닫았을 때",
        "이전 맥락을 유지하며 추가 작업할 때"
      ],
      "examples": [
        "claude -c",
        "claude -c -p \"타입 에러 확인해줘\""
      ],
      "cautions": [
        "현재 디렉토리 기준으로 마지막 세션을 찾음",
        "다른 디렉토리의 세션은 -r로 직접 지정해야 함"
      ],
      "related": ["resume", "resume-flag"],
      "tags": ["이어서", "계속", "이어하기", "복구", "세션"],
      "category": "시작",
      "difficulty": "beginner",
      "session_executable": false
    },
    {
      "id": "diff",
      "type": "slash",
      "name": "/diff",
      "aliases": [],
      "args": "",
      "summary_ko": "커밋되지 않은 변경사항과 턴별 diff를 인터랙티브하게 확인",
      "detail_ko": "현재 git diff와 Claude가 각 턴에서 변경한 내용을 비교할 수 있는 인터랙티브 뷰어.",
      "when_to_use": [
        "Claude가 수정한 내용을 확인할 때",
        "커밋 전에 변경사항을 검토할 때",
        "어떤 턴에서 뭘 바꿨는지 추적할 때"
      ],
      "examples": ["/diff"],
      "cautions": ["git 저장소 안에서만 동작"],
      "related": ["rewind", "review"],
      "tags": ["변경사항", "diff", "확인", "리뷰", "수정내역", "바뀐것"],
      "category": "코드작업",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "rewind",
      "type": "slash",
      "name": "/rewind",
      "aliases": ["/checkpoint"],
      "args": "",
      "summary_ko": "대화와 코드를 이전 시점으로 되돌림",
      "detail_ko": "Claude가 작업한 내용이 마음에 안 들 때, 특정 시점으로 대화와 파일 상태를 모두 되돌릴 수 있음.",
      "when_to_use": [
        "Claude의 수정이 마음에 안 들 때",
        "다른 접근법으로 다시 시도하고 싶을 때",
        "실수로 잘못된 방향으로 진행됐을 때"
      ],
      "examples": ["/rewind"],
      "cautions": [
        "되돌리면 해당 시점 이후의 작업은 사라짐",
        "git 저장소 필요"
      ],
      "related": ["diff", "branch"],
      "tags": ["되돌리기", "복구", "취소", "이전으로", "롤백", "체크포인트"],
      "category": "대화관리",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "worktree-flag",
      "type": "flag",
      "name": "--worktree (-w)",
      "aliases": [],
      "args": "[이름]",
      "summary_ko": "독립된 git worktree에서 Claude를 실행",
      "detail_ko": "메인 브랜치를 건드리지 않고 별도의 작업 공간에서 Claude를 돌림.",
      "when_to_use": [
        "큰 리팩토링을 안전하게 시킬 때",
        "여러 작업을 동시에 돌릴 때",
        "실험적 변경을 해보고 싶을 때"
      ],
      "examples": [
        "claude -w feature-auth",
        "claude -w feature-auth --tmux"
      ],
      "cautions": [
        "git 저장소 안에서만 동작",
        "worktree는 .claude/worktrees/ 아래에 생성됨",
        "작업 끝나면 git worktree remove로 정리"
      ],
      "related": ["tmux-flag", "diff", "continue-flag"],
      "tags": ["워크트리", "브랜치", "병렬", "독립", "안전", "리팩토링"],
      "category": "고급",
      "difficulty": "intermediate",
      "session_executable": false
    },
    {
      "id": "effort-flag",
      "type": "flag",
      "name": "--effort",
      "aliases": [],
      "args": "low|medium|high|max",
      "summary_ko": "Claude가 얼마나 깊이 생각할지 설정",
      "detail_ko": "low는 빠르지만 얕게, high는 느리지만 깊게 분석. max는 Opus 4.6 전용.",
      "when_to_use": [
        "복잡한 버그를 깊이 분석할 때 (high)",
        "간단한 질문에 빠르게 답받고 싶을 때 (low)",
        "아키텍처 설계 같은 고난도 작업 (max)"
      ],
      "examples": [
        "claude --effort high",
        "/effort high",
        "/effort low"
      ],
      "cautions": [
        "max는 Opus 4.6에서만 사용 가능",
        "높을수록 토큰 소비가 많아짐"
      ],
      "related": ["model", "cost", "fast"],
      "tags": ["깊게", "빠르게", "성능", "품질", "생각", "분석"],
      "category": "설정",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "init",
      "type": "slash",
      "name": "/init",
      "aliases": [],
      "args": "",
      "summary_ko": "프로젝트에 CLAUDE.md 가이드 파일 생성",
      "detail_ko": "프로젝트 루트에 CLAUDE.md를 만들어서 Claude가 프로젝트 맥락을 이해하도록 설정.",
      "when_to_use": [
        "새 프로젝트에서 Claude Code를 처음 설정할 때",
        "기존 프로젝트에 Claude Code를 도입할 때"
      ],
      "examples": ["/init"],
      "cautions": [
        "이미 CLAUDE.md가 있으면 덮어쓸 수 있음",
        "한국어 템플릿이 필요하면 /cck:설정 추천"
      ],
      "related": ["memory", "permissions"],
      "tags": ["초기화", "설정", "셋업", "CLAUDE.md", "시작"],
      "category": "프로젝트",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "model",
      "type": "slash",
      "name": "/model",
      "aliases": [],
      "args": "[모델명]",
      "summary_ko": "AI 모델 선택 또는 변경",
      "detail_ko": "현재 세션에서 사용할 모델을 변경. 변경은 즉시 적용됨.",
      "when_to_use": [
        "더 강력한 모델이 필요할 때 (opus)",
        "비용을 절약하고 싶을 때 (sonnet)",
        "현재 모델의 응답이 만족스럽지 않을 때"
      ],
      "examples": [
        "/model",
        "claude --model opus"
      ],
      "cautions": ["모델에 따라 비용이 다름"],
      "related": ["effort", "cost", "fast"],
      "tags": ["모델", "변경", "opus", "sonnet", "바꾸기"],
      "category": "설정",
      "difficulty": "beginner",
      "session_executable": true
    },
    {
      "id": "batch",
      "type": "slash",
      "name": "/batch",
      "aliases": [],
      "args": "<지시사항>",
      "summary_ko": "대규모 변경을 병렬 에이전트로 처리",
      "detail_ko": "코드베이스를 분석해서 작업을 5~30개 단위로 쪼갠 후 병렬 실행.",
      "when_to_use": [
        "대규모 마이그레이션 (프레임워크, 라이브러리 교체)",
        "코드베이스 전체에 일괄 변경이 필요할 때",
        "여러 파일에 동일한 패턴 적용"
      ],
      "examples": [
        "/batch src/ 디렉토리를 React에서 Vue로 마이그레이션",
        "/batch 모든 API 엔드포인트에 rate limiting 추가"
      ],
      "cautions": [
        "git 저장소 필요",
        "많은 토큰을 소비함",
        "각 단위가 독립적이어야 효과적"
      ],
      "related": ["worktree-flag", "simplify"],
      "tags": ["대규모", "병렬", "마이그레이션", "일괄", "자동화"],
      "category": "고급",
      "difficulty": "advanced",
      "session_executable": true
    },
    {
      "id": "fast",
      "type": "slash",
      "name": "/fast",
      "aliases": [],
      "args": "[on|off]",
      "summary_ko": "빠른 모드 토글 (속도 우선)",
      "detail_ko": "응답 속도를 우선시하는 모드. 간단한 질문이나 빠른 수정에 유용.",
      "when_to_use": [
        "간단한 질문에 빠르게 답받고 싶을 때",
        "사소한 코드 수정을 빠르게 할 때"
      ],
      "examples": ["/fast on", "/fast off"],
      "cautions": ["복잡한 작업에는 품질이 떨어질 수 있음"],
      "related": ["effort", "model"],
      "tags": ["빠르게", "속도", "패스트"],
      "category": "설정",
      "difficulty": "beginner",
      "session_executable": true
    }
  ]
}
```

> **V3 변경점: `session_executable` 필드 추가.**
> `true`면 korean-autopilot이 직접 실행 (패턴 A).
> `false`면 터미널 커맨드로 안내 (패턴 B).

MVP(Phase 1) 추가 예정 커맨드 ID 목록 (위 10개 외):

**슬래시 커맨드 (추가 예정)**:
`clear`, `config`, `context`, `copy`, `cost`, `exit`, `export`, `feedback`,
`help`, `memory`, `permissions`, `plugin`, `resume`, `review`, `security-review`,
`simplify`, `skills`, `agents`, `branch`, `btw`, `color`, `doctor`,
`install-github-app`, `install-slack-app`, `mcp`, `pr-comments`, `status`,
`theme`, `usage`, `vim`, `voice`, `debug`, `plan`

**CLI 플래그 (추가 예정)**:
`-p (--print)`, `-r (--resume)`, `-n (--name)`, `--model`, `--add-dir`,
`--chrome`, `--tmux`, `--dangerously-skip-permissions`, `--append-system-prompt`,
`--mcp-config`, `--bare`, `--remote`, `--permission-mode`, `--fallback-model`

> 각 항목은 위 예시와 동일한 JSON 스키마로 작성.

### 4.2 data/aliases.json

V2와 동일. 전문:

```json
{
  "version": "1.0.0",
  "mappings": {
    "이어서": ["continue-flag", "resume"],
    "이어하기": ["continue-flag", "resume"],
    "계속": ["continue-flag", "resume"],
    "정리": ["compact", "clear"],
    "요약": ["compact"],
    "압축": ["compact"],
    "초기화": ["clear", "init"],
    "되돌리기": ["rewind"],
    "롤백": ["rewind"],
    "취소": ["rewind"],
    "변경사항": ["diff"],
    "뭐바뀌었어": ["diff"],
    "수정내역": ["diff"],
    "비용": ["cost"],
    "얼마": ["cost"],
    "토큰": ["cost", "context"],
    "모델": ["model"],
    "바꾸기": ["model"],
    "복사": ["copy"],
    "리뷰": ["review"],
    "검토": ["review", "security-review"],
    "보안": ["security-review"],
    "권한": ["permissions"],
    "설정": ["config"],
    "메모리": ["memory"],
    "브랜치": ["worktree-flag", "branch"],
    "병렬": ["worktree-flag", "batch"],
    "동시에": ["worktree-flag", "batch"],
    "에이전트": ["agents"],
    "플러그인": ["plugin"],
    "디버그": ["debug"],
    "깊게": ["effort-flag"],
    "빠르게": ["fast", "effort-flag"],
    "PR": ["pr-comments", "install-github-app"],
    "MCP": ["mcp"],
    "슬랙": ["install-slack-app"],
    "종료": ["exit"],
    "나가기": ["exit"],
    "도움말": ["help"],
    "버그신고": ["feedback"],
    "업데이트": ["update"],
    "내보내기": ["export"],
    "테마": ["theme"],
    "단순화": ["simplify"],
    "대규모": ["batch"],
    "자동": ["auto-mode", "dangerously-skip-permissions"],
    "느려": ["compact", "effort-flag", "fast"],
    "공간부족": ["compact", "context"],
    "시작": ["init", "continue-flag"],
    "새로": ["clear", "init"]
  }
}
```

### 4.3 data/recipes.json

V2와 동일. 10개 레시피 전문:

```json
{
  "version": "1.0.0",
  "recipes": [
    {
      "id": "code-review",
      "title": "코드 리뷰 받기",
      "category": "일상",
      "difficulty": "beginner",
      "description": "Claude에게 현재 변경사항에 대한 코드 리뷰를 받는 워크플로우",
      "prerequisites": [],
      "steps": [
        {
          "order": 1,
          "action": "claude plugin install code-review@claude-plugins-official",
          "description": "코드 리뷰 플러그인 설치 (최초 1회)",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "/review",
          "description": "리뷰 실행",
          "type": "claude-command"
        },
        {
          "order": 3,
          "action": "/diff",
          "description": "변경사항 인터랙티브 확인",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "/security-review",
          "description": "(선택) 보안 취약점 분석",
          "type": "claude-command",
          "optional": true
        }
      ],
      "tips": [
        "CLAUDE.md에 코드 스타일 가이드를 적어두면 리뷰 품질이 올라감",
        "/compact로 컨텍스트 정리 후 리뷰하면 더 정확함"
      ],
      "related_recipes": ["pr-workflow"]
    },
    {
      "id": "bug-hunting",
      "title": "버그 잡기",
      "category": "일상",
      "difficulty": "beginner",
      "description": "에러 로그를 Claude에게 넘겨서 디버깅",
      "prerequisites": [],
      "steps": [
        {
          "order": 1,
          "action": "cat error.log | claude -p \"이 에러 분석해줘\"",
          "description": "에러 로그를 파이프로 넘기기",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "또는: claude 세션에서 에러 메시지 직접 붙여넣기",
          "description": "인터랙티브하게 디버깅",
          "type": "claude-prompt"
        },
        {
          "order": 3,
          "action": "/diff",
          "description": "Claude가 수정한 코드 확인",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "/rewind",
          "description": "수정이 마음에 안 들면 되돌리기",
          "type": "claude-command",
          "optional": true
        }
      ],
      "tips": [
        "--effort high로 복잡한 버그를 더 깊이 분석하게 할 수 있음",
        "\"이 에러가 왜 발생하는지 단계별로 추적해줘\" 같은 구체적 지시가 효과적"
      ],
      "related_recipes": ["refactoring"]
    },
    {
      "id": "refactoring",
      "title": "리팩토링",
      "category": "일상",
      "difficulty": "intermediate",
      "description": "메인 코드를 보호하면서 안전하게 리팩토링",
      "prerequisites": ["git 저장소"],
      "steps": [
        {
          "order": 1,
          "action": "claude -w refactor-name",
          "description": "독립 워크트리에서 시작 — 메인 브랜치 보호",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "\"이 모듈을 단일 책임 원칙에 맞게 분리해줘\"",
          "description": "리팩토링 목표를 구체적으로 지시",
          "type": "claude-prompt"
        },
        {
          "order": 3,
          "action": "/diff",
          "description": "변경사항 확인",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "/rewind",
          "description": "마음에 안 들면 특정 시점으로 복구",
          "type": "claude-command",
          "optional": true
        },
        {
          "order": 5,
          "action": "\"테스트 돌려서 깨진 거 없는지 확인해줘\"",
          "description": "리팩토링 후 기존 기능 검증",
          "type": "claude-prompt"
        }
      ],
      "tips": [
        "CLAUDE.md에 아키텍처 규칙을 적어두면 일관성 있는 리팩토링 가능",
        "큰 리팩토링은 --effort high 추천"
      ],
      "related_recipes": ["code-review", "test-writing"]
    },
    {
      "id": "test-writing",
      "title": "테스트 작성",
      "category": "일상",
      "difficulty": "beginner",
      "description": "기존 코드에 대한 테스트를 Claude에게 자동 작성시키기",
      "prerequisites": ["테스트 프레임워크 설치"],
      "steps": [
        {
          "order": 1,
          "action": "claude",
          "description": "세션 시작",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "\"src/auth/ 디렉토리의 유닛 테스트 작성해줘. 커버리지 80% 이상 목표\"",
          "description": "테스트 대상과 목표를 구체적으로 지정",
          "type": "claude-prompt"
        },
        {
          "order": 3,
          "action": "(Claude가 자동으로 테스트 실행 → 실패 시 수정 반복)",
          "description": "Claude가 테스트를 작성하고 실행까지 처리",
          "type": "auto"
        },
        {
          "order": 4,
          "action": "/diff",
          "description": "생성된 테스트 파일 확인",
          "type": "claude-command"
        }
      ],
      "tips": [
        "CLAUDE.md에 테스트 프레임워크와 컨벤션을 명시하면 품질 올라감",
        "\"엣지 케이스도 포함해줘\" 같은 지시를 추가하면 더 견고한 테스트 생성"
      ],
      "related_recipes": ["refactoring", "code-review"]
    },
    {
      "id": "pr-workflow",
      "title": "PR 만들어서 올리기",
      "category": "프로젝트관리",
      "difficulty": "intermediate",
      "description": "Claude로 작업 → 커밋 → PR 생성까지 한번에",
      "prerequisites": ["gh CLI 설치", "git 저장소"],
      "steps": [
        {
          "order": 1,
          "action": "claude -w feature-name",
          "description": "독립 워크트리에서 작업 시작",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "\"로그인 API에 rate limiting 추가해줘\"",
          "description": "작업 내용을 자연어로 지시",
          "type": "claude-prompt"
        },
        {
          "order": 3,
          "action": "/diff",
          "description": "변경사항 확인",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "\"커밋하고 PR 만들어줘. PR 설명도 작성해줘\"",
          "description": "Claude가 커밋 + PR 생성 + 설명 작성까지 수행",
          "type": "claude-prompt"
        },
        {
          "order": 5,
          "action": "/pr-comments",
          "description": "(이후) PR에 달린 코멘트 확인",
          "type": "claude-command",
          "optional": true
        }
      ],
      "tips": [
        "--tmux 옵션으로 여러 PR 작업을 병렬로 돌릴 수 있음",
        "CLAUDE.md에 커밋 메시지 컨벤션을 적어두면 일관된 커밋 메시지 생성"
      ],
      "related_recipes": ["parallel-tasks", "code-review"]
    },
    {
      "id": "parallel-tasks",
      "title": "여러 작업 병렬 처리",
      "category": "프로젝트관리",
      "difficulty": "advanced",
      "description": "git worktree를 활용해서 Claude 세션 여러 개를 동시에 돌림",
      "prerequisites": ["tmux 설치", "git 저장소"],
      "steps": [
        {
          "order": 1,
          "action": "claude -w auth-refactor --tmux",
          "description": "첫 번째 작업 시작",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "claude -w fix-tests --tmux",
          "description": "두 번째 작업 시작 (새 tmux 패인)",
          "type": "terminal"
        },
        {
          "order": 3,
          "action": "각 세션에서 별도 작업 지시",
          "description": "각 tmux 패인에서 독립적으로 작업",
          "type": "claude-prompt"
        },
        {
          "order": 4,
          "action": "/diff (각 세션에서)",
          "description": "각 작업의 변경사항 확인 후 메인에 머지",
          "type": "claude-command"
        }
      ],
      "tips": [
        "3~4개가 적당. 너무 많으면 API rate limit에 걸림",
        "작업 끝나면 git worktree remove <이름>으로 정리"
      ],
      "related_recipes": ["pr-workflow"]
    },
    {
      "id": "project-setup",
      "title": "프로젝트 초기 셋업",
      "category": "프로젝트관리",
      "difficulty": "beginner",
      "description": "새 프로젝트에서 CLAUDE.md를 설정하고 Claude Code 환경 잡기",
      "prerequisites": [],
      "steps": [
        {
          "order": 1,
          "action": "/cck:설정",
          "description": "프로젝트 유형에 맞는 한국어 CLAUDE.md 템플릿 생성",
          "type": "claude-command"
        },
        {
          "order": 2,
          "action": "CLAUDE.md에서 [TODO] 항목 채우기",
          "description": "프로젝트 구조, 코딩 컨벤션, 주의사항 직접 작성",
          "type": "manual"
        },
        {
          "order": 3,
          "action": "claude",
          "description": "Claude Code 세션 시작 — CLAUDE.md를 자동으로 읽음",
          "type": "terminal"
        },
        {
          "order": 4,
          "action": "/permissions",
          "description": "프로젝트에 맞는 권한 규칙 설정",
          "type": "claude-command"
        }
      ],
      "tips": [
        "CLAUDE.md는 .gitignore에 넣지 마 — 팀원들도 공유해서 써야 함",
        "처음에 완벽하게 안 써도 됨 — 점진적으로 보강"
      ],
      "related_recipes": ["code-review"]
    },
    {
      "id": "mcp-setup",
      "title": "MCP 서버 연결",
      "category": "고급",
      "difficulty": "advanced",
      "description": "외부 도구를 MCP로 Claude에 연결",
      "prerequisites": ["MCP 서버 준비"],
      "steps": [
        {
          "order": 1,
          "action": "claude mcp add <이름> -- <실행커맨드>",
          "description": "MCP 서버 등록",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": "/mcp",
          "description": "연결 상태 확인",
          "type": "claude-command"
        },
        {
          "order": 3,
          "action": "\"DB에서 users 테이블 스키마 확인해줘\"",
          "description": "연결된 도구 사용 지시",
          "type": "claude-prompt"
        }
      ],
      "tips": [
        "프로젝트별: claude mcp add --scope project",
        "--mcp-config으로 JSON 파일에서 여러 서버 한번에 로드 가능"
      ],
      "related_recipes": ["project-setup"]
    },
    {
      "id": "subagent-usage",
      "title": "서브에이전트 활용",
      "category": "고급",
      "difficulty": "advanced",
      "description": "커스텀 서브에이전트를 만들어서 반복 작업 자동화",
      "prerequisites": [],
      "steps": [
        {
          "order": 1,
          "action": "mkdir -p .claude/agents",
          "description": "에이전트 디렉토리 생성",
          "type": "terminal"
        },
        {
          "order": 2,
          "action": ".claude/agents/reviewer.md 파일 작성",
          "description": "에이전트의 역할, 도구 제한, 프롬프트 정의",
          "type": "manual"
        },
        {
          "order": 3,
          "action": "/agents",
          "description": "등록된 에이전트 목록 확인",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "\"reviewer 에이전트로 이 PR 검토해줘\"",
          "description": "특정 에이전트에게 작업 위임",
          "type": "claude-prompt"
        }
      ],
      "tips": [
        "context: fork로 독립 컨텍스트에서 실행하면 메인 대화를 오염시키지 않음",
        "/batch는 내부적으로 서브에이전트 활용 — 먼저 써보고 커스텀으로 넘어가기 추천"
      ],
      "related_recipes": ["parallel-tasks"]
    },
    {
      "id": "context-management",
      "title": "컨텍스트 관리",
      "category": "일상",
      "difficulty": "beginner",
      "description": "대화가 길어졌을 때 컨텍스트를 효율적으로 관리",
      "prerequisites": [],
      "steps": [
        {
          "order": 1,
          "action": "/context",
          "description": "현재 컨텍스트 사용량 시각화",
          "type": "claude-command"
        },
        {
          "order": 2,
          "action": "/cost",
          "description": "토큰 사용량 확인",
          "type": "claude-command"
        },
        {
          "order": 3,
          "action": "/compact \"핵심 작업 내용만 남겨줘\"",
          "description": "불필요한 대화 제거하고 핵심만 보존",
          "type": "claude-command"
        },
        {
          "order": 4,
          "action": "/clear",
          "description": "(대안) 완전히 새로 시작",
          "type": "claude-command",
          "optional": true
        }
      ],
      "tips": [
        "Claude 응답이 느려지면 컨텍스트가 가득 찬 신호",
        "중요한 결정사항은 CLAUDE.md에 기록해두면 /clear 해도 유지됨"
      ],
      "related_recipes": ["project-setup"]
    }
  ]
}
```

---

## 5. CLAUDE.md 템플릿 5종 (전문)

### 5.1 templates/web-app.md

```markdown
# CLAUDE.md — 웹 앱 프로젝트

## 프로젝트 개요
[TODO: 프로젝트가 뭘 하는지 한 줄로]

## 기술 스택
- 프레임워크: [TODO: React/Next.js/Vue/Svelte/Angular]
- 언어: [TODO: TypeScript/JavaScript]
- 스타일링: [TODO: Tailwind/CSS Modules/styled-components/Sass]
- 상태관리: [TODO: Redux/Zustand/Recoil/Pinia/없음]
- API 통신: [TODO: fetch/axios/React Query/SWR]
- 패키지 매니저: [TODO: npm/yarn/pnpm]

## 프로젝트 구조
[TODO: 주요 디렉토리 설명]
src/
├── components/    # 재사용 UI 컴포넌트
├── pages/         # 페이지 (라우팅 단위)
├── hooks/         # 커스텀 훅
├── utils/         # 유틸리티 함수
├── styles/        # 전역 스타일
├── api/           # API 호출 함수
└── types/         # 타입 정의

## 코딩 컨벤션
- 컴포넌트: [TODO: 함수형 컴포넌트, PascalCase 파일명 등]
- 스타일: [TODO: BEM, utility-first 등]
- 타입: [TODO: interface vs type, any 사용 금지 등]
- 임포트 순서: [TODO: 외부 → 내부 → 스타일 → 타입]

## 빌드 & 실행
- 개발 서버: [TODO: npm run dev]
- 빌드: [TODO: npm run build]
- 테스트: [TODO: npm run test]
- 린트: [TODO: npm run lint]

## 주의사항
- [TODO: 절대 수정하면 안 되는 파일/디렉토리]
- [TODO: 환경 변수 파일 (.env*) 건드리지 말 것]
- [TODO: 특정 브라우저 호환성 요구사항]

## 컴포넌트 설계 원칙
- [TODO: atomic design, 컴포넌트 분리 기준 등]
- [TODO: props 네이밍 규칙]
- [TODO: 접근성(a11y) 규칙]
```

### 5.2 templates/api-server.md

```markdown
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
```

### 5.3 templates/data-pipeline.md

```markdown
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
```

### 5.4 templates/monorepo.md

```markdown
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
```

### 5.5 templates/side-project.md

```markdown
# CLAUDE.md — 사이드 프로젝트 (빠른 시작)

## 뭘 만드는지
[TODO: 한 줄로 설명]

## 기술 스택
[TODO: 사용하는 기술 나열]

## 프로젝트 구조
[TODO: 주요 파일/디렉토리만 간단히]

## 실행 방법
- 설치: [TODO: npm install / pip install -r requirements.txt]
- 실행: [TODO: npm run dev / python main.py]
- 테스트: [TODO: npm test / pytest]

## 작업 방향
[TODO: 지금 하고 있는 작업이나 다음에 할 작업]
- [ ] [TODO: 할 일 1]
- [ ] [TODO: 할 일 2]

## 규칙
- [TODO: 코드 스타일 한 줄]
- [TODO: 절대 하지 마 한 줄 (있으면)]
```

---

## 6. agents/korean-helper.md (서브에이전트)

```markdown
---
description: >
  한국어로 Claude Code 사용법을 묻는 사용자를 돕는 서브에이전트.
  커맨드 실행, 워크플로우 안내, CLAUDE.md 작성을 한국어로 지원합니다.
allowed-tools: Read Grep Glob
model: sonnet
---

# 한국어 도우미 에이전트

당신은 Claude Code 한국어 도우미입니다.

## 핵심 원칙
1. **한국어로 응답** — 커맨드/플래그는 영어 원문 유지
2. **실행 먼저, 설명은 최소한** — 가능하면 바로 실행
3. **간결하게** — 장황한 설명 금지

## 데이터 접근
- 커맨드 DB: data/commands.json
- 키워드 매핑: data/aliases.json
- 워크플로우 레시피: data/recipes.json
```

---

## 7. 설치 및 배포

### 7.1 GitHub 기반 설치 (Phase 1)
```bash
# 방법 1: git clone
git clone https://github.com/xsignal/cck.git ~/.claude-plugins/cck
claude --plugin-dir ~/.claude-plugins/cck

# 방법 2: 직접 경로 지정
claude --plugin-dir /path/to/cck
```

### 7.2 마켓플레이스 등록 (Phase 2)
```bash
# 자체 마켓플레이스
claude plugin install cck@xsignal

# 공식 마켓플레이스 (제출 후)
claude plugin install cck@claude-plugins-official
```

---

## 8. 사용 흐름 예시

### 시나리오 1: 자연어로 커맨드 실행
```
사용자: 코드 리뷰해줘

Claude (korean-autopilot 자동 활성화):
  [/review 직접 실행]
  [리뷰 결과 출력]
```

### 시나리오 2: 자연어로 워크플로우 안내
```
사용자: PR 만들어서 올리고 싶어

Claude:
  PR 워크플로우 — 4단계

  **지금 할 것:**
  이 세션에서 바로 작업할 수 있어.
  독립 워크트리에서 안전하게 하려면:
  `claude -w feature-name`

  먼저 어떤 변경을 하고 싶은지 알려줘.
```

### 시나리오 3: 문제 상황 감지
```
사용자: 응답이 점점 느려지는 것 같아

Claude:
  컨텍스트가 많이 찼을 수 있어. 정리할까?

  [사용자: 응]
  [/compact 실행]

  정리했어. 핵심 작업 내용만 남겼어.
```

### 시나리오 4: 커맨드 설명 요청
```
사용자: worktree가 뭐야?

Claude:
  독립된 작업 공간이야. 메인 브랜치를 건드리지 않고
  별도 폴더에서 Claude를 돌릴 수 있어.

  `claude -w 작업이름`

  리팩토링이나 실험적 변경을 안전하게 할 때 유용해.
  💡 --tmux 붙이면 여러 작업을 병렬로 돌릴 수 있어.
```

### 시나리오 5: CLAUDE.md 설정
```
사용자: CLAUDE.md 만들어줘

Claude:
  어떤 프로젝트야?
  1. 웹앱
  2. API 서버
  3. 데이터 파이프라인
  4. 모노레포
  5. 사이드 프로젝트

  [사용자: 2]
  [프로젝트 스캔 → Spring Boot 감지 → 템플릿 생성]

  ✅ CLAUDE.md를 생성했어.
  [TODO] 부분만 채워넣으면 돼.
```

### 시나리오 6: 치트시트 (유일한 명시적 호출)
```
사용자: /cck:치트

Claude:
  [치트시트 테이블 전체 출력]
```

---

## 9. 차별화 정리

| 영역 | Claude Code 공식 | V2 (CCK 커맨드 학습) | **V3 (투명)** |
|------|------------------|---------------------|--------------|
| 사용자 학습 비용 | 영어 커맨드 암기 | CCK 커맨드 암기 | **없음** |
| 커맨드 검색 | `/help` | `/cck:찾기` | **자연어로 말하면 됨** |
| 워크플로우 | 문서 산재 | `/cck:레시피` | **자연어 요청 → 자동 안내** |
| 커맨드 실행 | 직접 타이핑 | 찾고 → 직접 타이핑 | **Claude가 직접 실행** |
| 문제 감지 | 수동 | 수동 | **자동 감지 + 제안** |
| CLAUDE.md | 영어 빈 파일 | `/cck:설정` | `/cck:설정` 또는 **"CLAUDE.md 만들어줘"** |
| 사용자 호출 스킬 수 | - | 6개 | **2개 (치트, 설정)** |

---

## 10. 개발 로드맵

### Phase 1: MVP (1주)
- [ ] plugin.json, 디렉토리 구조 생성
- [ ] `korean-autopilot` SKILL.md (핵심)
- [ ] commands.json — 핵심 20개 커맨드 데이터
- [ ] aliases.json — 한국어 키워드 50개
- [ ] `/cck:치트` SKILL.md
- [ ] README.md (한국어 + 영어)
- [ ] GitHub 레포 공개

### Phase 2: 레시피 + 템플릿 (+1주)
- [ ] recipes.json — 10개 레시피
- [ ] CLAUDE.md 템플릿 5종
- [ ] `/cck:설정` SKILL.md
- [ ] `korean-helper` 에이전트
- [ ] commands.json 전체 커맨드 확장 (50개+)

### Phase 3: 커뮤니티 + 마켓 (+2주)
- [ ] 플러그인 마켓플레이스 등록
- [ ] awesome-claude-code PR
- [ ] 한국 개발자 커뮤니티 공유
- [ ] 기여 가이드 (CONTRIBUTING.md)
- [ ] 공식 마켓플레이스 제출

---

## 11. README.md

```markdown
# 🇰🇷 CCK — Claude Code를 한국어로

> 설치만 하면 한국어로 말하는 것만으로 Claude Code를 완전히 조작할 수 있습니다.
> 새로운 커맨드를 배울 필요 없습니다.

## 설치

### 방법 1: Git Clone
git clone https://github.com/xsignal/cck.git ~/.claude-plugins/cck
claude --plugin-dir ~/.claude-plugins/cck

### 방법 2: 마켓플레이스 (준비 중)
claude plugin install cck@xsignal

## 사용법

### 그냥 한국어로 말하면 됩니다

| 이렇게 말하면 | Claude가 알아서 |
|-------------|----------------|
| "코드 리뷰해줘" | `/review` 실행 |
| "변경사항 보여줘" | `/diff` 실행 |
| "컨텍스트 정리해" | `/compact` 실행 |
| "이전으로 되돌려" | `/rewind` 실행 |
| "비용 얼마 썼어?" | `/cost` 실행 |
| "보안 점검해줘" | `/security-review` 실행 |
| "PR 올리고 싶어" | 워크플로우 단계별 안내 |
| "어제 하던 거 이어서" | `claude -c` 안내 |
| "CLAUDE.md 만들어줘" | 한국어 템플릿 생성 |
| "좀 느려진 것 같아" | 컨텍스트 문제 감지 + 해결 |

### 명시적 호출 (선택사항)

전체 커맨드를 한 눈에 보고 싶을 때:
/cck:치트

CLAUDE.md를 프로젝트 유형별로 생성하고 싶을 때:
/cck:설정

## 지원하는 워크플로우

| 카테고리 | 워크플로우 |
|---------|-----------|
| 일상 | 코드 리뷰, 버그 잡기, 리팩토링, 테스트 작성, 컨텍스트 관리 |
| 프로젝트 | PR 만들기, 병렬 작업, 초기 셋업 |
| 고급 | MCP 연결, 서브에이전트 활용 |

## 기여하기
이슈와 PR 환영합니다! 특히:
- commands.json에 빠진 커맨드 추가
- aliases.json에 한국어 키워드 추가
- 새로운 워크플로우 레시피 제안
- CLAUDE.md 템플릿 추가

## 라이선스
MIT

---

# 🇰🇷 CCK — Korean Autopilot for Claude Code

> Just install and speak Korean. Claude Code handles the rest.

[English documentation coming soon]
```

---

## 12. 라이선스
MIT License
