# AGENTS.md

웹 초대장 제작 서비스. 정보만 입력하면 모바일 웹 초대장을 만들고, 링크로 공유해 참석 여부를 받는다.
자세한 요구사항은 `docs/prd.md`를 기준으로 한다. PRD와 이 문서가 다르면 작업 전에 사용자에게 묻는다.

---

## 1. 사용자

- 이 서비스를 만드는 사람은 **코딩 초보자**다.
- 답변과 설명은 **한국어**로, 전문 용어는 처음 나올 때 한 줄로 풀어서 설명한다.
- 한 번에 많이 바꾸지 않는다. 작은 단위로 작업하고, 무엇을 했는지 쉬운 말로 알려준다.
- 사용자가 직접 실행해야 하는 명령은 그대로 복사해 쓸 수 있게 적는다.
- **모르거나 애매한 부분은 추측하지 말고 먼저 질문한다.** 특히 PRD에 없는 기능, 화면, 데이터 구조를 정해야 할 때.

## 2. 기술 스택 (변경 금지)

아래 스택만 사용한다. 버전 변경, 라이브러리 추가·교체·삭제는 **사용자 승인 없이 하지 않는다.**

| 구분 | 사용 기술 |
|---|---|
| 프레임워크 | Next.js 16.4.0 (App Router, `src/app/`) |
| UI | React 19.3.0 |
| 언어 | TypeScript 5 (`strict` 모드) |
| 스타일 | Tailwind CSS 4 (`@tailwindcss/turbopack`) |
| 데이터 저장 | Supabase (초대장·응답 저장) |
| 린트 | ESLint 9 + `eslint-config-next` |
| 패키지 매니저 | npm (`package-lock.json`) |

- Supabase 패키지(`@supabase/supabase-js`)는 아직 설치되어 있지 않다. 처음 저장 기능을 만들 때 사용자에게 알리고 설치한다.
- `next.config.ts`의 설정(`cacheComponents`, `partialPrefetching`, turbopack 규칙)은 임의로 바꾸지 않는다.
- import 경로는 `@/*` 별칭(`src/*`)을 쓴다.
- 이 Next.js 버전은 학습 데이터와 다를 수 있다. 코드를 쓰기 전에 `node_modules/next/dist/docs/`의 관련 문서를 먼저 읽는다 (아래 Next.js 규칙 참고).

## 3. 디자인 규칙

- 최종 디자인은 **추후 Figma MCP로 구현**한다. 그전까지 디자인을 임의로 만들어내지 않는다.
- 피그마 디자인이 나오기 전에는 **최소한의 기본 스타일**만 쓴다: Tailwind 기본 여백·글자 크기·정렬 정도. 색·폰트·그림자·애니메이션 등 꾸밈은 넣지 않는다.
- **모바일 우선**으로 만든다. 기본 화면 폭은 모바일 기준이고, 큰 화면에서도 깨지지 않게만 한다.
- 피그마 디자인을 구현할 때:
  - Figma 관련 스킬(`figma-design-to-code` 등)을 먼저 불러오고 그 절차를 따른다.
  - 색·글꼴·간격은 `src/app/globals.css`의 토큰(CSS 변수, `@theme`)으로 정리해서 쓴다. 같은 값을 여러 곳에 하드코딩하지 않는다.
  - 피그마와 다르게 만들어야 할 이유가 있으면 먼저 사용자에게 묻는다.
- 버튼·입력창 등 반복되는 UI는 컴포넌트로 분리한다 (`src/components/`).

## 4. 서비스 구조

로그인 없이 호스트와 게스트 모두 링크로 사용한다.

### 화면과 경로

| 경로 | 사용자 | 하는 일 |
|---|---|---|
| `/create` | 호스트 | 제목·일시·장소·본문 입력, 실시간 미리보기, 발행. 발행 후 초대장 링크와 응답 현황 링크를 보여주고 복사 버튼 제공 |
| `/i/[id]` | 게스트 | 초대장 보기, 이름·참석 여부·동반 인원 제출, 제출 후 완료 화면 |
| `/i/[id]/responses?key=[hostKey]` | 호스트 | 응답 목록, 참석·불참 수, 참석 인원 합계 |

### 흐름

- 호스트: 만들기 → 발행 → 링크 공유 → 응답 확인
- 게스트: 초대장 보기 → 응답 제출 → 완료

### 데이터 (Supabase)

- 초대장: 고유 ID, 호스트 키, 제목, 일시, 장소(주소 텍스트), 본문, 만든 시각
- 응답: 초대장 ID, 이름, 참석 여부, 동반 인원, 제출 시각
- 테이블 이름·컬럼을 실제로 정할 때는 사용자에게 확인받는다.

### MVP 범위

- 만들기 폼 / 발행 / 장소(주소를 글자로 표시) / 참석 응답 / 응답 현황 — 이 다섯 가지만 만든다.
- **만들지 않는 것:** 로그인, 결제, 네이티브 앱, 드래그 편집, 축의금 송금, AI 생성, 다국어, 지도 표시.

### 폴더

- `src/app/` — 페이지와 경로
- `src/components/` — 재사용 UI 컴포넌트
- `src/lib/` — Supabase 연결, 데이터 처리 함수
- `docs/` — PRD, 작업기록

## 5. 안전 규칙

### 데이터와 비밀 정보

- 초대장 ID와 호스트 키는 **추측하기 어려운 무작위 값**으로 만든다 (예: `crypto.randomUUID()`). 1, 2, 3 같은 순번을 쓰지 않는다.
- 응답 현황(`/responses`)은 **호스트 키가 맞을 때만** 보여준다. 키가 없거나 틀리면 응답을 보여주지 않는다. 호스트 키는 게스트 화면이나 게스트 링크에 절대 노출하지 않는다.
- Supabase 키 등 비밀 값은 `.env.local`에만 둔다. `.env*`는 커밋하지 않는다 (`.gitignore`에 이미 포함).
- `service_role` 키처럼 강한 권한의 키는 서버 코드에서만 쓰고, 브라우저로 가는 코드(`NEXT_PUBLIC_` 변수, 클라이언트 컴포넌트)에 넣지 않는다.
- 게스트 입력값(이름, 동반 인원 등)은 서버에서 다시 검사한다 (빈 값, 너무 긴 글자, 음수·비정상적으로 큰 인원).

### 작업 방식

- 파일 삭제, 대량 수정, 패키지 설치·삭제, DB 테이블 변경·데이터 삭제는 **먼저 사용자에게 묻고** 한다.
- `git push`, 배포, 외부 서비스에 무언가를 올리는 작업은 사용자가 요청할 때만 한다.
- 커밋은 사용자가 요청할 때만 한다. 커밋 메시지는 `type(scope): 설명` 형식을 따른다 (예: `feat(create): 만들기 폼 추가`). type은 `feat`·`fix`·`style`·`docs`·`chore` 중 하나이고, scope가 애매하면 생략한다 (예: `docs: 작업기록 정리`). 커밋·푸시는 `/commit-push` 스킬로 한다.
- 작업 후 `npm run lint`로 오류가 없는지 확인하고, 실패하면 숨기지 말고 결과를 그대로 알린다.

### 작업기록 남기기

작업이 끝나면 `docs/작업기록.md` **맨 아래**에 아래 형식으로 기록을 추가한다. 시간은 한국 시간(KST) 기준이다.

```markdown
## YYYY-MM-DD HH:MM — 작업 제목

- **무엇을 바꿨는지:** 바꾸거나 새로 만든 파일과 내용
- **왜:** 이 작업을 한 이유
- **직접 확인하는 방법:** 사용자가 따라 할 수 있는 단계 (실행할 명령, 열어볼 주소, 눌러볼 버튼 등)
```

- 질문에 답만 하고 파일을 바꾸지 않은 경우에는 기록하지 않는다.

---

<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
