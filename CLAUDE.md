# Dev Crew

이 프로젝트는 사용자 아이디어를 실제 동작하는 코드로 변환하는 멀티에이전트 파이프라인이다.

## 사용 방법

사용자가 새 프로젝트 아이디어를 제시하면 아래 파이프라인을 실행한다.
**항상 Step 0부터 시작한다.**

---

## 파이프라인

### Step 0: 아이디어 인터뷰 (Interviewer 에이전트) ✋

`interviewer` 서브에이전트를 실행한다:

> "The user has just shared a new project idea. Do three things before asking anything:
> 1. **React to the idea** — highlight what's genuinely interesting, point out any weak spots, and suggest a sharper angle if you see one.
> 2. **Assess business viability** — briefly evaluate: Who pays for this and why? Is there a real market or is it a solution looking for a problem? What's the most dangerous assumption? Give a candid 2–3 sentence verdict (e.g. 'Strong B2C hook but monetization is unclear', 'Niche but defensible', 'Crowded market — needs a sharper wedge').
> 3. **Confirm direction** — ask the user via AskUserQuestion whether to proceed as-is or pivot based on your assessment.
> Once confirmed, write docs/brief.md (PROJECT_NAME: TBD on the first line)."

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

사용자가 수정을 요청하면 `docs/brief.md`를 수정 내용으로 업데이트한 뒤 Step 1을 다시 실행한다.
사용자가 승인하면 Step 3으로 진행한다.

### Step 3: 아키텍처 설계 (Architect)

`architect` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/requirements.md. Design the system architecture and write the output to docs/PRD.md. Use the PROJECT_NAME from brief.md for all directory paths. Stack: Next.js 16 fullstack (App Router) — code lives in src/[PROJECT_NAME]/."

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

사용자가 수정을 요청하면 수정 내용을 `docs/requirements.md`에 반영한 뒤 Step 3을 다시 실행한다.
사용자가 승인하면 Step 3.6으로 진행한다.

### Step 3.6: 디자인 시스템 정의 (Design) 🎨

**에이전트 실행 전, 사용자에게 먼저 물어본다:**

```
🎨 디자인 작업을 시작하기 전에 —

원하는 느낌의 레퍼런스 이미지나 스크린샷이 있으면 첨부해주세요.
색상, 레이아웃, 타이포 분위기 등을 참고해서 디자인 시스템에 반영할게요.

없으면 "없어요"라고 하시면 텍스트 설명만으로 진행합니다.
```

사용자 응답을 받은 뒤 `design` 서브에이전트를 실행한다:

> "Read @docs/requirements.md and @docs/PRD.md. Act as a product designer. If the user has attached reference images, extract visual cues (color palette, layout density, typography weight, overall mood) and reflect them in your direction. Recommend 2-3 tone-and-manner options and 2-3 concept directions first, then pick one best direction with clear rationale. Based on the selected direction, automatically define a practical design system token set (color, typography, spacing, radius, shadow), component states, and screen-level application guidance. Write the output to docs/design-system.md."

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

사용자가 수정을 요청하면 수정 내용을 반영해 Step 3.6을 다시 실행한다.
사용자가 승인하면 Step 4로 진행한다.

### Step 4: 구현 — TDD (풀스택)

`fullstack` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME), @docs/PRD.md, and @docs/design-system.md. Stack: Next.js 16 fullstack (App Router). Write all files to src/[PROJECT_NAME]/. Apply the approved design system from docs/design-system.md. Use TDD: write tests first, confirm they fail, implement, confirm they pass. The app must pass npm run test AND npm run lint AND npm run build before you finish."

완료될 때까지 기다린다.

### Step 4.5: 코드 리뷰 (Reviewer)

`reviewer` 서브에이전트를 실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME). Review implemented code in src/[PROJECT_NAME]/ against @docs/requirements.md, @docs/PRD.md, and @docs/design-system.md. Pay attention to Out of Scope items — do not flag unimplemented features that are explicitly listed as Out of Scope. Run actual builds/tests/lint, and write findings to docs/review.md."

완료될 때까지 기다린다.

### Step 5: E2E QA

`docs/review.md`의 STATUS가 `PASS`일 때만 `qa` 서브에이전트를 실행한다. NEEDS_REVISION이면 Step 6으로 바로 이동한다.

> "Read @docs/brief.md (first line contains PROJECT_NAME), @docs/requirements.md for MVP Acceptance Criteria. Read the project scripts to determine how to start the server. Start the server in background, run Playwright E2E tests against each Acceptance Criteria, shut down the server, and write findings to docs/qa-report.md."

완료될 때까지 기다린다.

### Step 6: 수정 루프

#### 6-A: Reviewer NEEDS_REVISION (최대 2회)

`docs/review.md`의 STATUS가 `NEEDS_REVISION`이면 `fullstack` 서브에이전트를 재실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/review.md for issues found in your previous implementation. Fix all issues listed. The existing code is in src/[PROJECT_NAME]/. Re-run npm run test, npm run lint, and npm run build to verify fixes."

수정 완료 후 Step 4.5(Reviewer)부터 다시 실행한다. 최대 2회 반복한다.
2회 후에도 NEEDS_REVISION이면 Step 5(QA)는 건너뛰고 Step 7로 이동한다.

#### 6-B: QA NEEDS_REVISION (최대 2회)

`docs/qa-report.md`의 STATUS가 `NEEDS_REVISION`이면 `fullstack` 서브에이전트를 재실행한다:

> "Read @docs/brief.md (first line contains PROJECT_NAME) and @docs/qa-report.md for issues found during E2E testing. Fix all issues listed. The existing code is in src/[PROJECT_NAME]/. Re-run npm run test, npm run lint, and npm run build to verify fixes."

수정 완료 후 Step 5(QA)만 다시 실행한다. 최대 2회 반복한다.

### Step 7: 최종 안내

STATUS가 PASS이면 (또는 최대 수정 횟수 도달 시) 다음을 사용자에게 안내한다:

```
빌드 완료!

  cd src/[PROJECT_NAME]
  npm install
  npm run dev

브라우저: http://localhost:3000
```

---

## 파일 레이아웃

```
docs/
  brief.md          # 첫 줄: "# PROJECT_NAME: [이름]" + 원문 아이디어
  requirements.md   # PM 출력
  PRD.md            # Architect 출력 (PRD + 기술 스펙 통합)
  design-system.md  # Design 출력 (톤앤매너/컨셉/디자인 토큰)
  review.md         # Reviewer 출력
src/
  [PROJECT_NAME]/   # Next.js 풀스택 프로젝트 루트
.claude/agents/     # 서브에이전트 정의
```
