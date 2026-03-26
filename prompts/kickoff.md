# Harness Kickoff

아래 가이드를 읽고, 이 프로젝트의 harness를 처음부터 끝까지 구축한다.

## 참조 가이드

@harness-prompts/harness-engineering-guide.md

## 진행 방식

### Step 1: 프로젝트 탐색
- 파일 구조, 설정 파일(package.json, pyproject.toml, Cargo.toml 등), 기존 코드를 읽는다
- 관찰 결과를 요약하고, 프로젝트의 목적·기술 스택·구조를 **내가 확인할 수 있도록** 보고한다
- 내 확인을 받은 후 다음 단계로 진행한다

### Step 2: Phase 선택
- 가이드의 Phase 선택 기준에 따라 이 프로젝트에 필요한 Phase를 제안한다
- 형식: `[Required] / [Recommended] / [Skip]` + 각각의 이유
- 내 승인을 받은 후 다음 단계로 진행한다

### Step 3: Phase별 실행
- 승인된 Phase를 순서대로 실행한다
- **Phase 1 (CLAUDE.md):**
  - `/init` 실행 → 가이드 기준으로 편집
  - CLAUDE.md Workflow 섹션에 가이드의 "Starting Development > Session start routine"을 반영한다
  - 부속 파일 생성 (가이드의 "Phase 1 artifacts" 및 관련 섹션 참조):
    - `.claude/memory/harness-candidates.md`
    - `.claude/commands/harness-review.md`
    - `.claude/commands/harness-health-check.md`
- **Phase 2 (Settings):** `.claude/settings.local.json` 생성 — allowedBashCommands, permissions, hooks
- **Phase 3 (Skills & Subagents):** `.claude/skills/`, `.claude/agents/` 에 필요한 파일 생성
- **Phase 4 (Hooks):** `.claude/settings.local.json`에 hook 설정 추가, 필요 시 `.claude/hooks/` 스크립트
- **Phase 5 (MCP):** `.mcp.json` 생성
- **Phase 6 (Code Structure):** 구조 가이드 문서 작성
- **Phase 7 (Working Memory & Parallel):** `.claude/memory/`에 plan, checklist, context notes
- 각 Phase의 산출물을 섹션 단위로 제안하고, 내 확인 후 적용한다
- 하나의 Phase가 끝나면 다음 Phase로 넘어갈지 확인한다

### Step 4: 개발 준비 완료 확인
- 모든 Phase가 끝나면, 생성된 harness 파일 목록을 보여준다
- "harness 구축 완료. 개발을 시작할 준비가 되었습니다" 로 마무리한다

## 규칙
- 가이드의 Interaction Principles와 Coding Discipline을 반드시 따른다
- 확신이 없으면 가정하지 말고 질문한다
- 한 번에 모든 것을 생성하지 않는다 — 섹션 단위로 제안하고 승인을 받는다
