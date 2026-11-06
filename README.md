# session-board

A Claude Code mod that keeps a session's goal, work in progress, waits, remaining work and finished work
beside the conversation. Its on-screen text is in Korean.

Claude Code 세션의 목표, 진행중, 기다림, 남음, 완료를 대화 옆 pane 에 보여 주는 mod 입니다. 작업이 길어져도 이 세션이
무엇을 하고 있고 무엇이 남았는지 한눈에 볼 수 있습니다. 노드의 모든 세션을 한 표로 보는 셸 명령 `session-boards` 도
있어서, 재부팅 뒤 어느 세션을 다시 켤지 고를 수 있습니다.

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

셸 명령 `session-boards` 는 플러그인이 PATH 에 올려 주지 않으니 따로 복사합니다. bash, jq, GNU coreutils 가 있는
Linux 에서 돕니다.

```sh
git clone https://github.com/LJLee37/session-board.git
install -m 755 session-board/bin/session-boards ~/.local/bin/
```

mod API 는 early access 라 예고 없이 바뀔 수 있습니다. 0.1.0 은 2.1.317, 0.2.0 은 2.1.318 의 실제 세션에서 시험했습니다.
2.1.312 에서는 0.1.0 의 `claude plugin validate` 와 `claude plugin test` 만 돌렸습니다.

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

## 노드 전체 보기

`session-boards` 는 이 노드의 Claude Code 세션을 표로 보여 줍니다. Claude 없이 돌고 아무것도 쓰지 않으니, 재부팅 직후
세션이 하나도 없을 때도 씁니다. 아래는 가짜 세션으로 만든 예입니다.

```text
이름                   폴더        목표                진행  기다림  남음  완료  갱신       프로세스  상태
테스트 실패 원인 조사  ~/work      -                   -     -       -     -     대화 방금  -         보드 없음
app-3f                 ~/work/app  로그인 화면 고치기  1     0       1     1     5분 전     48211     -
6f1c9d3a               ~/work/pay  결제 모듈 정리      0     1       1     1     2일 전     -         다시 켤 후보,낡음

다시 켤 후보 1개 (일이 남았는데 돌고 있지 않은 세션)
  6f1c9d3a: 결제 모듈 정리
    cd ~/work/pay && claude --resume 6f1c9d3a-2222-4b7e-8a41-0c5d7e2f9a13

보드 없이 멈춘 세션 1개 (남은 일이 있는지는 모름)
  테스트 실패 원인 조사
    cd ~/work && claude --resume a3e85b20-3333-4f0d-b6c2-9e7a1d4c5f28
```

- **다시 켤 후보:** 보드에 일이 남았는데 프로세스가 없는 세션입니다. 재개 명령을 붙입니다. 보드가 없는 세션은 남은 일을 알 수
  없어 따로 모읍니다.
- **이름:** 돌고 있으면 Claude Code 가 붙인 세션 이름, 아니면 대화 제목, 둘 다 없으면 id 앞부분입니다. 세션 이름은 재개하면
  바뀌어서 보드에 적지 않습니다.
- **낡음:** 보드가 24시간 넘게 갱신되지 않은 것입니다(`--stale-hours N`).
- **숨기는 것:** 끝난 보드, 비대화형 실행(`claude -p`, SDK), 대화가 7일 넘게 바뀌지 않은 보드 없는 세션(`--days N`)입니다.
  `--all` 로 모두 봅니다.
- **`--tsv`:** 모든 칸과 모든 세션의 재개 명령을 탭으로 나눠 냅니다.

읽는 곳은 `~/.claude`(또는 `$CLAUDE_CONFIG_DIR`) 아래 세 곳입니다.

- `session-boards/<id>.json`: 이 mod 의 보드입니다.
- `projects/*/<id>.jsonl`: 대화 파일에서 첫 `cwd`, 첫 `entrypoint`, 마지막 제목만 읽습니다.
- `sessions/<pid>.json`: 돌고 있는지만 봅니다. pid 가 살아 있고 시작 시각이 파일과 같아야 돌고 있는 것으로 칩니다. 그래서
  전원이 끊겨 남은 파일이나 재사용된 pid 는 세지 않습니다. 옆의 `.key` 파일은 읽지 않습니다.

## 동작

- 보드는 세션마다 `~/.claude/session-boards/<세션 id>.json` 에 저장됩니다. 저절로 지워지지 않습니다.
- 0.2.0 부터 보드에 세션이 시작한 폴더를 적습니다. 처음 정한 폴더는 바꾸지 않습니다.
- 보드 폴더와 파일은 처음 저장할 때 다른 사용자가 읽지 못하게(`chmod go-rwx`) 맞춥니다.
- 시스템 프롬프트에 넣지 않습니다. 대화가 요약(compact)되거나 재개되면 다음 프롬프트나 도구 호출에 다시 알립니다.
- 비대화형 세션(`claude -p`, SDK)에는 도구, 명령, 알림을 등록하지 않습니다.
- 서브에이전트는 보드를 쓸 수 없습니다.
- 보드의 오류가 다른 도구나 프롬프트를 막지 않게 만들었습니다.

## 아직 시험하지 못한 것

- auto 모드가 아닐 때 이 도구에 권한 창이 뜨는지. 문서대로라면 뜨지 않아야 합니다.
- Remote Control 화면에 pane 과 한 줄 표시가 보이는지. 안 보이면 `/board show` 를 쓰세요.
- 터미널 폭을 바꿀 때의 전환.
- 갑작스런 전원 끊김 뒤 남은 등록 파일. 시작 시각을 맞춰 걸러 내지만 가짜 파일로만 시험했습니다.

재부팅하고 `--resume` 으로 재개한 세션은 같은 보드를 이어 읽었습니다(2.1.318). `/board show` 는 Remote Control 에서도
동작했습니다(2.1.318).

## 개발

```sh
claude plugin validate .
claude plugin test .
bash tests/session-boards.test.sh
shellcheck bin/session-boards tests/session-boards.test.sh
```

타입 검사는 mod 가 한 번 로드된 뒤 `tsc -p .` 로 합니다. `.claude-plugin/types/` 와 `tsconfig.json` 이 참조하는 타입은
엔진이 만듭니다.

## 라이선스와 연락

[MIT](LICENSE). 문의는 GitHub 이슈나 LJLee37 (ljlee3759@gmail.com) 으로 주세요.
