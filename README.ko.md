# AI 버그 탐색 방법론

[English](README.md)

AI 코딩 에이전트가 프로젝트 전체에서 **실제로 재현되는 버그**를 찾도록 이끄는 단계별 방법론이에요. 단순한 코드 리뷰가 아니에요.

Claude Code, Codex, Cursor, Gemini CLI 같은 에이전트에게 주면 이렇게 진행해요.

1. 먼저 **범위 표**를 만들어요. 화면마다 들어오는 길, 나가는 길, 상태를 모두 적고, 방법을 몇 개 돌렸는지가 아니라 표의 칸이 다 채워졌는지로 범위를 재요.
2. **20가지 방법**을 적용해요. 피라미드형, 역피라미드형, 사용자 시나리오, 상태 전이, 경계값, 암묵적 가정, 데이터 흐름, 계약, 중복, 비동기·경쟁 조건, 권한 역방향, 변경점 회귀, 외부 동작 실측, 시간 제한, 서버 강제, 화면 이동 수명, 실패 뒤의 쓰기와 부수효과, 화면 일관성과 문구.
3. 첫 탐색 결과를 보지 않은 **다른 눈이 독립적으로 한 번 더** 찾아요.
4. 버그마다 **Confirmed / Suspected**로 나누고 심각도, 재현 방법, 원인, 파일, 실제로 어디까지 확인했는지를 적어요.
5. 놓친 버그가 나오면 **회고**해서 방법론을 고쳐요.

방법론 본문은 [`METHODOLOGY.ko.md`](METHODOLOGY.ko.md)(한국어)와 [`skills/bug-hunt/SKILL.md`](skills/bug-hunt/SKILL.md)(영어)예요. 다른 AI에게 줄 때는 해석 차이가 적은 영어판을 권해요.

## 설치

### Claude Code

```text
/plugin marketplace add Moulru/ai-bug-hunt-methodology
/plugin install bug-hunt@ai-bug-hunt-methodology
```

그다음 "이 프로젝트 전체 버그 탐색 해줘"라고 하면 돼요.

### 다른 에이전트 (Codex, Cursor, Gemini CLI, ChatGPT 등)

아래 파일을 주고 그대로 따르라고 하면 돼요.

```text
https://raw.githubusercontent.com/Moulru/ai-bug-hunt-methodology/main/skills/bug-hunt/SKILL.md
```

또는 [`AGENTS.md`](AGENTS.md)를 프로젝트 루트에 복사해 두세요.

## 기여

이 방법론이 잡았어야 할 버그를 찾으셨다면 이슈에 한 줄로 남겨 주세요. 어떤 버그인지, 어느 방법이 잡았어야 했는지, 왜 못 잡았는지, 무엇을 고치면 되는지. 방법 20번의 회고 그대로예요.

## 라이선스

[MIT](LICENSE)
