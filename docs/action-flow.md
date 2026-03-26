# 액션 플로우: GitHub → Harness → 개발

## 사전 조건
- Claude Code CLI 설치 완료
- GitHub에 harness-starter-kit 레포 준비 완료
- 프로젝트 디렉토리에 최소 뼈대 존재 (package.json 또는 pyproject.toml 등)

---

## Action 1: 스타터 킷 복사 (1회, 터미널, 10초)

```bash
git clone https://github.com/narnia-ai-harry/harness-starter-kit.git /tmp/harness-kit
mkdir -p harness-prompts
cp /tmp/harness-kit/guide/harness-engineering-guide.md ./harness-prompts/
cp /tmp/harness-kit/prompts/kickoff.md ./harness-prompts/
rm -rf /tmp/harness-kit
```

결과: 프로젝트 루트에 `harness-prompts/` 디렉토리 (파일 2개).

---

## Action 2: Harness 구축 킥오프 (1회, Claude Code)

```bash
claude
```

세션에서:

```
@harness-prompts/kickoff.md
```

**에이전트가 자율 진행합니다. 당신은 판단만 합니다:**

| 에이전트가 하는 것 | 당신이 하는 것 |
|------------------|--------------|
| 프로젝트 탐색 후 관찰 보고 | "맞아" 또는 교정 |
| Phase 선택 제안 | 승인 |
| CLAUDE.md 섹션별 제안 | 섹션별 승인/수정 요청 |
| 부속 파일 생성 (candidates, review command, health-check command) | 확인 |
| 추가 Phase 산출물 제안 | 섹션별 승인/수정 요청 |
| "harness 구축 완료" 보고 | — |

**총 인간 개입: 승인/교정 5-10회. 내용 작성은 전부 에이전트.**

구축 완료 후:

```bash
git add CLAUDE.md .claude/
git commit -m "feat(harness): initial harness setup"
```

세션을 종료한 후 스타터 킷을 정리합니다:

```bash
rm -rf harness-prompts/
git add -A && git commit -m "chore: remove harness starter prompts"
```

---

## Action 3: 일상 개발 (매일, Claude Code)

```bash
claude
```

CLAUDE.md가 자동 로딩됩니다. 자연어로 작업을 지시합니다:

```
auth 모듈의 JWT 검증 로직을 리팩토링해줘.
```

**자동으로 동작하는 것들 (인간 개입 없음):**
- Hooks에 의한 자동 포매팅/린트
- 교정 시 `.claude/memory/harness-candidates.md`에 자동 기록
- 후보 5개 이상이면 리뷰 권장 알림

**병렬 작업:**

```bash
claude -w feature-auth    # 터미널 1
claude -w bugfix-payment  # 터미널 2
```

**이전 세션 이어가기:**

```
/resume
```

---

## Action 4: Harness 리뷰 (주 1회 또는 후보 5개 축적 시)

```
/project:harness-review
```

에이전트가 후보를 분석하고 번호 목록으로 제안합니다. 당신은 번호로 승인/거부만 합니다.

---

## Action 5: 위생 점검 (분기 1회)

```
/project:harness-health-check
```

에이전트가 불필요한 규칙을 찾아 번호 목록으로 제시합니다. 당신은 번호로 승인/거부만 합니다.

---

## 전체 흐름

```
Action 1: git clone + cp  ─────── 1회, 10초
Action 2: @kickoff.md     ─────── 1회, 승인 5-10회
    │
    ▼
Action 3: claude + 자연어  ─────── 매일 (자동: hook, 포매팅, 후보기록)
Action 4: /project:harness-review ── 주 1회 (번호 승인)
Action 5: /project:harness-health-check ── 분기 1회 (번호 승인)
```

## GitHub 레포 구조

```
harness-starter-kit/
├── README.md
├── guide/
│   └── harness-engineering-guide.md
├── prompts/
│   └── kickoff.md
└── docs/
    └── action-flow.md
```

가이드가 참조 지식, kickoff이 실행 트리거, 나머지는 kickoff 과정에서 에이전트가 생성합니다.
