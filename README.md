---
id: 20261029-logs-readme
title: Defense.Logs — 세션 로그
summary: Defense.Logs 의 prompts · sessions 폴더를 훅이 쓰는 방식을 적은 문서로, UserPromptSubmit · Stop · SessionEnd 훅과 log_prompt.py · digest_session.py 의 연결, settings.json 훅 등록 설정, 수동 실행법, 내부 추론이 전사에 거의 없다는 한계를 담는다.
doc_type: guide
status: final
lifecycle: living
created: 2026-10-29
updated: 2026-10-31
---
# Defense.Logs — 세션 로그

Claude Code 훅이 쓰는 저장소다(G-05). 사람과 세션은 `prompts/` · `sessions/` 를 손으로 고치지 않는다. 토큰 비용은 0 — 훅은 모델 밖에서 돈다. 스크립트 정본은 `../../../_공통/_도구/`.

| 폴더 · 파일 | 무엇 | 누가 쓰나 |
|---|---|---|
| `prompts/YYYYMMDD.md` | 그날 입력한 프롬프트 전문(시각 · 세션 8자 · 작업 폴더) | `UserPromptSubmit` → `log_prompt.py` |
| `sessions/YYYYMMDD_<세션8>.md` | 세션 다이제스트 — 프롬프트 · 답변 · thinking(있는 만큼) · 도구 호출 · 결과 앞 600자 · 변경 파일 · 명령 목록 | `Stop`(턴마다 덮어쓰기) · `SessionEnd`(확정 + 로컬 커밋) → `digest_session.py` |
| `hook_errors.log` | 훅 실패 기록(git 밖, 있으면 문제) | 스크립트 |

## 훅 등록 (작업 공간 `.claude/settings.json` 사본 — git 밖, 이 Mac 에만)

```json
{
  "hooks": {
    "UserPromptSubmit": [{"hooks": [{"type": "command", "timeout": 10,
      "command": "python3 -I /Users/devman/_Projects/_Personal/Games/_공통/_도구/log_prompt.py /Users/devman/_Projects/_Personal/Games/Defense/z.docs/Defense.Logs"}]}],
    "Stop": [{"hooks": [{"type": "command", "timeout": 30,
      "command": "python3 -I /Users/devman/_Projects/_Personal/Games/_공통/_도구/digest_session.py /Users/devman/_Projects/_Personal/Games/Defense/z.docs/Defense.Logs"}]}],
    "SessionEnd": [{"hooks": [{"type": "command", "timeout": 60,
      "command": "python3 -I /Users/devman/_Projects/_Personal/Games/_공통/_도구/digest_session.py /Users/devman/_Projects/_Personal/Games/Defense/z.docs/Defense.Logs"}]}]
  }
}

```

- 훅은 **세션 시작 때** 읽힌다. 고치면 세션을 새로 연다.
- 세션은 작업 공간 루트(`~/_Projects/_Personal/Games/Defense`)에서 연다.
- `UserPromptSubmit` 의 stdout 은 대화에 들어가므로 스크립트는 아무것도 출력하지 않는다.

## 수동 실행

훅이 돌지 않은 세션은 전사를 직접 넣는다.

```bash
python3 -I ~/_Projects/_Personal/Games/_공통/_도구/digest_session.py ~/_Projects/_Personal/Games/Defense/z.docs/Defense.Logs ~/.claude/projects/<프로젝트 폴더>/<세션>.jsonl
```

결과 길이는 환경 변수 `GAMES_LOG_RESULT_CHARS`(기본 600)로 바꾼다.

## 한계

- 내부 추론(thinking)은 전사에 거의 없다(세션당 0~4K자 실측). 「왜」는 Planning 의 결정기록 · 검토서 · 작업 보고에 세션이 명시적으로 쓴다.
- 서브에이전트의 전사는 별도 파일이라 다이제스트에 들어가지 않는다. 주 세션의 위임 프롬프트와 보고 요약만 남는다.
- 비밀값 걸러내기는 하지 않는다. 이 저장소는 공개다(`_공통` G-15) — 세션이 토큰 · 키 · 개인 이메일을 출력하지 않는 것이 방어선이고, 붙여 넣은 것이 있으면 세션 종료 때 사용자가 지운다. 첫 푸시 전에 전체 이력을 한 번 검사했다(G-19, 2026-10-31) — 다이제스트 3곳에 남아 있던 개인 이메일은 이력 재작성으로 `[개인 이메일 가림]` 이 됐다(G-16). `prompts/20261030.md` 의 비밀번호 · 개인 키 꼴 문자열은 검사 도구 리뷰 보고에 실린 시험용 가짜 값이다.
