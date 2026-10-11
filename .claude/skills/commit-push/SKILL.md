---
name: commit-push
description: 지금까지의 변경사항을 쉬운 말로 보여주고, 비밀 값 검사와 lint·build 검사를 한 뒤 한국어 커밋 메시지를 제안하고, 사용자가 확인하면 커밋하고 push한다. 사용자가 "/commit-push"로 부를 때만 사용한다.
disable-model-invocation: true
---

# commit-push — 변경사항 확인 후 커밋·푸시

사용자는 코딩 초보자다. 모든 설명은 **한국어**로, 전문 용어는 처음 나올 때 한 줄로 풀어 쓴다.
아래 단계를 **순서대로** 진행하고, "멈춘다"라고 적힌 곳에서는 다음 단계로 넘어가지 않는다.

## 절대 규칙

- `git push --force`, `git push -f`, `--force-with-lease` 등 **강제 push는 어떤 경우에도 쓰지 않는다.** 사용자가 요청해도 쓰지 않고, 왜 위험한지 설명한다.
- 사용자가 **명시적으로 확인하기 전에는** `git add`, `git commit`, `git push`를 하지 않는다.
- `git add -A`, `git add .`처럼 한꺼번에 넣지 않는다. 확인받은 파일만 이름을 적어서 넣는다.
- `git reset --hard`, `git checkout -- .`, `git clean`처럼 작업 내용을 지우는 명령은 쓰지 않는다.
- `--no-verify`로 검사를 건너뛰지 않는다.
- 실패한 결과는 숨기지 않고 그대로 보여준다.

## 1. 바뀐 파일 보여주기

1. `git status`와 `git diff --stat`을 실행한다. 새 파일은 내용을 직접 읽어 무엇인지 파악한다.
2. 바뀐 것이 없으면 "커밋할 변경사항이 없어요."라고 알리고 **멈춘다.**
3. 현재 브랜치 이름을 알려준다.
4. 파일마다 아래처럼 쉬운 말로 정리해 보여준다.

```markdown
| 상태 | 파일 | 무엇이 바뀌었나 |
|---|---|---|
| 수정 | AGENTS.md | 커밋 메시지 규칙 문장을 고침 |
| 새 파일 | .claude/agents/ui-reviewer.md | UI 리뷰 에이전트 설명서 |
| 삭제 | ... | ... |
```

- 상태 표기: `M` → 수정, `??` → 새 파일(아직 git이 모르는 파일), `D` → 삭제, `R` → 이름 변경.

## 2. 비밀 값 검사

아래 중 하나라도 있으면 **멈추고** 어떤 파일의 몇 번째 줄인지 알린다. 값 전체를 화면에 다시 출력하지 말고 앞 몇 글자만 보여준다 (예: `eyJhbGc...`).

**파일 이름**
- `.env`, `.env.local`, `.env.*` (단, `.env.example`처럼 빈 예시 파일은 내용을 확인해서 실제 값이 없으면 통과)
- `*.pem`, `*.key`, `id_rsa`, `credentials.json`, `service-account*.json`

**파일 내용** (`git diff`와 새 파일 내용에서 찾는다)
- `SUPABASE_SERVICE_ROLE_KEY`, `service_role`, `SECRET`, `PASSWORD`, `PRIVATE_KEY`, `API_KEY`, `TOKEN` 뒤에 실제 값이 붙어 있는 줄
- `eyJ`로 시작하는 긴 문자열 (JWT — Supabase 키가 이런 모양이다)
- `sk-`, `sb_secret_`, `ghp_`, `github_pat_`, `AKIA`로 시작하는 값
- `-----BEGIN ... PRIVATE KEY-----`
- `postgres://` 또는 `postgresql://` 주소 안의 비밀번호

멈췄을 때는 "이 값은 `.env.local`에만 두고 커밋하지 않아야 해요"처럼 이유를 설명하고, 어떻게 할지 사용자에게 묻는다.
변수 이름만 있고 값이 `process.env.XXX`처럼 환경 변수에서 읽어오는 코드는 문제 아님.

## 3. lint·build 검사

1. `npm run lint`를 실행한다.
2. `npm run build`를 실행한다. (lint: 코드 규칙 검사 / build: 실제 서비스용으로 만들어 보는 검사)
3. 하나라도 실패하면 **멈추고**, 오류 내용을 그대로 보여준 뒤 어떤 파일의 어떤 문제인지 쉬운 말로 설명한다. 고치는 것은 사용자가 원할 때만 한다.

## 4. 커밋 메시지 제안

형식: `type(scope): 한국어 설명` — scope(바뀐 범위)는 분명할 때만 쓰고, 애매하면 생략해 `type: 설명`으로 쓴다.

| type | 언제 |
|---|---|
| `feat` | 새 기능·화면 추가 |
| `fix` | 버그 수정 |
| `style` | 화면 모양·스타일만 바꿈 (기능 변화 없음) |
| `docs` | 문서만 바꿈 (`docs/`, `AGENTS.md`, `*.md`) |
| `chore` | 설정·도구·에이전트·스킬 등 기타 작업 |

- scope 예시: `create`, `invite`, `responses`, `supabase`, `agents`, `skills`
- 설명은 50자 이내, "~추가", "~수정"처럼 명사형으로 끝낸다.
- 성격이 다른 변경이 섞여 있으면 커밋을 나누자고 제안하고, 파일을 어떻게 나눌지 보여준다.
- 메시지 마지막 줄에는 빈 줄 하나 뒤에 아래 줄을 붙인다:
  `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`

## 5. 사용자 확인 (반드시 멈춤)

아래를 한 번에 보여주고 **사용자의 답을 기다린다.**

```markdown
**커밋할 파일**
- AGENTS.md
- .claude/agents/ui-reviewer.md

**커밋 메시지**
chore(agents): UI 리뷰 에이전트 추가

**push할 곳:** origin / main

이대로 커밋하고 push할까요? 파일이나 메시지를 바꾸고 싶으면 알려주세요.
```

- 사용자가 "응", "좋아", "진행해"처럼 **분명히 동의**할 때만 다음 단계로 간다.
- 수정 요청이 오면 반영해서 이 단계를 다시 보여준다.
- 커밋만 하고 push는 하지 말라고 하면 6번까지만 한다.

## 6. 커밋

1. 확인받은 파일만 `git add <파일1> <파일2>`로 넣는다.
2. 메시지는 HEREDOC으로 넘긴다:
   ```bash
   git commit -m "$(cat <<'EOF'
   chore(agents): UI 리뷰 에이전트 추가

   Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
   EOF
   )"
   ```
3. 커밋이 실패하면 오류를 보여주고 멈춘다.

## 7. push

1. `git push`를 실행한다. 원격에 아직 이 브랜치가 없으면 `git push -u origin <브랜치>`를 쓴다.
2. push가 거절되면 (예: 원격에 내 컴퓨터에 없는 커밋이 있음) **강제 push하지 말고 멈춘다.** 무슨 상황인지 쉽게 설명하고, `git pull`로 먼저 받아올지 사용자에게 묻는다.
3. 로그인·권한 오류가 나면 사용자가 직접 해야 할 일을 복사할 수 있는 명령으로 알려준다.

## 8. 마무리

- 커밋 해시(짧은 7자리), 커밋 메시지, push한 브랜치를 알려준다.
- 마지막으로 `git status`를 실행해 남은 변경사항이 있으면 알려준다.
- 이 스킬로 커밋·push만 한 경우에는 `docs/작업기록.md`에 따로 기록하지 않는다.
