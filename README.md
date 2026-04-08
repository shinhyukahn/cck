# CCK — Claude Code Korean

> **한국어로 말하면 Claude Code가 알아서 실행합니다.**
> 새로운 명령어를 외울 필요 없습니다. 설치 후 바로 한국어로 사용하세요.

---

## 목차

- [소개](#소개)
- [설치](#설치)
- [사용법](#사용법)
- [스킬 상세](#스킬-상세)
- [지원 워크플로우](#지원-워크플로우)
- [플러그인 구조](#플러그인-구조)
- [기여하기](#기여하기)
- [라이선스](#라이선스)

---

## 소개

CCK는 한국어 사용자가 Claude Code를 자연스럽게 사용할 수 있게 해주는 **투명한 플러그인**입니다.

- `korean-autopilot` 스킬이 백그라운드에서 자동으로 동작
- 사용자가 한국어로 입력하면 적절한 Claude Code 커맨드를 자동으로 실행하거나 안내
- 별도의 명령어 학습 불필요 — 그냥 말하면 됨

### 동작 원리

```
사용자 입력 (한국어)
        ↓
korean-autopilot 자동 감지
        ↓
aliases.json → 키워드 매핑
commands.json → 커맨드 정보 조회
recipes.json → 멀티스텝 워크플로우 참조
        ↓
실행 가능 커맨드 → 직접 실행
터미널 커맨드   → 복사 가능한 코드 블록으로 안내
멀티스텝 작업  → 단계별 순서 안내
```

---

## 설치

### 요구사항

- [Claude Code](https://claude.ai/code) 최신 버전

### 방법 1: 마켓플레이스 (권장)

공식 마켓플레이스에 등록되면 한 줄로 설치 가능합니다:

```bash
claude plugin install cck@shinhyukahn
```

> 공식 마켓플레이스 등록 전까지는 아래 방법을 사용하세요.

### 방법 2: 자체 마켓플레이스

먼저 마켓플레이스를 등록합니다 (최초 1회):

```bash
claude plugin marketplace add shinhyukahn/cck
```

이후 설치:

```bash
claude plugin install cck@shinhyukahn
```

### 방법 3: Git Clone 후 직접 실행

```bash
git clone https://github.com/shinhyukahn/cck.git
cd cck
claude --plugin-dir .
```

> **Windows 사용자:** 절대 경로(`C:\path\to\cck`)는 동작하지 않을 수 있습니다. 반드시 `cck` 디렉토리 안에서 `claude --plugin-dir "."` 로 실행하세요.

### 설치 확인

Claude Code 세션에서 `/cck:cheat`를 입력했을 때 한국어 치트시트가 출력되면 정상입니다.

---

## 사용법

### 그냥 한국어로 말하면 됩니다

CCK가 설치된 상태에서 한국어로 입력하면 자동으로 처리됩니다.

| 이렇게 말하면 | Claude가 알아서 |
|---|---|
| "코드 리뷰해줘" | `/review` 실행 |
| "변경사항 보여줘" | `/diff` 실행 |
| "컨텍스트 정리해" | `/compact` 실행 |
| "이전으로 되돌려" | `/rewind` 실행 |
| "비용 얼마야?" | `/cost` 실행 |
| "보안 취약점 점검해줘" | `/security-review` 실행 |
| "다른 모델로 바꿔줘" | `/model` 실행 |
| "어제 하던 거 이어서 하고 싶어" | `claude -c` 터미널 커맨드로 안내 |
| "독립된 공간에서 작업하고 싶어" | `claude -w 작업명` 안내 |
| "PR 올리고 싶어" | PR 워크플로우 단계별 안내 |
| "여러 작업 동시에 하고 싶어" | 병렬 워크트리 워크플로우 안내 |
| "응답이 느려진 것 같아" | 컨텍스트 문제 감지 → `/compact` 제안 |
| "CLAUDE.md 만들어줘" | `/cck:setup` 실행 안내 |

### 명시적 호출 명령어

```
/cck:cheat
```
Claude Code 한국어 치트시트를 즉시 출력합니다. 자주 쓰는 커맨드와 단축키를 한눈에 볼 수 있습니다.

```
/cck:setup
/cck:setup 웹앱
/cck:setup api서버
```
현재 프로젝트에 맞는 `CLAUDE.md` 한국어 템플릿을 생성합니다. 프로젝트 파일을 자동 감지해서 기술 스택을 채워넣습니다.

---

## 스킬 상세

### `korean-autopilot` (자동 활성화)

`user-invocable: false`로 설정되어 사용자가 직접 호출하지 않아도 백그라운드에서 자동 동작합니다.

한국어 입력을 감지하면 아래 네 가지 패턴 중 하나로 응답합니다.

**패턴 A — 세션 내 실행 가능한 커맨드**

`/compact`, `/diff`, `/review`, `/rewind`, `/cost` 등 Claude Code 세션 안에서 직접 실행할 수 있는 커맨드는 바로 실행합니다.

**패턴 B — 터미널 커맨드 안내**

`claude -c`, `claude -w`, `claude mcp add` 등 새 터미널 세션에서 입력해야 하는 커맨드는 복사 가능한 코드 블록으로 안내합니다.

```
[한 줄 설명]

터미널에서 실행해:
`[커맨드]`
```

**패턴 C — 멀티스텝 워크플로우**

PR 만들기, 병렬 작업, MCP 연결 등 여러 단계가 필요한 작업은 `recipes.json`을 참조해서 현재 단계를 실행하고 다음 단계를 안내합니다.

**패턴 D — 문제 상황 선제 감지**

사용자가 요청하지 않아도 문제 신호를 감지하면 먼저 제안합니다.

| 신호 | 제안 |
|---|---|
| "응답이 느려졌어", "이상하게 대답해" | `/compact` 제안 |
| "아까 한 거 되돌리고 싶어" | `/rewind` 제안 |
| "토큰 많이 쓰고 있어?" | `/cost` 실행 |
| 새 주제로 전환 감지 | `/compact`로 정리 제안 |

---

### `cheat` — `/cck:cheat`

포함 내용:
- 시작 커맨드 (`claude`, `claude -c`, `claude -r` 등)
- 세션 커맨드 (`/compact`, `/diff`, `/rewind`, `/cost` 등)
- 프로젝트 관리 (`/init`, `/memory`, `/permissions`, `/plugin`)
- 코드 작업 (`/review`, `/security-review`, `/simplify`, `/batch`)
- 실행 플래그 (`--worktree`, `--effort`, `--tmux`, `--chrome` 등)
- 키보드 단축키

---

### `setup` — `/cck:setup [프로젝트유형]`

현재 프로젝트 디렉토리를 탐색해서 기술 스택을 자동 감지하고, 프로젝트 유형에 맞는 한국어 `CLAUDE.md` 템플릿을 생성합니다.

| 유형 | 감지 파일 |
|---|---|
| 웹앱 | `package.json` + React/Next.js/Vue 등 |
| API 서버 | `pom.xml`, `package.json` + Express, `requirements.txt` + FastAPI 등 |
| 데이터 파이프라인 | `requirements.txt`, `Pipfile`, `pyproject.toml` |
| 모노레포 | `pnpm-workspace.yaml`, `lerna.json`, `nx.json` |
| 사이드 프로젝트 | 감지 파일 없을 때 기본값 |

인자 없이 `/cck:setup`만 입력하면 유형을 물어봅니다.

---

## 지원 워크플로우

`data/recipes.json`에 정의된 워크플로우입니다. 한국어로 말하면 `korean-autopilot`이 해당 레시피를 참조해서 단계별로 안내합니다.

| 워크플로우 | 카테고리 | 난이도 |
|---|---|---|
| 코드 리뷰 받기 | 일상 | 초급 |
| 버그 잡기 | 일상 | 초급 |
| 리팩토링 | 일상 | 중급 |
| 테스트 작성 | 일상 | 초급 |
| 컨텍스트 관리 | 일상 | 초급 |
| PR 만들어서 올리기 | 프로젝트 관리 | 중급 |
| 여러 작업 병렬 처리 | 프로젝트 관리 | 고급 |
| 프로젝트 초기 셋업 | 프로젝트 관리 | 초급 |
| MCP 서버 연결 | 고급 | 고급 |
| 서브에이전트 활용 | 고급 | 고급 |

---

## 플러그인 구조

```
cck/
├── .claude-plugin/
│   ├── plugin.json              # 플러그인 매니페스트 (이름, 버전, 저자, 경로)
│   └── marketplace.json         # 자체 마켓플레이스 등록 정보
├── data/
│   ├── commands.json            # Claude Code 커맨드 DB (한국어 설명 포함)
│   ├── aliases.json             # 한국어 자연어 키워드 → 커맨드 ID 매핑
│   └── recipes.json             # 멀티스텝 워크플로우 레시피 정의
├── commands/
│   ├── cheat.md                 # /cck:cheat — 치트시트 출력
│   ├── setup.md                 # /cck:setup — CLAUDE.md 템플릿 생성
│   └── templates/
│       ├── web-app.md
│       ├── api-server.md
│       ├── data-pipeline.md
│       ├── monorepo.md
│       └── side-project.md
├── skills/
│   └── korean-autopilot/        # 백그라운드 자동 활성화 스킬
│       └── SKILL.md
└── agents/
    └── korean-helper.md         # 한국어 서브에이전트 정의
```

### 데이터 파일 스키마

**`commands.json`** 핵심 필드:

```jsonc
{
  "id": "compact",
  "type": "slash",              // "slash" | "flag" | "terminal"
  "name": "/compact",
  "summary_ko": "...",          // 한 줄 한국어 설명
  "detail_ko": "...",           // 상세 한국어 설명
  "session_executable": true,   // true → 세션 내 직접 실행, false → 터미널 안내
  "tags": ["정리", "컨텍스트"], // 자연어 검색 키워드
  "category": "대화관리",
  "difficulty": "beginner"
}
```

**`aliases.json`** 구조:

```jsonc
{
  "이어서": ["continue-flag", "resume"],
  "정리":   ["compact", "clear"],
  "느려":   ["compact", "effort-flag", "fast"]
}
```

**`recipes.json`** 스텝 타입:

| 타입 | 설명 |
|---|---|
| `terminal` | 터미널에서 실행하는 커맨드 (복사 가능하게 안내) |
| `claude-command` | Claude Code 세션 내 슬래시 커맨드 (직접 실행) |
| `claude-prompt` | 사용자가 Claude에게 입력할 프롬프트 예시 |
| `manual` | 사용자가 직접 수행해야 하는 작업 |
| `auto` | Claude가 자동으로 처리하는 단계 |

---

## 기여하기

이슈와 PR 환영합니다.

| 영역 | 방법 |
|---|---|
| 새 커맨드 추가 | `data/commands.json`에 커맨드 객체 추가 |
| 자연어 표현 추가 | `data/aliases.json`에 키워드 → 커맨드 ID 추가 |
| 새 워크플로우 | `data/recipes.json`에 레시피 추가 |
| CLAUDE.md 템플릿 | `commands/templates/`에 `.md` 파일 추가 |
| 버그 수정 | 이슈 등록 후 PR |

### 로컬 테스트

```bash
git clone https://github.com/shinhyukahn/cck.git
cd cck
claude --plugin-dir .
```

---

## 라이선스

MIT © [shinhyukahn](https://github.com/shinhyukahn)

---

# CCK — Claude Code Korean (English)

> **Speak Korean. Claude Code handles the rest.**
> No commands to memorize. Just install and talk naturally in Korean.

## What It Does

CCK is a transparent plugin for Claude Code. Once installed, its `korean-autopilot` skill runs silently in the background. When it detects Korean input, it automatically routes to the right Claude Code command — executing it directly when possible, or providing a ready-to-copy terminal command when not.

## Installation

```bash
# Via marketplace (once listed)
claude plugin install cck@shinhyukahn

# Via self-hosted marketplace
claude plugin marketplace add shinhyukahn/cck
claude plugin install cck@shinhyukahn

# Local (confirmed working)
git clone https://github.com/shinhyukahn/cck.git
cd cck
claude --plugin-dir .
```

> **Windows:** Absolute paths may not work with `--plugin-dir`. Run from inside the `cck` directory using `claude --plugin-dir "."`.

## How It Works

The `korean-autopilot` skill (`user-invocable: false`) activates automatically when Korean is detected. It references three data files:

- `data/commands.json` — full command DB with Korean metadata and `session_executable` flag
- `data/aliases.json` — natural language keyword → command ID mappings
- `data/recipes.json` — multi-step workflow recipes

Commands with `session_executable: true` are executed directly. Commands with `session_executable: false` are shown as copyable terminal instructions.

## Explicit Commands

```
/cck:cheat          Korean cheatsheet for all Claude Code commands
/cck:setup          Generate a Korean CLAUDE.md for the current project
/cck:setup web-app  Skip the prompt and generate a specific template
```

## Plugin Structure

```
cck/
├── .claude-plugin/
│   ├── plugin.json             # Manifest
│   └── marketplace.json        # Self-hosted marketplace registry
├── data/
│   ├── commands.json           # Command DB
│   ├── aliases.json            # NL keyword mappings
│   └── recipes.json            # Workflow recipes
├── commands/
│   ├── cheat.md                # /cck:cheat
│   ├── setup.md                # /cck:setup
│   └── templates/              # CLAUDE.md templates
├── skills/
│   └── korean-autopilot/       # Auto-activated background skill
│       └── SKILL.md
└── agents/
    └── korean-helper.md
```

## License

MIT © [shinhyukahn](https://github.com/shinhyukahn)
