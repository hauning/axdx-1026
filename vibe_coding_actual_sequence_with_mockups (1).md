# 바이브코딩 실제 순서
## AX/DX Harness Engineering × TDD 실습 가이드

이 문서는 `problem.md`, `PRD.md`, `acceptance_criteria.md`, `test_cases.md`, `AGENTS.md`를 만든 뒤 실제로 ChatGPT, Gemini, Codex, Gemini CLI 등을 사용해 웹 앱을 구현하는 순서를 정리한 실습용 가이드입니다.

핵심 원칙은 간단합니다.

> 바이브코딩은 AI에게 마음대로 만들라고 하는 것이 아니다.  
> 문서, 테스트, 하네스를 먼저 만들고 AI가 그 안에서만 움직이게 하는 것이다.

---

# 0. 전체 흐름

```text
Markdown 산출물 다운로드
→ React/Vite 프로젝트 생성
→ docs/ 폴더와 AGENTS.md 배치
→ AI에게 기준 문서 제공
→ 테스트 먼저 작성
→ 최소 코드 구현
→ npm test / npm run build 확인
→ GitHub push
→ Vercel 배포
```

---

# 1. 준비물

## 로컬 환경

```bash
node -v
git --version
npm -v
```

## 사용할 도구

초급자는 아래 중 하나면 충분합니다.

```text
ChatGPT 웹
Gemini 웹
```

중급 이상은 다음을 사용할 수 있습니다.

```text
ChatGPT + GitHub 연결
Codex
Gemini CLI
Gemini Code Assist
Cursor
VS Code
```

---

# 2. React/Vite 프로젝트 생성

```bash
npm create vite@latest axdx-vibe-project -- --template react-ts
cd axdx-vibe-project
npm install
```

필요 패키지 설치:

```bash
npm install lucide-react
npm install -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

---

# 3. 테스트 환경 설정

## 3.1 `vite.config.ts`

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
})
```

## 3.2 `src/test/setup.ts`

```ts
import '@testing-library/jest-dom'
```

## 3.3 `package.json` scripts 확인

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "test": "vitest",
    "preview": "vite preview"
  }
}
```

---

# 4. Markdown 산출물 배치

워크숍 빌더에서 내려받은 Markdown 산출물을 아래처럼 배치합니다.

```text
axdx-vibe-project/
├─ AGENTS.md
├─ docs/
│  ├─ problem.md
│  ├─ persona.md
│  ├─ journey.md
│  ├─ PRD.md
│  ├─ acceptance_criteria.md
│  ├─ test_cases.md
│  ├─ screen_spec.md
│  ├─ kpi.md
│  ├─ experiment_plan.md
│  └─ mockups/
│     ├─ home.png
│     ├─ task-form.png
│     ├─ task-list.png
│     └─ export.png
├─ src/
├─ package.json
└─ README.md
```

중요도는 다음 순서입니다.

```text
1. AGENTS.md
2. docs/PRD.md
3. docs/acceptance_criteria.md
4. docs/test_cases.md
5. docs/screen_spec.md
6. docs/mockups/
7. docs/kpi.md
```

화면 목업은 다음처럼 사용합니다.

```text
목업 = 레이아웃, 정보 배치, 화면 톤 참고
PRD = 무엇을 만들지에 대한 기준
Acceptance Criteria = 통과/실패 판단 기준
Test Cases = 구현 검증 기준
AGENTS.md = AI 작업 규칙
```

목업과 Acceptance Criteria가 충돌하면 **Acceptance Criteria를 우선**합니다.  
목업은 화면의 모양을 안내하지만, 기능의 성공 기준은 테스트 가능한 문서가 결정합니다.

---

# 4-1. 화면 목업을 기준 문서로 넣기

원하는 화면이 있다면 스크린샷, Figma export, 손그림, PPT 캡처를 `docs/mockups/` 폴더에 넣습니다.

권장 파일명:

```text
docs/mockups/
├─ home.png
├─ task-form.png
├─ task-list.png
├─ kpi-dashboard.png
└─ export.png
```

목업은 반드시 `screen_spec.md`로 한 번 번역합니다.  
AI에게 이미지를 직접 보여주는 것보다, 목업을 화면 요구사항으로 바꾼 뒤 구현시키는 편이 안정적입니다.

---

## 4-1-1. `docs/screen_spec.md` 템플릿

```md
# Screen Spec

## 1. Home 화면

### 참고 목업
- docs/mockups/home.png

### 화면 목적
사용자가 워크숍의 전체 흐름을 이해하고 다음 단계로 이동한다.

### 주요 구성요소
- 상단 제목
- 워크숍 설명
- 진행률 표시
- 단계별 네비게이션
- 시작하기 버튼

### 레이아웃 규칙
- 흰 배경
- 좌측 또는 상단 단계 메뉴
- 본문은 카드형 섹션으로 구분
- 모바일에서는 메뉴가 가로 스크롤 또는 드롭다운으로 전환

### 상호작용
- 단계 버튼 클릭 시 해당 단계로 이동
- 시작하기 클릭 시 Problem 화면으로 이동

### 접근성
- 버튼에는 명확한 accessible name 사용
- 제목 계층은 h1 → h2 → h3 순서 유지

---

## 2. TaskForm 화면

### 참고 목업
- docs/mockups/task-form.png

### 화면 목적
사용자가 과제명을 입력하고 저장한다.

### 주요 구성요소
- 과제명 label
- 과제명 input
- 저장 button
- 오류 메시지 영역

### 상태
- 정상 상태: 과제명 입력 시 저장 버튼 활성화
- 빈 상태: 과제명이 없으면 저장 버튼 비활성화
- 오류 상태: 100자 초과 시 오류 메시지 표시
- 엣지 케이스: 공백만 입력하면 저장 버튼 비활성화

### 테스트 연결
- AC-01
- AC-02
- AC-03
- AC-04
```

---

## 4-1-2. 목업을 AI에게 설명시키는 프롬프트

ChatGPT 또는 Gemini에 목업 이미지를 업로드할 수 있다면 먼저 아래처럼 시킵니다.

```text
이 이미지는 내가 만들고 싶은 웹 앱 화면 목업이다.

이 목업을 코드로 바로 만들지 말고,
먼저 docs/screen_spec.md에 넣을 수 있는 화면 정의서로 변환해줘.

반드시 포함할 것:
1. 화면 목적
2. 주요 구성요소
3. 레이아웃 구조
4. 사용자 입력
5. 상태: 정상 / 빈 상태 / 오류 상태 / 엣지 케이스
6. 접근성 요구사항
7. 연결되는 Acceptance Criteria
8. 구현 시 지켜야 할 스타일 규칙

주의:
- 목업의 시각 요소를 그대로 베끼기보다 정보 구조를 추출해라.
- 기능 기준은 docs/acceptance_criteria.md를 우선한다.
- 테스트 가능한 문장으로 정리해라.
```

---

## 4-1-3. 목업 기반 구현 프롬프트

목업을 넣은 뒤 코드를 만들 때는 아래 프롬프트를 사용합니다.

```text
너는 React + TypeScript + Vite 기반 프론트엔드 개발자다.

다음 기준 문서를 모두 반영해 TaskForm 화면을 구현해라.

우선순위:
1. AGENTS.md
2. docs/acceptance_criteria.md
3. docs/test_cases.md
4. docs/screen_spec.md
5. docs/mockups/task-form.png

작업 규칙:
- 목업은 레이아웃과 시각적 참고 자료로만 사용한다.
- 기능 성공 기준은 Acceptance Criteria와 테스트가 결정한다.
- 테스트를 먼저 작성한다.
- 테스트가 찾는 label text와 button name은 바꾸지 않는다.
- 목업에 있어도 MVP 범위 밖 기능은 구현하지 않는다.
- TypeScript any 사용 금지

먼저 TaskForm.test.tsx를 작성하고,
그 다음 테스트를 통과하는 TaskForm.tsx를 작성해라.
```

---

## 4-1-4. 목업 기반 디자인 프롬프트

기능 구현이 끝나고 테스트가 통과한 뒤에만 디자인을 다듬습니다.

```text
기존 테스트가 모두 통과한 상태에서 TaskForm의 UI만 목업에 가깝게 다듬어줘.

참고:
- docs/screen_spec.md
- docs/mockups/task-form.png

조건:
- 기능 변경 금지
- 테스트 코드 수정 금지
- label text와 button name 변경 금지
- 접근성 유지
- 모바일에서 깨지지 않게 반응형 처리
- TypeScript any 사용 금지

디자인 목표:
- 흰 배경
- 카드형 입력 영역
- 명확한 label
- 오류 메시지는 입력창 아래 표시
- 저장 버튼은 오른쪽 또는 하단에 배치
```

---

## 4-1-5. 목업과 구현 결과 비교 프롬프트

구현 후 화면 캡처를 다시 AI에게 보여주고 비교할 수 있습니다.

```text
첫 번째 이미지는 목표 목업이고,
두 번째 이미지는 현재 구현 화면이다.

두 화면을 비교해서 다음 형식으로 피드백해줘.

1. 기능적으로 반드시 고쳐야 할 점
2. 레이아웃 차이
3. 시각적 위계 차이
4. 접근성 문제
5. 모바일에서 예상되는 문제
6. 테스트를 깨지 않고 수정하는 방법

주의:
- 단순 취향 차이는 제외해라.
- Acceptance Criteria와 충돌하는 제안은 하지 마라.
- 수정 우선순위를 High / Medium / Low로 나눠라.
```

---

## 4-1-6. 목업 사용 시 금지사항

```text
목업만 보고 기능을 새로 추가하지 않는다.
목업에 있는 장식 요소 때문에 테스트 기준을 바꾸지 않는다.
목업과 다르다는 이유로 label text, button name을 바꾸지 않는다.
목업의 색상과 간격을 맞추려고 접근성을 희생하지 않는다.
목업을 보고 전체 앱을 한 번에 다시 만들라고 하지 않는다.
```

목업은 방향을 잡는 지도입니다.  
하지만 실제 구현의 합격 기준은 `acceptance_criteria.md`와 테스트입니다.

---

# 5. 첫 번째 AI 프롬프트: 파일 구조 제안 받기

ChatGPT 또는 Gemini에 아래 프롬프트를 넣습니다.

```text
너는 React + TypeScript + Vite + Tailwind 기반 프론트엔드 개발자다.

이 프로젝트는 AX/DX 하네스 엔지니어링 워크숍의 MVP다.
아래 문서를 기준으로만 구현한다.

우선순위:
1. AGENTS.md
2. docs/PRD.md
3. docs/acceptance_criteria.md
4. docs/test_cases.md
5. docs/screen_spec.md
6. docs/mockups/
7. docs/kpi.md

작업 규칙:
- 테스트를 먼저 작성한다.
- High 우선순위 Acceptance Criteria부터 구현한다.
- 한 번에 하나의 기능만 구현한다.
- 테스트가 실패하면 원인부터 설명한다.
- 테스트를 통과시키기 위해 요구사항을 삭제하지 않는다.
- TypeScript any를 사용하지 않는다.
- 구현 후 실행 명령과 확인 방법을 알려준다.

먼저 전체 파일 구조를 제안해라.
아직 코드는 작성하지 말고, 각 파일의 역할만 설명해라.

[AGENTS.md 붙여넣기]

[docs/PRD.md 붙여넣기]

[docs/acceptance_criteria.md 붙여넣기]

[docs/test_cases.md 붙여넣기]

[docs/screen_spec.md 붙여넣기]

[목업 이미지가 있다면 함께 업로드하거나 파일명/화면 설명을 적기]
```

---

# 6. 권장 파일 구조

AI가 제안한 구조를 바탕으로 아래 정도의 최소 구조를 만듭니다.

```text
src/
├─ App.tsx
├─ main.tsx
├─ components/
│  ├─ TaskForm.tsx
│  └─ TaskList.tsx
├─ lib/
│  ├─ types.ts
│  ├─ validation.ts
│  ├─ repository.ts
│  └─ kpi.ts
├─ test/
│  └─ setup.ts
└─ __tests__/
   ├─ TaskForm.test.tsx
   ├─ TaskList.test.tsx
   ├─ validation.test.ts
   └─ kpi.test.ts
```

---

# 7. 구현 순서

한 번에 전체 앱을 만들지 않습니다.  
아래 순서대로 작은 단위로 진행합니다.

```text
1. docs/mockups/에 화면 목업 저장
2. docs/screen_spec.md 작성
3. types.ts
4. TaskForm.test.tsx
5. TaskForm.tsx
6. TaskList.test.tsx
7. TaskList.tsx
8. validation.test.ts
9. validation.ts
10. kpi.test.ts
11. kpi.ts
12. repository.test.ts
13. repository.ts
14. App.tsx 연결
15. localStorage 저장
16. 목업 기준 UI 다듬기
17. README 정리
18. npm run build
19. GitHub push
20. Vercel 배포
```

---

# 8. Step 1: 타입 정의 만들기

## 프롬프트

```text
docs/PRD.md와 docs/test_cases.md를 기준으로 TypeScript 타입을 먼저 정의해줘.

파일:
src/lib/types.ts

조건:
- TaskStatus는 'todo' | 'done'
- Task에는 id, title, dueDate, status, createdAt, completedAt 포함
- TaskInput에는 title, dueDate 포함
- EventName 타입도 정의
- TypeScript any 사용 금지
```

## 예상 파일

```ts
export type TaskStatus = 'todo' | 'done'

export type Task = {
  id: string
  title: string
  dueDate: string | null
  status: TaskStatus
  createdAt: string
  completedAt: string | null
}

export type TaskInput = {
  title: string
  dueDate: string | null
}

export type EventName =
  | 'task_created'
  | 'task_completed'
  | 'task_overdue_viewed'
  | 'due_date_set'
```

---

# 9. Step 2: TaskForm 테스트 먼저 작성

## 프롬프트

```text
docs/acceptance_criteria.md와 docs/test_cases.md를 기준으로
TaskForm.test.tsx를 먼저 작성해줘.

파일:
src/__tests__/TaskForm.test.tsx

조건:
- Testing Library 사용
- 과제명 입력 시 저장 버튼 활성화
- 빈 값이면 저장 버튼 비활성화
- 공백만 입력하면 저장 버튼 비활성화
- 100자 초과 시 오류 메시지 표시
- 정상 제출 시 trim된 값이 onSubmit으로 전달
- 테스트가 찾는 label text는 "과제명"
- 테스트가 찾는 button name은 "저장"
```

## 실행

```bash
npm test
```

이 시점에서는 실패해야 정상입니다.  
아직 `TaskForm.tsx`가 없기 때문입니다.

이 단계가 TDD의 Red 단계입니다.

---

# 10. Step 3: TaskForm 구현

## 프롬프트

```text
방금 작성한 TaskForm.test.tsx를 통과하는
TaskForm.tsx를 작성해줘.

파일:
src/components/TaskForm.tsx

조건:
- label text는 "과제명"
- button name은 "저장"
- 빈 값과 공백은 저장 불가
- 100자 초과 시 "과제명은 100자 이하여야 합니다" 표시
- 정상 제출 시 trim된 값을 onSubmit에 전달
- 제출 후 입력창 초기화
- 테스트 코드는 수정하지 말 것
- TypeScript any 사용 금지
```

## 실행

```bash
npm test
```

테스트가 통과하면 Green 단계입니다.

---

# 11. Step 4: TaskList 테스트 작성

## 프롬프트

```text
docs/acceptance_criteria.md와 docs/test_cases.md를 기준으로
TaskList.test.tsx를 작성해줘.

파일:
src/__tests__/TaskList.test.tsx

조건:
- 과제 목록을 보여준다
- 과제가 없으면 "아직 등록된 일정이 없습니다" 표시
- 완료 과제는 "완료" 배지 표시
- todo 과제는 "미완료" 또는 적절한 상태 표시
- TypeScript any 사용 금지
```

## 실행

```bash
npm test
```

실패하면 정상입니다.

---

# 12. Step 5: TaskList 구현

## 프롬프트

```text
방금 작성한 TaskList.test.tsx를 통과하는
TaskList.tsx를 작성해줘.

파일:
src/components/TaskList.tsx

조건:
- tasks prop을 받는다
- 과제가 없으면 빈 상태 메시지를 표시한다
- 완료 과제는 "완료" 배지를 표시한다
- 접근 가능한 HTML 구조를 사용한다
- TypeScript any 사용 금지
- 테스트 코드는 수정하지 말 것
```

## 실행

```bash
npm test
```

---

# 13. Step 6: 입력 검증 로직 TDD

## 프롬프트

```text
docs/test_cases.md를 기준으로 validateTaskInput 테스트를 작성해줘.

파일:
src/__tests__/validation.test.ts

조건:
- 정상 과제명은 valid true
- 빈 과제명은 valid false
- 공백만 있는 과제명은 valid false
- 100자 초과 과제명은 valid false
- 잘못된 날짜 형식은 valid false
- 반환 형식은 { valid: boolean, errors: string[] }
```

## 구현 프롬프트

```text
validation.test.ts를 통과하는 validateTaskInput 함수를 작성해줘.

파일:
src/lib/validation.ts

조건:
- title.trim().length === 0이면 오류
- title.length > 100이면 오류
- dueDate가 null이 아니면 YYYY-MM-DD 형식 검증
- 실제 존재하지 않는 날짜도 막을 것
- TypeScript any 사용 금지
```

## 실행

```bash
npm test
```

---

# 14. Step 7: KPI 계산 로직 TDD

## 프롬프트

```text
docs/kpi.md를 기준으로 calculateCompletionRate 테스트를 작성해줘.

파일:
src/__tests__/kpi.test.ts

조건:
- 완료 과제 비율을 계산한다
- 과제가 없으면 0을 반환한다
- 모든 과제가 완료되면 1을 반환한다
- 일부만 완료되면 해당 비율을 반환한다
```

## 구현 프롬프트

```text
kpi.test.ts를 통과하는 calculateCompletionRate 함수를 작성해줘.

파일:
src/lib/kpi.ts

조건:
- 입력은 Task[]
- 반환은 number
- tasks.length가 0이면 0
- done 상태인 과제 수 / 전체 과제 수 반환
- TypeScript any 사용 금지
```

---

# 15. Step 8: Mock Repository TDD

## 프롬프트

```text
docs/test_cases.md를 기준으로 Mock Repository 테스트를 작성해줘.

파일:
src/__tests__/repository.test.ts

조건:
- 과제를 추가하면 목록에서 조회된다
- 추가된 과제 제목은 trim 처리된다
- 완료 처리하면 status가 done으로 바뀐다
- 없는 ID를 완료 처리하면 에러를 반환한다
- completedAt이 설정된다
```

## 구현 프롬프트

```text
repository.test.ts를 통과하는 createTaskRepository를 작성해줘.

파일:
src/lib/repository.ts

조건:
- add(input: TaskInput): Task
- findAll(): Task[]
- findById(id: string): Task | undefined
- markDone(id: string): void
- 없는 ID는 "과제를 찾을 수 없습니다" 에러
- TypeScript any 사용 금지
```

---

# 16. Step 9: App.tsx 연결

## 프롬프트

```text
지금까지 만든 TaskForm, TaskList, repository, kpi를 연결하는 App.tsx를 작성해줘.

파일:
src/App.tsx

조건:
- 제목은 "AI Syllabus Planner"
- TaskForm에서 과제를 추가한다
- TaskList에서 과제 목록을 보여준다
- 완료율을 화면에 표시한다
- localStorage는 아직 붙이지 않는다
- TypeScript any 사용 금지
```

## 실행

```bash
npm run dev
```

브라우저에서 확인합니다.

```text
http://localhost:5173
```

확인 항목:

```text
☐ 과제명이 비어 있으면 저장 버튼이 비활성화된다
☐ 공백만 입력하면 저장 버튼이 비활성화된다
☐ 101자 이상 입력하면 오류 메시지가 표시된다
☐ 정상 입력 후 저장하면 목록에 추가된다
☐ 과제가 없을 때 빈 상태 메시지가 보인다
☐ 완료율이 표시된다
```

---

# 17. Step 10: localStorage 저장 추가

## 프롬프트

```text
App.tsx에 localStorage 저장 기능을 추가해줘.

조건:
- tasks 상태를 localStorage에 저장한다
- 새로고침 후에도 목록이 유지된다
- JSON.parse 실패 시 빈 배열로 복구한다
- 기존 테스트를 깨지 않도록 한다
- TypeScript any 사용 금지
```

## 확인

```bash
npm run dev
```

확인 항목:

```text
☐ 과제를 추가한다
☐ 브라우저를 새로고침한다
☐ 과제 목록이 유지된다
```

---

# 18. Step 11: 오류 로그를 AI에게 줄 때

오류가 나면 바로 “고쳐줘”라고 하지 않습니다.  
먼저 원인 분석부터 시킵니다.

## 디버깅 프롬프트

```text
다음 테스트 실패 로그를 분석해줘.

바로 코드를 고치지 말고 먼저 원인 후보를 3가지로 나눠 설명해줘.
그 다음 가장 가능성 높은 원인부터 수정안을 제시해줘.

조건:
- 테스트 코드를 임의로 바꾸지 말 것
- acceptance criteria를 삭제하지 말 것
- 수정 파일과 수정 이유를 분리해서 설명할 것

테스트 실패 로그:
[로그 붙여넣기]

관련 테스트 코드:
[테스트 코드 붙여넣기]

관련 구현 코드:
[구현 코드 붙여넣기]
```

---

# 19. Step 12: 리팩토링 프롬프트

테스트가 모두 통과한 뒤에만 리팩토링합니다.

```text
다음 컴포넌트를 리팩토링해줘.

조건:
- 기존 테스트가 모두 통과해야 한다
- label text와 button name은 바꾸지 않는다
- 기능 추가 금지
- 요구사항 삭제 금지
- TypeScript any 사용 금지

개선 목표:
- 중복 제거
- 함수 이름 명확화
- UI와 비즈니스 로직 분리
- 접근성 유지
- 과도한 추상화 금지

대상 코드:
[코드 붙여넣기]
```

---

# 20. Step 13: README 작성

## 프롬프트

```text
이 프로젝트의 README.md를 작성해줘.

포함할 것:
- 프로젝트 이름
- 문제 상황
- 핵심 사용자
- MVP 기능
- 실행 방법
- 테스트 방법
- 빌드 방법
- 산출물 문서 위치
- 배포 URL을 적을 자리
```

---

# 21. Step 14: 빌드 확인

```bash
npm run test
npm run build
```

둘 다 통과해야 배포합니다.

---

# 22. Step 15: GitHub에 올리기

```bash
git init -b main
git add .
git commit -m "Initial vibe coding MVP"
git remote add origin https://github.com/YOUR_ID/axdx-vibe-project.git
git push -u origin main
```

---

# 23. Step 16: Vercel 배포

Vercel에서:

```text
Add New
→ Project
→ Import Git Repository
→ axdx-vibe-project 선택
```

설정값:

```text
Framework Preset: Vite
Build Command: npm run build
Output Directory: dist
Install Command: npm install
```

Deploy 클릭.

---

# 24. Step 17: 배포 후 확인

```text
☐ URL 접속이 된다
☐ 과제 입력이 된다
☐ 저장 버튼 조건부 활성화/비활성화가 된다
☐ 과제 목록이 보인다
☐ 빈 상태 메시지가 있다
☐ 오류 메시지가 있다
☐ 새로고침 후 데이터가 유지된다
☐ 모바일에서 크게 깨지지 않는다
☐ 브라우저 콘솔 오류가 없다
```

---

# 25. 제출물

최종 제출물은 다음입니다.

```text
1. GitHub Repository URL
2. Vercel 배포 URL
3. docs/PRD.md
4. docs/acceptance_criteria.md
5. docs/test_cases.md
6. docs/screen_spec.md
7. docs/mockups/ 참고 목업
8. AGENTS.md
9. 실행 화면 캡처
10. 테스트 통과 화면 캡처
```

---

# 26. 초급자용 최소 루프

초급자는 이 루프만 지키면 됩니다.

```text
1. 문서 붙여넣기
2. 테스트 먼저 요청
3. 테스트 파일 저장
4. npm test로 실패 확인
5. 구현 파일 요청
6. npm test로 통과 확인
7. npm run build 확인
8. GitHub push
9. Vercel 배포
```

---

# 27. AI에게 절대 하면 안 되는 요청

```text
"그냥 예쁘게 만들어줘"
"목업이랑 똑같이 알아서 전체 앱 만들어줘"
"알아서 전체 앱 만들어줘"
"테스트는 나중에 하고 일단 돌아가게 해줘"
"에러 나니까 테스트를 고쳐줘"
"요구사항은 적당히 바꿔도 돼"
"목업에 있으니 MVP 범위 밖 기능도 넣어줘"
```

이런 요청은 하네스를 무너뜨립니다.

---

# 28. AI에게 해야 하는 요청

```text
"AGENTS.md를 먼저 읽고 작업해줘"
"docs/acceptance_criteria.md의 AC-01부터 구현해줘"
"docs/screen_spec.md와 docs/mockups/task-form.png는 레이아웃 참고로만 써줘"
"테스트를 먼저 작성해줘"
"테스트 실패 원인을 먼저 설명해줘"
"요구사항을 삭제하지 말고 최소 수정해줘"
"목업과 AC가 충돌하면 AC를 우선해줘"
"변경 파일 목록과 실행 명령을 마지막에 요약해줘"
```

---

# 29. 최종 메시지

바이브코딩의 핵심은 속도가 아닙니다.

```text
문제정의가 AI의 방향을 잡고,
Acceptance Criteria가 판단 기준을 만들고,
테스트가 회귀를 막고,
AGENTS.md가 AI의 행동 범위를 제한한다.
```

따라서 실제 바이브코딩 순서는 다음 한 문장으로 요약됩니다.

> 먼저 하네스를 만들고, 그 다음 AI에게 코드를 쓰게 하라.
