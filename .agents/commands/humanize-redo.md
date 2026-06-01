# humanize Redo Codex Prompt

Use this prompt after an existing `_workspace/{run_id}/` result:

```text
humanize-korean skill로 이전 결과를 Strict 모드에서 부분 재검토해줘.
대상 run_id: YYYY-MM-DD-NNN
조정 지시: 번역투만 다시 / 이 문단만 / 강도 낮춰 / 2차 윤문
```

Redo requests always use Strict mode so every additional edit is tied to a finding and rechecked for content fidelity.
