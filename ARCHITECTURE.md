# Dev Crew — 기술 아키텍처

## 개요

Dev Crew는 Claude Code의 **멀티에이전트 시스템(Multi-Agent System)** 위에서 동작하는 소프트웨어 생성 파이프라인이다. **오케스트레이터-서브에이전트(Orchestrator-Subagent) 패턴**을 사용하며, 각 전문 서브에이전트가 순서대로 협업해 아이디어를 실제 동작하는 코드로 변환한다.

두 가지 모드로 동작한다: 새 프로젝트를 처음 만드는 **신규 개발 모드**와 기존 코드를 수정하는 **유지보수 모드**.

---

## 주요 개념

Dev Crew 설계에 사용된 멀티에이전트 패턴 용어 정리.

| 용어 | 정의 | Dev Crew에서의 역할 |
|------|------|------|
| **Orchestrator** | 전체 흐름을 지휘하는 중앙 LLM. 태스크를 분해하고 서브에이전트에 위임한다 | `CLAUDE.md` (메인 Claude Code 세션) |
| **Subagent** | 오케스트레이터 지시 아래 독립된 컨텍스트로 동작하는 전문 에이전트 | `interviewer`, `pm`, `architect`, `design`, `frontend`, `backend`, `reviewer`, `qa` |
| **Agentic Loop** | "gather context → take action → verify work → repeat" 패턴 | Frontend/Backend의 TDD Red→Green 사이클 (단위 테스트당 최대 10회 재시도) |
| **Human in the Loop** | 특정 체크포인트에서 사람의 피드백을 기다리는 구조 | 요구사항/설계/디자인 확인 단계 |
| **Context Isolation** | 각 서브에이전트가 독립된 컨텍스트 윈도우를 사용해 정보 오염을 방지 | 에이전트 간 통신을 `docs/*.md` 파일로만 수행 |
| **Workflow** | LLM과 도구가 사전 정의된 순서로 실행되는 구조 | Dev Crew 전체 (CLAUDE.md가 순서 고정) |

---

## 아키텍처 구조

### 신규 개발 모드

```
┌─────────────────────────────────────────────────────────┐
│                    Orchestrator                          │
│                    (CLAUDE.md)                           │
│  - 모드 판단 (신규 / 유지보수)                             │
│  - 파이프라인 순서 제어                                   │
│  - Human-in-the-loop 체크포인트 관리                      │
└──────────────────────────┬──────────────────────────────┘
                           │ Agent Tool 호출
                           ▼
                    [interviewer]          ✋ Step 0
                    ↔ 아이디어 방향 결정 + 환경 확인
                    Read, Write, AskUserQuestion (Opus)
                           │
                           ▼
                       docs/brief.md
                           │
         ┌─────────────────┼──────────────────┐
         ▼                 ▼                  ▼
    [pm]            [architect]          [design]
    ↔ 사용자 인터뷰   Read, Write          Read, Write
    Read, Write      기술 스택/API/DB 설계  톤앤매너/디자인 토큰
    AskUserQuestion  (Opus)               + Section 14 구현 가이드
         │                 │                  │
         ▼                 ▼                  ▼
  requirements.md   architecture.md   design-system.md
         └─────────────────┴──────────────────┘
                           │ (docs/ = 에이전트 간 공유 메모리)
         ┌─────────────────┴──────────────────┐
         ▼                                    ▼
    [frontend]                           [backend]
    Read, Write, Bash                    Read, Write, Bash
    TDD Red→Green (단위당 최대 10회)      TDD Red→Green (라우터당 최대 10회)
    src/[PROJECT_NAME]/                  src/[PROJECT_NAME]/backend/ (옵션)
         └─────────────────┬──────────────────┘
                           ▼
                      [reviewer]
                      Read, Bash (grep/find만)
                      코드 리뷰 (완성도·품질·TDD 준수)
                      docs/review.md
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        NEEDS_REVISION                PASS
        (최대 2회, Step 6-A)           │
              │                         ▼
              └──────────────────▶  [qa]
                                   Read, Write, Bash
                                   서버 기동 → Playwright E2E
                                   docs/qa-report.md
                                        │
                           ┌────────────┴────────────┐
                           ▼                         ▼
                     NEEDS_REVISION                PASS
                     (최대 2회, Step 6-B)           │
                           │                         ▼
                           └──────────────────▶   완료
```

### 유지보수 모드

```
변경 요청 감지 (src/ 존재 + 수정/추가/버그 요청)
                           │
                           ▼
                         [pm]
                   ↔ 변경 범위 인터뷰
                   docs/maintenance-request.md
                           │
                    ✋ 범위 확인
                           │
         ┌─────────────────┴──────────────────┐
         ▼                                    ▼
    [frontend]                           [backend]
    기존 코드 읽기 → 최소 변경              기존 코드 읽기 → 최소 변경
    Change Spec 항목만 수정                Change Spec 항목만 수정
         └─────────────────┬──────────────────┘
                           ▼
                      [reviewer]
                      변경 구현 확인 (Change Spec 이행 여부)
                      파일 범위 준수 여부 (관련 파일만 수정했는지)
                           │
                    ✋ QA 실행 여부 확인
                           │
                    (사용자 선택)
                           ▼
                         [qa]  (선택적)
                   서버 기동 → Playwright E2E
```

---

## 에이전트 명세

### Orchestrator — `CLAUDE.md`

오케스트레이터는 별도 프로세스가 아니라 Claude Code의 메인 세션 자체다. `CLAUDE.md`가 시스템 프롬프트 역할을 하며 파이프라인 전체 흐름을 정의한다.

- **도구**: 모든 도구 (Agent Tool로 서브에이전트 호출)
- **책임**: 모드 판단, 단계 순서 제어, Human-in-the-loop 체크포인트, 서브에이전트 출력 요약

### Subagents — `.claude/agents/`

| 에이전트 | 페르소나 | 입력 | 출력 | 허용 도구 | 역할 |
|---|---|---|---|---|---|
| `interviewer` | CEO | 사용자 아이디어 (직접 입력) | `docs/brief.md` | Read, Write, AskUserQuestion | 아이디어 방향 결정 + 환경 확인 → brief 작성 (Opus) |
| `pm` | Product Manager | `docs/brief.md` 또는 `docs/maintenance-request.md` | `docs/requirements.md` 또는 Change Spec | Read, Write, AskUserQuestion | 요구사항 인터뷰 → 확정 (Opus) |
| `architect` | CTO | `docs/requirements.md` | `docs/architecture.md` | Read, Write | 기술 스택/API/DB 설계 (Opus) |
| `design` | Product Designer | `docs/requirements.md`, `docs/architecture.md` | `docs/design-system.md` | Read, Write | 톤앤매너·디자인 토큰·Section 14 구현 가이드 정의 (Sonnet) |
| `frontend` | Frontend Engineer | `docs/architecture.md`, `docs/design-system.md` 또는 Change Spec | `src/[PROJECT_NAME]/` | Read, Write, Bash | TDD Red→Green 구현 / 최소 변경 유지보수 (Sonnet) |
| `backend` | Backend Engineer | `docs/architecture.md` 또는 Change Spec | `src/[PROJECT_NAME]/backend/` | Read, Write, Bash | TDD Red→Green 구현 / 최소 변경 유지보수 (Sonnet) |
| `reviewer` | Engineering Manager | `src/`, `docs/architecture.md`, `docs/requirements.md` | `docs/review.md` | Read, Bash (grep/find) | 코드 리뷰 — 완성도·품질·TDD 준수 확인 (Sonnet) |
| `qa` | QA Lead | `docs/requirements.md`, 프로젝트 스크립트 | `docs/qa-report.md` | Read, Write, Bash | 서버 기동 → Playwright E2E → 골든 패스 검증 (Sonnet) |

**도구 제한 이유**:
- `interviewer`/`pm`: 사용자 인터뷰를 위해 `AskUserQuestion` 보유
- `architect`/`design`: 파일 읽기/쓰기만 필요하므로 `Bash` 제외 — 불필요한 시스템 명령 실행 차단
- `reviewer`: 코드 리뷰 전용 — `grep`/`find` 수준의 `Bash`만 허용, 빌드/테스트 실행 금지

---

## 설계 결정

### 1. docs/ 를 공유 메모리로 사용

각 서브에이전트는 독립된 컨텍스트 윈도우를 가지므로 직접 통신이 불가능하다. `docs/*.md` 파일이 에이전트 간 유일한 정보 전달 경로 역할을 한다.

```
pm → requirements.md → architect → architecture.md → frontend
design → design-system.md → frontend (Section 14 구현 가이드)
pm → maintenance-request.md → frontend/backend (유지보수)
```

이 구조 덕분에:
- 어느 단계에서든 중간 산출물을 사람이 읽고 수정할 수 있다
- 에이전트 하나가 실패해도 파일 기반으로 재실행이 가능하다
- 각 에이전트의 컨텍스트가 오염되지 않는다

### 2. 두 단계 인터뷰 — Interviewer + PM

`interviewer`는 아이디어를 듣고 방향을 결정한다. 선택지를 나열하지 않고 "이게 맞다, 이 방향으로 간다"고 결단력 있게 정리하는 CEO 역할이다. 환경·스택·포트 확인 후 `docs/brief.md`를 작성한다.

`pm`은 브리프를 읽고 요구사항 인터뷰를 이어받는다. "무엇을 만들지"를 확정하는 단계로, 최대 2라운드로 제한해 무한 대화를 방지한다.

두 에이전트 모두 `AskUserQuestion` 도구로 사용자와 직접 소통하며, 두 단계의 대화가 끝나야 PRD 작성이 시작된다.

### 3. Human-in-the-Loop 배치

신규 개발: 요구사항 → 설계 → 디자인 3회 강제 체크포인트. 코드 생성 이전에 방향을 바로잡는 비용이 코드 생성 후 수정 비용보다 훨씬 낮기 때문이다.

유지보수: 변경 범위 확인 1회 + QA 실행 여부 확인 1회. Architect/Design을 다시 돌리지 않아 불필요한 재작업을 방지한다.

### 4. TDD Agentic Loop — Frontend / Backend

Frontend와 Backend는 단위 기능을 구현할 때마다 **Red→Green 사이클**을 반복한다.

```
테스트 작성 → 실패 확인(Red) → 구현 → 통과 확인(Green) → 다음 단위
                                  ↑
                           실패 시 최대 10회 재시도
                           10회 초과 → 중단 + 실패 리포트
```

이 방식으로:
- 구현 완료 시점에 테스트가 이미 통과 상태다
- Reviewer가 테스트 파일 존재 여부와 커버리지를 grep으로 확인 가능하다
- 해결 불가 실패는 조기에 차단되어 Reviewer/QA에서 반복 실패하는 낭비를 막는다

### 5. Reviewer와 QA의 역할 분리

두 에이전트는 서로 다른 관점에서 검증한다:

| 에이전트 | 관점 | 방법 | 산출물 |
|---|---|---|---|
| **Reviewer** | 코드 품질 | grep/find로 정적 분석 | `docs/review.md` |
| **QA** | 사용자 경험 | Playwright E2E 동적 테스트 | `docs/qa-report.md` |

Reviewer가 PASS를 내야 QA로 진행한다. 각각 독립적인 NEEDS_REVISION 루프를 가진다 (최대 2회).

### 6. 모델 배분 (Opus / Sonnet)

방향 결정 단계는 Opus, 실행 단계는 Sonnet을 사용한다.

| 모델 | 에이전트 | 이유 |
|------|------|------|
| Opus | `interviewer`, `pm`, `architect` | 방향 판단, 요구사항 해석, 기술 설계 — 추론 품질이 결과물 전체에 영향 |
| Sonnet | `design`, `frontend`, `backend`, `reviewer`, `qa` | PRD를 실행하는 단계 — 속도와 비용 효율 우선 |

### 7. 모드 분기

`src/` 아래 프로젝트 디렉터리(`src/[PROJECT_NAME]/`)의 존재 여부 + 요청 내용으로 모드를 판단한다. 유지보수 모드는 Interviewer/Architect/Design을 건너뛰고 PM → Frontend/Backend → Reviewer만 실행해 불필요한 재설계를 막는다.

---

## 이 시스템으로 볼 수 있는 것

| 관찰 포인트 | 확인 방법 |
|---|---|
| 오케스트레이터가 서브에이전트에 어떤 프롬프트를 전달하는지 | Claude Code 세션 로그 |
| PM 인터뷰 라운드 수와 질문 패턴 | 에이전트 실행 중 AskUserQuestion 호출 내역 |
| 각 에이전트의 컨텍스트 윈도우 사용량 | 에이전트 실행 중 토큰 카운터 |
| 에이전트 간 정보 손실 여부 | `docs/*.md` 파일을 각 단계 후 직접 확인 |
| TDD Red→Green 사이클 동작 | Frontend/Backend Bash 실행 로그 |
| Reviewer 코드 리뷰 결과 | `docs/review.md`의 이슈 목록 |
| QA E2E 시나리오별 결과 | `docs/qa-report.md`의 PASS/FAIL 표 |
| 유지보수 시 변경 범위 준수 여부 | `docs/review.md`의 Maintenance Check 섹션 |

---

## 파일 구조

```
claude-architect/
├── CLAUDE.md               ← 오케스트레이터 (신규 개발 + 유지보수 모드 정의)
├── README.md               ← 사용자 가이드
├── ARCHITECTURE.md         ← 이 문서
├── .claude/agents/         ← 서브에이전트 정의
│   ├── interviewer.md      ← Step 0: 아이디어 방향 결정, CEO (Opus)
│   ├── pm.md               ← 요구사항 인터뷰, Product Manager (Opus)
│   ├── architect.md        ← 기술 설계, CTO (Opus)
│   ├── design.md           ← 디자인 시스템 + Section 14 구현 가이드, Product Designer (Sonnet)
│   ├── frontend.md         ← TDD 구현 / 신규·유지보수 분기, Frontend Engineer (Sonnet)
│   ├── backend.md          ← TDD 구현 / 신규·유지보수 분기, Backend Engineer (Sonnet)
│   ├── reviewer.md         ← 코드 리뷰 (완성도·품질·TDD 준수), Engineering Manager (Sonnet)
│   └── qa.md               ← 서버 기동 → Playwright E2E, QA Lead (Sonnet)
├── docs/                   ← 에이전트 간 공유 메모리 (런타임 생성)
│   ├── brief.md            ← interviewer 출력 (PROJECT_NAME 포함)
│   ├── requirements.md
│   ├── architecture.md
│   ├── design-system.md
│   ├── maintenance-request.md   ← 유지보수 모드 시 생성
│   ├── review.md           ← reviewer 출력 (코드 리뷰 결과)
│   └── qa-report.md        ← qa 출력 (E2E 결과)
└── src/
    └── [PROJECT_NAME]/     ← 최종 산출물 (런타임 생성)
        │                      Next.js 풀스택: app/, components/ 등 직접 위치
        ├── frontend/          React+Vite 선택 시
        └── backend/           React+Vite+FastAPI 선택 시
```
