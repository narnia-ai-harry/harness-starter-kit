# Harness Starter Kit

Claude Code 프로젝트의 harness를 빠르게 구축하기 위한 스타터 킷.

## Quick Start

```bash
# 1. 스타터 킷에서 필요한 파일만 복사
git clone https://github.com/narnia-ai-harry/harness-starter-kit.git /tmp/harness-kit
mkdir -p harness-prompts
cp /tmp/harness-kit/guide/harness-engineering-guide.md ./harness-prompts/
cp /tmp/harness-kit/prompts/kickoff.md ./harness-prompts/
rm -rf /tmp/harness-kit

# 2. Claude Code 실행 후 킥오프
claude
```

세션 안에서:
```
@harness-prompts/kickoff.md
```

에이전트가 가이드를 읽고 → 프로젝트 탐색 → Phase 선택 → 구축 → 개발 준비 완료까지 진행합니다.

## 구조

```
harness-starter-kit/
├── README.md
├── guide/
│   └── harness-engineering-guide.md   ← 전체 참조 가이드
├── prompts/
│   ├── kickoff.md                     ← 최초 1회: harness 구축 킥오프
│   └── examples/
│       └── merge-worktree-SKILL.md    ← Phase 7 참조: 워크트리 머지 스킬
└── docs/
    └── action-flow.md                 ← 상세 액션 플로우 (사용자 가이드)
```

## 킥오프가 생성하는 것

| 파일 | 용도 |
|------|------|
| `CLAUDE.md` | 프로젝트 컨텍스트 (매 세션 자동 로딩) |
| `.claude/memory/harness-candidates.md` | harness 개선 후보 버퍼 |
| `.claude/commands/harness-review.md` | 주기적 harness 리뷰 (`/project:harness-review`) |
| `.claude/commands/harness-health-check.md` | 분기별 위생 점검 (`/project:harness-health-check`) |
| `.claude/settings.local.json` | 프로젝트 설정 (Phase 2 선택 시) |
| `.claude/skills/`, `.claude/agents/` | Skills, Subagents (Phase 3 선택 시) |

## 킥오프 후 정리

harness 구축이 완료되고 **세션을 종료한 후**, `harness-prompts/`를 삭제합니다.
이후 운영은 CLAUDE.md와 `.claude/commands/`가 담당합니다.

```bash
rm -rf harness-prompts/
git add -A && git commit -m "chore: remove harness starter prompts"
```

## 일상 운영 요약

| 시점 | 명령 |
|------|------|
| 매일 개발 | `claude` → 자연어로 작업 지시 (CLAUDE.md가 자동 로딩) |
| 주 1회 리뷰 | `/project:harness-review` |
| 분기 1회 점검 | `/project:harness-health-check` |
