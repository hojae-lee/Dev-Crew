# Dev Crew

이 프로젝트는 사용자 아이디어를 실제 동작하는 코드로 변환하는 멀티에이전트 파이프라인이다.

## 사용 방법

사용자 요청이 오면 **모드를 먼저 판단**한다.

- `src/` 아래에 프로젝트 디렉터리가 존재하고, 요청이 기존 코드 변경/추가/수정이면 → **유지보수 모드** (아래 별도 섹션)
- 새 프로젝트 시작이면 → **신규 개발 모드** (Step 0부터)

---

## 신규 개발 모드

사용자가 아이디어를 제시하면 (어떤 언어로든) 아래 파이프라인을 실행한다.
**항상 Step 0부터 시작한다.**

---

## 파이프라인

### Step 0: 아이디어 인터뷰 (Interviewer 에이전트) ✋

`interviewer` 서브에이전트를 실행한다:

> "The user has just shared a new project idea. React to the idea decisively — highlight what's interesting, suggest a better angle if you see one. Ask the user to confirm the direction via AskUserQuestion. Once confirmed, write docs/brief.md (PROJECT_NAME: TBD on the first line)."

완료될 때까지 기다린다. `docs/brief.md`가 생성된 뒤에만 Step 1로 진행한다.

### Step 1: 요구사항 정의 (PM 인터뷰)

`pm` 서브에이전트를 실행한다:

> "Read @docs/brief.md. First question must be the project name (lowercase, hyphens allowed, used as folder name src/[name]/). Then ask 2–4 requirements questions in the same message. Once you receive the project name, immediately update the first line of docs/brief.md to '# PROJECT_NAME: [name]'. After the interview (max 2 rounds), write docs/requirements.md."

PM이 사용자와 직접 티키타카한 뒤 `docs/brief.md` 첫 줄을 업데이트하고 `docs/requirements.md`를 작성한다. 완료될 때까지 기다린다.

`docs/brief.md` 첫 줄은 반드시 아래 형식이어야 한다:
```
# PROJECT_NAME: [프로젝트명]
```

이후 모든 에이전트 프롬프트에서 `[PROJECT_NAME]`은 이 값을 지칭한다.

### Step 1.5: 요구사항 사용자 확인 (Human in the Loop) ✋

**반드시 멈추고 사용자에게 확인을 받아야 한다. 확인 전에는 절대 다음 단계로 진행하지 않는다.**

`docs/requirements.md`의 핵심 내용을 아래 형식으로 요약하여 사용자에게 보여준다:

```
📋 요구사항 정리가 완료됐어요. 확인해주세요.

**만들 것**: [Summary 한 줄 요약]

**MVP 기능**:
- [기능 1]
- [기능 2]
- ...

**제외 항목**: [Out of Scope 요약]

이대로 설계를 시작할까요?
수정할 부분이 있으면 말씀해주세요. (예: "결제 기능도 넣어줘", "로그인은 빼줘")
```

사용자가 수정을 요청하면:

- `docs/brief.md`를 수정 내용으로 업데이트한 뒤 Step 1을 다시 실행한다.
- 사용자가 승인하면 Step 3으로 진행한다.

### Step 3: 아키텍처 설계 (Architect)

`architect` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/requirements.md. Design the system architecture and write the output to docs/PRD.md. Use the PROJECT_NAME from brief.md for all directory paths. Default to Next.js App Router fullstack — code lives in src/[PROJECT_NAME]/ (no frontend/backend split). If the user explicitly requests React+Vite, design React+Vite+TypeScript (FSD) frontend in src/[PROJECT_NAME]/frontend/ on port 5173 with Python+FastAPI backend in src/[PROJECT_NAME]/backend/ on port 8000 and SQLite at src/[PROJECT_NAME]/backend/app.db."

완료될 때까지 기다린다.

### Step 3.5: 아키텍처 사용자 확인 (Human in the Loop) ✋

**반드시 멈추고 사용자에게 확인을 받아야 한다. 확인 전에는 절대 다음 단계로 진행하지 않는다.**

`docs/PRD.md`의 핵심 내용을 아래 형식으로 요약하여 사용자에게 보여준다:

```
🏗️ 설계가 완료됐어요. 확인해주세요.

**화면 구성**:
- [페이지/화면 목록]

**주요 API**:
- [엔드포인트 목록, 3~5개]

**데이터 구조**:
- [엔티티 목록]

이 설계로 개발을 시작할까요?
변경할 부분이 있으면 말씀해주세요. (예: "화면을 하나로 합쳐줘", "API 구조 바꿔줘")
```

사용자가 수정을 요청하면:

- 수정 내용을 `docs/requirements.md`에 반영한 뒤 Step 3을 다시 실행한다.
- 사용자가 승인하면 Step 4로 진행한다.

### Step 3.6: 디자인 시스템 정의 (Desing) 🎨

`design` 서브에이전트를 실행한다:

> "Read @docs/requirements.md and @docs/PRD.md. Act as a product designer. Recommend 2-3 tone-and-manner options and 2-3 concept directions first, then pick one best direction with clear rationale. Based on the selected direction, automatically define a practical design system token set (color, typography, spacing, radius, shadow), component states, and screen-level application guidance. Write the output to docs/design-system.md."

완료될 때까지 기다린다.

### Step 3.7: 디자인 사용자 확인 (Human in the Loop) ✋

**반드시 멈추고 사용자에게 확인을 받아야 한다. 확인 전에는 절대 다음 단계로 진행하지 않는다.**

`docs/design-system.md`의 핵심 내용을 아래 형식으로 요약하여 사용자에게 보여준다:

```
🎨 디자인 가이드가 완료됐어요. 확인해주세요.

**추천 톤앤매너 옵션**:
- [옵션 A]
- [옵션 B]
- [옵션 C]

**선택된 컨셉**: [최종 선택안 + 선택 이유 한 줄]

**자동 정의된 토큰 요약**:
- Color: [핵심 팔레트]
- Typography: [타입 스케일]
- Spacing/Radius/Shadow: [핵심 규칙]

이 디자인 시스템으로 개발을 시작할까요?
수정할 부분이 있으면 말씀해주세요. (예: "더 미니멀하게", "브랜드 컬러를 블루로")
```

사용자가 수정을 요청하면:

- 수정 내용을 반영해 Step 3.6을 다시 실행한다.
- 사용자가 승인하면 Step 4로 진행한다.

### Step 4: 구현 — TDD (스택별 분기)

스택 선택에 따라 구현 흐름을 분기한다. 각 에이전트는 **TDD 방식**으로 작동한다: 테스트 먼저 작성 → 실패 확인 → 구현 → 통과 확인.

#### Step 4-A: Frontend 구현 (공통)

`frontend` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME), @docs/PRD.md, and @docs/design-system.md. Default stack is Next.js App Router fullstack — write all files to src/[PROJECT_NAME]/ (e.g. src/[PROJECT_NAME]/app/, src/[PROJECT_NAME]/components/). If React+Vite was explicitly selected, write to src/[PROJECT_NAME]/frontend/. Apply approved design system tokens and component rules from docs/design-system.md. The app must pass npm run test AND npm run lint AND npm run build before you finish."

완료될 때까지 기다린다.

#### Step 4.5: 프론트엔드 UI/UX 사용자 확인 (Human in the Loop) ✋

**React+Vite + FastAPI 스택일 때만 실행한다. Next.js 풀스택은 건너뛴다.**

**반드시 멈추고 사용자에게 확인을 받아야 한다. 확인 전에는 절대 Step 4-B로 진행하지 않는다.**

사용자에게 아래 형식으로 안내한다:

```
🖥️ 프론트엔드 구현이 완료됐어요. 백엔드 API 개발 전에 화면을 확인해주세요.

아래 명령으로 프론트엔드를 실행해보세요:
  cd src/[PROJECT_NAME]/frontend
  npm install
  npm run dev
  브라우저: http://localhost:5173

**확인 포인트**:
- 화면 구성과 흐름이 의도한 대로인가요?
- API 호출이 필요한 버튼/폼/데이터 표시 영역이 맞나요?
- 수정이 필요한 UI/UX가 있으면 알려주세요.

확인 후 "계속" 또는 수정 사항을 말씀해주세요.
```

사용자가 수정을 요청하면:
- `frontend` 서브에이전트를 재실행하여 UI/UX를 수정한다.
- 수정 완료 후 이 단계를 다시 반복한다.

사용자가 승인하면 Step 4-B로 진행한다.

#### Step 4-B: Backend 구현 (React+Vite 선택 시에만)

**React+Vite + FastAPI 스택일 때만** `backend` 서브에이전트를 실행한다.

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/PRD.md. Implement the complete backend API in src/[PROJECT_NAME]/backend/ using TDD. Use Python+FastAPI+SQLAlchemy+SQLite. Set up a virtualenv at src/[PROJECT_NAME]/backend/.venv before installing dependencies. The server must pass pytest AND ruff check . before you finish."

Next.js 풀스택 선택 시 이 Step은 건너뛴다.

#### Step 4-C: 코드 리뷰 (Reviewer)

`reviewer` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME). Review implemented code against @docs/requirements.md, @docs/PRD.md, and @docs/design-system.md. If React+Vite+FastAPI stack was selected, review both src/[PROJECT_NAME]/frontend/ and src/[PROJECT_NAME]/backend/. If Next.js fullstack was selected, review src/[PROJECT_NAME]/. Run actual builds/tests/lint, and write findings to docs/review.md."

완료될 때까지 기다린다.

### Step 5: E2E QA

`docs/review.md`의 STATUS가 `PASS`일 때만 `qa` 서브에이전트를 실행한다. NEEDS_REVISION이면 Step 6으로 바로 이동한다.

> "Read @docs/brief.md (first line contains PROJECT_NAME), @docs/requirements.md for MVP Acceptance Criteria. Read the project scripts to determine how to start the server(s). Start the server(s) in background, run Playwright E2E tests against each Acceptance Criteria, shut down the server(s), and write findings to docs/qa-report.md."

완료될 때까지 기다린다.

### Step 6: 수정 루프

#### 6-A: Reviewer NEEDS_REVISION (최대 2회)

`docs/review.md`의 STATUS가 `NEEDS_REVISION`이면:

- FRONTEND_ISSUES가 있으면 `frontend` 서브에이전트를 재실행:

  > "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/review.md for issues found in your previous implementation. Fix all FRONTEND_ISSUES listed. For Next.js fullstack, the existing code is in src/[PROJECT_NAME]/. For React+Vite, the existing code is in src/[PROJECT_NAME]/frontend/. Re-run npm run test, npm run lint, and npm run build to verify fixes."

- BACKEND_ISSUES가 있으면 `backend` 서브에이전트를 재실행 (React+Vite+FastAPI 스택일 때만):

  > "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/review.md for issues found in your previous implementation. Fix all BACKEND_ISSUES listed. The existing code is in src/[PROJECT_NAME]/backend/. Activate the venv first (cd src/[PROJECT_NAME]/backend && source .venv/bin/activate), then re-run pytest and ruff check . to verify fixes."

수정 완료 후 Step 4-C(Reviewer)부터 다시 실행한다. 최대 2회 반복한다.
2회 후에도 NEEDS_REVISION이면 Step 5(QA)는 건너뛰고 Step 7로 이동한다.

#### 6-B: QA NEEDS_REVISION (최대 2회)

`docs/qa-report.md`의 STATUS가 `NEEDS_REVISION`이면:

- FRONTEND_ISSUES가 있으면 `frontend` 서브에이전트를 재실행:

  > "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/qa-report.md for issues found during E2E testing. Fix all FRONTEND_ISSUES listed. For Next.js fullstack, the existing code is in src/[PROJECT_NAME]/. For React+Vite, the existing code is in src/[PROJECT_NAME]/frontend/. Re-run npm run test, npm run lint, and npm run build to verify fixes."

- BACKEND_ISSUES가 있으면 `backend` 서브에이전트를 재실행 (React+Vite+FastAPI 스택일 때만):

  > "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/qa-report.md for issues found during E2E testing. Fix all BACKEND_ISSUES listed. The existing code is in src/[PROJECT_NAME]/backend/. Activate the venv first (cd src/[PROJECT_NAME]/backend && source .venv/bin/activate), then re-run pytest and ruff check . to verify fixes."

수정 완료 후 Step 5(QA)만 다시 실행한다. 최대 2회 반복한다.

### Step 7: 최종 안내

STATUS가 PASS이면 (또는 최대 수정 횟수 도달 시) 다음을 사용자에게 안내한다:

```
빌드 완료!

[Next.js 풀스택일 때]
  cd src/[PROJECT_NAME]
  npm install
  npm run dev
  브라우저: http://localhost:3000

[React+Vite + FastAPI일 때]
백엔드 실행:
  cd src/[PROJECT_NAME]/backend
  source .venv/bin/activate        # Windows: .venv\Scripts\activate
  uvicorn app.main:app --reload --port 8000

프론트엔드 실행 (새 터미널):
  cd src/[PROJECT_NAME]/frontend
  npm install
  npm run dev

브라우저: http://localhost:5173
API 문서: http://localhost:8000/docs
```

---


## 파일 레이아웃

```
docs/
  brief.md                  # 첫 줄: "# PROJECT_NAME: [이름]" + 원문 아이디어
  requirements.md           # PM 출력
  PRD.md                    # Architect 출력 (PRD + 기술 스펙 통합)
  design-system.md          # Desing 출력 (톤앤매너/컨셉/디자인 토큰)
  review.md                 # Reviewer 출력
  maintenance-request.md    # 유지보수 요청 정리 (유지보수 모드 시)
src/
  [PROJECT_NAME]/           # Next.js 풀스택: 프로젝트 루트 (app/, components/ 등 직접 위치)
  [PROJECT_NAME]/           # React+Vite + FastAPI:
    frontend/               #   Frontend 에이전트 출력
    backend/                #   Backend 에이전트 출력
.claude/agents/       # 서브에이전트 정의 (Claude Code가 로드)
```

---

## 유지보수 모드

`src/` 아래에 프로젝트 디렉터리가 존재하고 사용자 요청이 기존 코드 변경/기능 추가/버그 수정이면 이 흐름을 사용한다.
프로젝트명은 `docs/brief.md` 첫 줄(`# PROJECT_NAME: [이름]`)에서 읽는다.

### M-Step 1: 변경 요청 캡처

`docs/maintenance-request.md`에 사용자의 원문 요청과 오늘 날짜를 기록한다.

### M-Step 2: PM 인터뷰 (변경 범위 확정) ✋

`pm` 서브에이전트를 실행한다:

> "Read @docs/maintenance-request.md and the existing @docs/requirements.md. Interview the user with AskUserQuestion to clarify the scope of this maintenance change (what to add/modify/remove, what must NOT break). Then write a concise change spec to docs/maintenance-request.md under a '## Change Spec' section — list only what changes, not the entire requirements."

완료될 때까지 기다린다.

### M-Step 2.5: 변경 범위 확인 ✋

**반드시 멈추고 사용자에게 확인을 받아야 한다.**

`docs/maintenance-request.md`의 Change Spec을 요약해 보여준다:

```
🔧 변경 범위를 확인해주세요.

**변경할 것**:
- [변경 항목 목록]

**건드리지 않을 것**:
- [보존 항목 목록]

이대로 진행할까요?
```

사용자가 승인하면 M-Step 3으로 진행한다.

### M-Step 3: 구현 (해당 에이전트만)

변경 범위에 따라 해당 에이전트만 실행한다.

**프론트엔드 변경이 있으면** `frontend` 서브에이전트 실행:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/maintenance-request.md for the Change Spec. For Next.js fullstack, existing code is in src/[PROJECT_NAME]/. For React+Vite, existing code is in src/[PROJECT_NAME]/frontend/. Read existing code before making any changes. Make only the changes described in the Change Spec — do not touch unrelated files. Existing tests must still pass. Run npm run test, npm run lint, npm run build to verify."

**백엔드 변경이 있으면** `backend` 서브에이전트 실행 (React+Vite+FastAPI 스택일 때만):

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/maintenance-request.md for the Change Spec. Existing code is in src/[PROJECT_NAME]/backend/. Read existing code before making any changes. Make only the changes described — do not touch unrelated files. Activate venv first (cd src/[PROJECT_NAME]/backend && source .venv/bin/activate). Existing tests must still pass. Run pytest and ruff check . to verify."

### M-Step 4: 회귀 검증 (Reviewer)

`reviewer` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/maintenance-request.md for the Change Spec. Review the changes in maintenance mode: verify that (1) the requested changes are implemented correctly in src/[PROJECT_NAME]/, (2) all previously passing tests still pass (no regression), (3) lint and build pass. Write findings to docs/review.md."

완료될 때까지 기다린다.

### M-Step 4.5: QA 실행 여부 확인 ✋

Reviewer STATUS가 `PASS`이면 **반드시 멈추고 사용자에게 확인을 받는다.**

```
✅ 코드 리뷰 통과했어요.

변경된 기능에 대해 E2E 테스트도 실행할까요?
- 예: 브라우저를 직접 열어 변경 흐름을 검증합니다 (시간 소요)
- 아니오: 코드 리뷰 통과로 완료 처리합니다
```

- 사용자가 **예**라고 하면 → qa 서브에이전트를 실행한다 (신규 개발 모드의 Step 5와 동일)
- 사용자가 **아니오**라고 하면 → M-Step 6으로 바로 이동한다

### M-Step 5: 수정 루프 (최대 2회)

신규 개발 모드의 Step 6과 동일하게 처리한다. STATUS가 NEEDS_REVISION이면 해당 에이전트만 재실행한다.

### M-Step 6: 완료 안내

변경이 완료되면 사용자에게 안내한다:

```
변경 완료!

변경된 항목: [Change Spec 요약]

실행 방법은 기존과 동일합니다.
```
