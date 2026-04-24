# Dev Crew

아이디어를 입력하면 멀티에이전트 파이프라인이 실제 동작하는 코드로 변환해준다.

인터뷰 → 요구사항 → 설계 → 디자인 → 구현 → 리뷰 → QA까지 워터폴 기반으로 순서대로 진행된다. 각 단계마다 사용자 확인을 거쳐 방향을 잡고, AI가 나머지를 완성한다.

---

## 빠른 시작

```bash
cd claude-architect
claude
```

아이디어를 자유롭게 입력한다.

```
할 일 관리 앱 만들어줘
```

```
유튜브 링크 붙여넣으면 요약해주는 웹앱
```

```
팀원들이 점심 메뉴 투표할 수 있는 사이트
```

---

## 파이프라인

```
✋ Step 0     Interviewer   아이디어 반응 + 사업성 평가 + 방향 확정
                                      docs/brief.md
       ↓
     PM          요구사항 인터뷰          docs/requirements.md
       ↓
✋ Step 1.5   요구사항 확인
       ↓
     Architect   PRD + 기술 설계          docs/PRD.md
       ↓
✋ Step 3.5   설계 확인
       ↓
     Design      UI/UX + 디자인 시스템    docs/design-system.md
       ↓
✋ Step 3.7   디자인 확인
       ↓
     Fullstack   TDD 구현                src/[PROJECT_NAME]/
       ↓
     Reviewer    코드 품질·완성도 검토     docs/review.md
       ↓
     NEEDS_REVISION → 수정 → Reviewer 재실행 (최대 2회)  /  PASS ↓
       ↓
     QA          E2E 테스트              docs/qa-report.md
       ↓
     NEEDS_REVISION → 수정 → QA 재실행 (최대 2회)  /  PASS ↓
       ↓
     완료 → 실행 안내
```

✋ 표시된 단계에서 파이프라인이 멈추고 사용자 확인을 기다린다.

---

## 에이전트 설명

### Interviewer
아이디어를 처음 듣는 에이전트. 세 가지를 한 번에 한다.

1. **아이디어 반응** — 흥미로운 점을 짚고, 약한 부분을 솔직하게 지적한다.
2. **사업성 평가** — "누가 돈을 내는가", "시장이 있는가", "가장 위험한 가정은 무엇인가"를 2~3문장으로 판단한다.
3. **방향 확정** — 사용자에게 그대로 진행할지 피벗할지 물어보고 확정한다.

확정 후 `docs/brief.md`를 작성한다.

---

### PM (Product Manager)
`docs/brief.md`를 읽고 사용자와 직접 대화해 요구사항을 정리한다. 첫 질문은 프로젝트 이름이며, 2~4개 요구사항 질문을 함께 던진다. 최대 2라운드 인터뷰 후 `docs/requirements.md`를 작성한다.

---

### Architect
`docs/requirements.md`를 읽고 풀스택 에이전트가 추가 결정 없이 바로 구현할 수 있는 PRD를 작성한다.

스택은 **Next.js 16 풀스택**으로 고정. PRD에 아래 내용을 모두 포함한다:
- 제품 컨텍스트 (Vision, Users, Scenarios, MVP Features, Out of Scope)
- Tech Stack + Dependency Versions (확정 버전 목록)
- Directory Structure, Data Model, API Contract
- Component Inventory, Test Plan

---

### Design
`docs/requirements.md`와 `docs/PRD.md`를 읽고 디자인 시스템을 정의한다. 레퍼런스 이미지가 있으면 색상·레이아웃·분위기를 반영한다.

톤앤매너 옵션 2~3개 → 컨셉 방향 2~3개 → 최적 방향 선택 → 컬러·타이포·스페이싱·라디우스 토큰 자동 정의. 결과를 `docs/design-system.md`에 작성한다.

---

### Fullstack
`docs/PRD.md`와 `docs/design-system.md`를 읽고 Next.js 16 App Router 풀스택 앱을 TDD로 구현한다.

- 백엔드(Route Handler, Service, Repository)와 프론트엔드(컴포넌트, 훅) 모두 테스트 먼저 작성
- PRD의 Directory Structure, Dependency Versions를 그대로 따름
- `npm run test` + `npm run lint` + `npm run build` 모두 통과 후 완료

---

### Reviewer
구현된 코드를 `docs/PRD.md`, `docs/requirements.md`, `docs/design-system.md` 기준으로 검토한다. Out of Scope 항목은 미구현이 정상이므로 지적하지 않는다. 실제 빌드·테스트·린트를 직접 실행해 결과를 `docs/review.md`에 작성한다.

STATUS: `PASS` 또는 `NEEDS_REVISION`

---

### QA
Reviewer가 PASS를 낸 뒤 실행된다. 서버를 직접 기동하고 Playwright로 `docs/requirements.md`의 Acceptance Criteria를 E2E 테스트한다. 서버 종료 후 결과를 `docs/qa-report.md`에 작성한다.

---

## 사전 요구사항

| 항목        | 버전  | 확인                                       |
| ----------- | ----- | ------------------------------------------ |
| Node.js     | 20+   | `node --version`                           |
| Claude Code | 최신  | `npm install -g @anthropic-ai/claude-code` |

---

## 결과물 실행

```bash
cd src/[PROJECT_NAME]
npm install
npm run dev
# http://localhost:3000
```

---

## 사용 팁

### 각 확인 단계에서 수정할 수 있다

파이프라인이 ✋ 표시에서 멈추면 그 자리에서 수정 요청을 할 수 있다.

**요구사항 확인 시**
```
결제 기능도 넣어줘
로그인은 빼줘
```

**설계 확인 시**
```
화면을 하나로 합쳐줘
API 엔드포인트 구조 바꿔줘
```

**디자인 확인 시**
```
더 미니멀하게 해줘
브랜드 컬러를 블루 계열로 바꿔줘
```

이 단계에서 잡아야 코드를 다 뜯어고치는 일이 없다.

### 중간 산출물 확인

| 파일                    | 작성 에이전트 | 내용                                    |
| ----------------------- | ------------- | --------------------------------------- |
| `docs/brief.md`         | Interviewer   | PROJECT_NAME + 아이디어 원문            |
| `docs/requirements.md`  | PM            | 제품 비전, 사용자, 기능, 수용 기준      |
| `docs/PRD.md`           | Architect     | 제품 컨텍스트 + API, DB, 화면 설계 통합 |
| `docs/design-system.md` | Design        | 톤앤매너, 디자인 토큰                   |
| `docs/review.md`        | Reviewer      | 코드 품질·완성도 검토 결과              |
| `docs/qa-report.md`     | QA            | E2E 시나리오별 PASS/FAIL 결과           |
| `docs/dev-log.md`       | Fullstack     | 구현 중 이슈·에러 누적 히스토리         |

### 에이전트가 중간에 멈추면

```
계속해줘
```

또는 어느 단계부터인지 지정한다.

```
reviewer 서브에이전트부터 다시 실행해줘
```

---

## 파일 구조

```
claude-architect/
├── CLAUDE.md               ← 오케스트레이터 (파이프라인 전체 정의)
├── README.md               ← 이 파일
│
├── .claude/agents/         ← 서브에이전트 정의 (Claude Code가 자동 로드)
│   ├── interviewer.md      ← Step 0: 아이디어 반응 + 사업성 평가 (Opus)
│   ├── pm.md               ← 요구사항 인터뷰 (Opus)
│   ├── architect.md        ← 기술 설계 + PRD 작성 (Opus)
│   ├── design.md           ← UI/UX + 디자인 시스템 정의 (Sonnet)
│   ├── fullstack.md        ← TDD 구현 (Sonnet)
│   ├── reviewer.md         ← 코드 품질·완성도 검토 (Sonnet)
│   └── qa.md               ← 서버 기동 → Playwright E2E (Sonnet)
│
├── docs/                   ← 에이전트 간 공유 메모리 (런타임 생성)
│   ├── brief.md
│   ├── requirements.md
│   ├── PRD.md
│   ├── design-system.md
│   ├── review.md
│   ├── qa-report.md
│   └── dev-log.md          ← 구현 중 이슈·에러 누적 히스토리
│
└── src/
    └── [PROJECT_NAME]/     ← Next.js 풀스택 결과물 (런타임 생성)
```
