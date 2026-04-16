# Dev Crew

아이디어를 입력하면 멀티에이전트 파이프라인이 실제 동작하는 코드로 변환해준다.

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

## 사전 요구사항

| 항목        | 버전  | 확인                                                 |
| ----------- | ----- | ---------------------------------------------------- |
| Node.js     | 20+   | `node --version`                                     |
| Python      | 3.11+ | `python --version` (React+Vite+FastAPI 스택 선택 시) |
| Claude Code | 최신  | `npm install -g @anthropic-ai/claude-code`           |

---

## 파이프라인

### 신규 개발

```
✋ Step 0    Interviewer  아이디어 청취 + 방향 결정 + 환경 확인
                                    docs/brief.md
      ↓
    PM        사용자 인터뷰          docs/requirements.md
      ↓
✋ Step 1.5  요구사항 확인
      ↓
    Architect  기술 설계             docs/architecture.md
      ↓
✋ Step 3.5  설계 확인
      ↓
    Desing     디자인 시스템         docs/design-system.md
      ↓
✋ Step 3.7  디자인 확인
      ↓
    Frontend   TDD 구현              src/[PROJECT_NAME]/
      ↓
✋ Step 4.5  UI/UX 확인             (React+Vite+FastAPI 스택일 때만)
      ↓
    Backend    TDD 구현              src/[PROJECT_NAME]/backend/  (React+Vite+FastAPI 스택일 때만)
      ↓
    QA         E2E 테스트            docs/qa-report.md
               (서버 기동 → Playwright → 서버 종료)
      ↓
    Reviewer   검증                  docs/review.md
      ↓
    PASS → 실행 안내  /  NEEDS_REVISION → 최대 2회 재실행
```

✋ 표시된 단계에서 파이프라인이 멈추고 사용자 확인을 기다린다.

### 유지보수 (기존 코드 변경)

`src/[PROJECT_NAME]/`가 존재하고 수정/추가/버그 요청이 들어오면 자동으로 유지보수 모드로 전환된다.

```
변경 요청
      ↓
    PM        변경 범위 인터뷰   docs/maintenance-request.md
      ↓
✋  변경 범위 확인
      ↓
    Frontend / Backend   Change Spec 항목만 최소 변경
      ↓
    Reviewer  변경 구현 확인 + 회귀 체크
```

Architect / Desing은 건너뛴다.

---

## 기술 스택

파이프라인 시작 시 선택한다.

|               | 기본값                           | 옵션                                                             |
| ------------- | -------------------------------- | ---------------------------------------------------------------- |
| **스택**      | Next.js 16 (App Router, 풀스택) | React 19 + Vite 5 + TypeScript 5 (FSD) + Python FastAPI + SQLite |
| **포트**      | 3000                             | FE: 5173 / BE: 8000                                              |
| **백엔드**    | Next.js Route Handler 내장       | FastAPI + SQLAlchemy 2.0 + SQLite                                |
| **FE 테스트** | Vitest + React Testing Library   | 동일                                                             |
| **BE 테스트** | —                                | pytest + httpx                                                   |
| **린트**      | ESLint                           | ESLint + Ruff                                                    |

---

## 결과물 실행

### Next.js 풀스택 (기본)

```bash
cd src/[PROJECT_NAME]
npm install
npm run dev
# http://localhost:3000
```

### React+Vite + FastAPI (옵션)

```bash
# 터미널 1 — 백엔드
cd src/[PROJECT_NAME]/backend
source .venv/bin/activate   # Windows: .venv\Scripts\activate
uvicorn app.main:app --reload --port 8000

# 터미널 2 — 프론트엔드
cd src/[PROJECT_NAME]/frontend
npm install
npm run dev
# http://localhost:5173
# API 문서: http://localhost:8000/docs
```

---

## 사용 팁

### 확인 단계에서 수정하기

파이프라인이 멈추면 그 자리에서 수정 요청을 할 수 있다.

**Interviewer와 대화 중** — 아이디어를 말하면 방향을 잡아준다. 모호해도 괜찮다.

```
유튜브 요약 앱 만들고 싶어요.
→ 히스토리 저장이 핵심이겠네요. 그 방향으로 갑니다.
```

**PM 인터뷰 중** — PM이 직접 3~5개 질문을 던진다. 자유롭게 답하면 된다.

```
Q. 로그인이 필요한가요?
→ 아니요, 로컬에서만 씁니다.
```

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

**UI/UX 확인 시** (React+Vite+FastAPI 스택)

```
버튼 위치 바꿔줘
목록 페이지를 카드 레이아웃으로 바꿔줘
```

이 단계에서 잡아야 코드 다 뜯어고치는 일이 없다.

### 처음 아이디어를 구체적으로 줄수록 결과가 좋다

```
할 일 관리 앱인데, 태그로 분류할 수 있고
마감일 설정하면 오늘 마감인 것들은 빨간색으로 표시해줘.
로그인은 필요 없고 그냥 로컬에서만 써.
```

### 중간 산출물 확인

뭔가 이상하면 중간 파일들을 직접 열어볼 수 있다.

| 파일                    | 작성 에이전트 | 내용                          |
| ----------------------- | ------------- | ----------------------------- |
| `docs/requirements.md`  | PM            | 기능 목록, 수용 기준          |
| `docs/architecture.md`  | Architect     | API, DB, 화면 설계            |
| `docs/design-system.md` | Desing        | 톤앤매너, 컨셉, 디자인 토큰   |
| `docs/qa-report.md`     | QA            | E2E 시나리오별 PASS/FAIL 결과 |
| `docs/review.md`        | Reviewer      | 테스트/린트/빌드 결과         |

---

## FAQ

**Q. 에이전트가 중간에 멈추면?**

```
계속해줘
```

또는 어느 단계부터인지 지정한다.

```
reviewer 서브에이전트부터 다시 실행해줘
```

**Q. 에러가 났는데 뭔지 모르겠으면?**

```
무슨 문제야? 고쳐줘
```

**Q. 기술 스택을 바꾸고 싶으면?**

파이프라인 시작 전 Step 0에서 선택할 수 있다. 아이디어와 함께 미리 말해도 된다.

```
React+Vite + FastAPI 스택으로 만들어줘
```

**Q. 기존 코드에 기능 추가하고 싶으면?**

`src/` 디렉터리가 있으면 자동으로 유지보수 모드로 전환된다.

```
다크모드 추가해줘
```

```
삭제할 때 확인 팝업 넣어줘
```

---

## 파일 구조

```
claude-architect/
├── CLAUDE.md               ← 오케스트레이터 (파이프라인 전체 정의)
├── README.md               ← 이 파일
├── ARCHITECTURE.md         ← 기술 아키텍처 (설계 결정, 주요 개념)
│
├── .claude/agents/         ← 서브에이전트 정의 (Claude Code가 로드)
│   ├── interviewer.md      ← Step 0: 아이디어 방향 결정 + 환경 확인 (Opus)
│   ├── pm.md               ← 요구사항 인터뷰 (Opus)
│   ├── architect.md        ← 기술 설계 (Opus)
│   ├── desing.md           ← 디자인 시스템 (Sonnet)
│   ├── frontend.md         ← 구현 (Sonnet)
│   ├── backend.md          ← 구현 (Sonnet)
│   ├── qa.md               ← 서버 기동 → Playwright E2E (Sonnet)
│   └── reviewer.md         ← 검증 (Sonnet)
│
├── agents/                 ← 에이전트 포맷 스펙 (참조용)
│
├── docs/                   ← 에이전트 간 공유 메모리 (런타임 생성)
│   ├── brief.md            ← interviewer 출력 (PROJECT_NAME 포함)
│   ├── requirements.md
│   ├── architecture.md
│   ├── design-system.md
│   ├── maintenance-request.md
│   ├── qa-report.md        ← qa 출력 (E2E 결과)
│   └── review.md
│
└── src/
    └── [PROJECT_NAME]/     ← 최종 산출물 (런타임 생성)
        │                      Next.js 풀스택: app/, components/ 등 직접 위치
        ├── frontend/          React+Vite 선택 시
        └── backend/           React+Vite+FastAPI 선택 시
```

기술 아키텍처와 설계 결정 → [ARCHITECTURE.md](./ARCHITECTURE.md)
