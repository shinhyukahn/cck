# 🇰🇷 CCK — Claude Code를 한국어로

> 설치만 하면 한국어로 말하는 것만으로 Claude Code를 완전히 조작할 수 있습니다.
> 새로운 커맨드를 배울 필요 없습니다.

## 설치

### 방법 1: Git Clone (권장)

```bash
git clone https://github.com/shinhyukahn/cck.git ~/.claude-plugins/cck
```

Claude Code를 실행할 때 플러그인 경로를 지정:

```bash
claude --plugin-dir ~/.claude-plugins/cck
```

또는 Claude Code 설정에 영구 등록:

```bash
claude plugin install ~/.claude-plugins/cck
```

### 방법 2: 로컬 테스트

```bash
git clone https://github.com/shinhyukahn/cck.git
cd cck
claude --plugin-dir .
```

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

```
/cck:cheat
```

CLAUDE.md를 프로젝트 유형별로 생성하고 싶을 때:

```
/cck:setup
/cck:setup 웹앱
/cck:setup api서버
```

## 동작 방식

CCK는 세 가지 스킬로 구성됩니다:

| 스킬 | 역할 |
|------|------|
| `korean-autopilot` | 한국어 입력을 자동 감지해서 적절한 커맨드 실행/안내 (백그라운드 동작) |
| `cheat` | `/cck:cheat` — 한국어 치트시트 즉시 출력 |
| `setup` | `/cck:setup` — 프로젝트 유형에 맞는 CLAUDE.md 한국어 템플릿 생성 |

`korean-autopilot`는 `user-invocable: false`로 설정되어, 사용자가 직접 호출하지 않아도 한국어를 감지하면 자동으로 활성화됩니다.

## 지원하는 워크플로우

| 카테고리 | 워크플로우 |
|---------|-----------|
| 일상 | 코드 리뷰, 버그 잡기, 리팩토링, 테스트 작성, 컨텍스트 관리 |
| 프로젝트 | PR 만들기, 병렬 작업, 초기 셋업 |
| 고급 | MCP 연결, 서브에이전트 활용 |

## 기여하기

이슈와 PR 환영합니다!

| 파일 | 기여 방법 |
|------|---------|
| `data/commands.json` | 빠진 커맨드 추가, 한국어 설명 개선 |
| `data/aliases.json` | 한국어 자연어 표현 추가 |
| `data/recipes.json` | 새로운 워크플로우 레시피 추가 |
| `skills/setup/templates/` | 프로젝트 유형별 CLAUDE.md 템플릿 추가/개선 |

## 라이선스

MIT

---

# 🇰🇷 CCK — Korean Autopilot for Claude Code

> Just install and speak Korean. Claude Code handles the rest.
> No need to learn new commands — just say what you want in Korean.

## Installation

### Option 1: Git Clone (recommended)

```bash
git clone https://github.com/shinhyukahn/cck.git ~/.claude-plugins/cck
```

Then launch Claude Code with the plugin:

```bash
claude --plugin-dir ~/.claude-plugins/cck
```

Or register permanently:

```bash
claude plugin install ~/.claude-plugins/cck
```

### Option 2: Local Testing

```bash
git clone https://github.com/shinhyukahn/cck.git
cd cck
claude --plugin-dir .
```

## How It Works

CCK's `korean-autopilot` skill automatically activates when you speak Korean.
It detects your intent and either executes the right Claude Code command directly
or guides you through multi-step workflows — all in Korean.

| Say this in Korean | Claude does this |
|---|---|
| "코드 리뷰해줘" (code review) | Runs `/review` |
| "변경사항 보여줘" (show changes) | Runs `/diff` |
| "컨텍스트 정리해" (clean up context) | Runs `/compact` |
| "이전으로 되돌려" (revert) | Runs `/rewind` |
| "PR 올리고 싶어" (create PR) | Walks through PR workflow |

## Explicit Commands (Optional)

```
/cck:cheat    — Korean cheatsheet for all Claude Code commands
/cck:setup    — Generate a Korean CLAUDE.md template for your project type
```

## Plugin Structure

```
cck/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── data/
│   ├── commands.json            # Command DB with Korean metadata
│   ├── aliases.json             # Korean natural language → command ID mappings
│   └── recipes.json             # Multi-step workflow recipes
├── skills/
│   ├── korean-autopilot/        # Background auto-activation (user-invocable: false)
│   │   └── SKILL.md
│   ├── cheat/                   # /cck:cheat
│   │   └── SKILL.md
│   └── setup/                   # /cck:setup [project-type]
│       ├── SKILL.md
│       └── templates/
│           ├── web-app.md
│           ├── api-server.md
│           ├── data-pipeline.md
│           ├── monorepo.md
│           └── side-project.md
└── agents/
    └── korean-helper.md
```

## License

MIT
