# session-board

A Claude Code mod that keeps a session's goal, work in progress, waits, remaining work and finished work
beside the conversation. Its on-screen text is in Korean.

Claude Code 세션의 목표, 진행중, 기다림, 남음, 완료를 대화 옆 pane 에 보여 주는 mod 입니다. 작업이 길어져도 이 세션이
무엇을 하고 있고 무엇이 남았는지 한눈에 볼 수 있습니다.

```text
대화                                     │목표
                                         │세션 현황을 대화 옆에 보여 주는 mod 완성
                                         │완료 기준
                                         │테스트 통과, 실제 세션에서 확인
                                         │2분 전 갱신
                                         │
                                         │▶ 진행중 1
                                         │#3 실제 세션에서 동작 확인
                                         │
                                         │⏸ 기다림 1
                                         │#4 표시 위치 결정 [사용자]
                                         │
                                         │☐ 남음 1
                                         │#5 문서 작성
                                         │
                                         │✓ 완료 2
                                         │#1 기능 작성
                                         │#2 자동 시험
```

## 설치

터미널 세션의 프롬프트에서 입력합니다.

```text
/plugin install session-board --marketplace LJLee37/session-board
```

마켓플레이스를 더할지 물으면 `y`, 이어서 범위(scope)를 고릅니다.

셸에서는 이렇게 합니다.

```sh
claude plugin install session-board --marketplace LJLee37/session-board
```

데스크톱 앱에서는 `/plugin` 이 동작하지 않습니다. 프롬프트 입력창 옆의 + 를 누르고 Plugins, Add marketplace 에
`LJLee37/session-board` 를 넣은 뒤 Add plugin 을 누릅니다.

mod API 는 early access 라 예고 없이 바뀔 수 있습니다. 2.1.317 의 실제 세션에서 시험했고, 2.1.312 와 2.1.318 에서는
`claude plugin validate` 와 `claude plugin test` 만 돌렸습니다.

## 쓰는 법

보드는 Claude 가 `mcp__session-board__update` 도구로 적습니다. 도구 설명에 언제 적을지가 들어 있고, 작업했는데
갱신하지 않으면 다음 프롬프트에 알림이 붙습니다. 더 확실하게 하려면 `CLAUDE.md` 에 규칙을 적어 두세요.

```md
## 세션 보드

- 사소하지 않은 작업은 시작할 때 목표와 완료 기준을 적고, 계획이 승인되면 항목으로 풉니다.
- 항목 상태가 바뀌면 바로 갱신합니다. 완료에는 근거(커밋 해시, 테스트 결과, 경로)를 붙입니다.
- 기다리는 항목에는 누구를 기다리는지 적습니다.
- 목표가 바뀌면 덮어쓰지 않고 새 목표(`startNewGoal`)로 올립니다.
```

사용자는 다음 명령을 씁니다.

| 명령 | 하는 일 |
|---|---|
| `/board` | pane 을 엽니다 |
| `/board show` | 보드를 텍스트로 보여 줍니다 |
| `/board goal <내용>` | 목표를 바꿉니다 |
| `/board add <내용>` | 항목을 더합니다 |
| `/board done <번호>` | 항목을 완료로 표시합니다 |

## 어디에 보이는가

- **fullscreen 이고 110칸 이상:** 대화 옆에 붙습니다(`/board` 로 연 경우). 터미널의 45% 안에서 42칸을 씁니다.
- **그보다 좁거나 fullscreen 이 아닐 때:** `/board` 로 열면 프롬프트 위에 붙습니다.
- **요청 없이 저절로 열리는 조건:** fullscreen 이고 144칸 이상일 때입니다. 139칸 터미널에서는 `/board` 를 한 번 입력해야 했습니다.
- **pane 이 화면에 없을 때:** 프롬프트 위에 한 줄로 보여 줍니다.

```text
🎯 목표 │ ▶ 첫 진행항목 외 1 │ ⏸ 1 │ ☐ 2 │ ✓ 3 │ 2분 전 갱신
```

손으로 닫으면 그 세션에서는 저절로 다시 열지 않습니다. `/board` 로 언제든 다시 엽니다.

대화 옆의 어느 쪽에 붙을지는 mod 가 고르지 못하고 Claude Code 가 정합니다. 시험한 환경에서는 오른쪽이었습니다.

## 동작

- 보드는 세션마다 `~/.claude/session-boards/<세션 id>.json` 에 저장됩니다. 저절로 지워지지 않습니다.
- 시스템 프롬프트에 넣지 않습니다. 대화가 요약(compact)되거나 재개되면 다음 프롬프트나 도구 호출에 다시 알립니다.
- 비대화형 세션(`claude -p`, SDK)에는 도구, 명령, 알림을 등록하지 않습니다.
- 서브에이전트는 보드를 쓸 수 없습니다.
- 보드의 오류가 다른 도구나 프롬프트를 막지 않게 만들었습니다.

## 아직 시험하지 못한 것

- auto 모드가 아닐 때 이 도구에 권한 창이 뜨는지. 문서대로라면 뜨지 않아야 합니다.
- `--resume` 뒤에 같은 보드로 이어지는지.
- Remote Control 화면에 보이는지. 안 보이면 `/board show` 를 쓰세요.
- 터미널 폭을 바꿀 때의 전환.

## 개발

```sh
claude plugin validate .
claude plugin test .
```

타입 검사는 mod 가 한 번 로드된 뒤 `tsc -p .` 로 합니다. `.claude-plugin/types/` 와 `tsconfig.json` 이 참조하는 타입은
엔진이 만듭니다.
