---
name: cheat
description: >
  Claude Code 핵심 커맨드를 한 눈에 볼 수 있는 한국어 치트시트.
  Use when the user asks for a cheatsheet, quick reference, or "전체 명령어 보여줘".
---

아래 치트시트를 그대로 출력해. 어떤 말도 앞뒤로 붙이지 마. 인사도, 설명도, 요약도 없이 표만 출력하고 끝내.

---

# ⚡ Claude Code 치트시트

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
