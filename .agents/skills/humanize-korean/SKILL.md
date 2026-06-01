---
name: humanize-korean
description: 한글 AI 티 제거, GPT 문체 제거, ChatGPT 문체 제거, 번역투 제거, 사람이 쓴 것처럼 윤문, AI 같은 한국어 문장 자연화, Korean humanizer, 한국어 AI 탐지·윤문 요청에 사용한다. 의미·사실·수치·고유명사·직접 인용은 보존하고, 탐지된 span에 근거한 국소 수정만 수행한다. 단순 맞춤법 교정이나 번역, 새 내용 추가가 필요한 글쓰기는 이 스킬 대상이 아니다.
---

# Humanize Korean

Codex에서 한글 AI 티를 탐지하고 자연스러운 한국어로 윤문한다. 이 스킬의 목표는 "다른 글"을 쓰는 것이 아니라, 원문 의미와 장르를 유지한 채 AI 생성문 특유의 번역투·기계적 구조·상투 표현·균일한 리듬을 줄이는 것이다.

## Core Rules

1. **의미 불변**: 사실, 주장, 수치, 날짜, 고유명사, 직접 인용, 법률 조문, 공식은 보존한다.
2. **Span-grounded 수정**: 탐지 finding에 연결되는 구간만 국소 수정한다. 탐지 없는 문장은 원칙적으로 건드리지 않는다.
3. **장르 유지**: 칼럼, 리포트, 블로그, 공적 문서, 학술적 설명 등 원문 장르와 register를 유지한다.
4. **과윤문 금지**: 변경률 30% 초과 시 경고하고, 50% 초과 시 중단 또는 롤백한다.
5. **Do-NOT list**: 수치·단위·날짜·인명·기관명·제품명·모델명·큰따옴표 안 직접 인용·법률/규정 조문·불가피한 학술 개념어는 윤문 대상에서 제외한다.

## References

Load only what the task needs.

- Fast mode: read `references/quick-rules.md`.
- Strict detection: read `references/ai-tell-taxonomy.md`.
- Strict rewriting: read `references/rewriting-playbook.md`.
- Scholarship/evidence work: read `references/scholarship.md`.
- Web-service design: read `references/web-service-spec.md`.
- Metrics/scripts: use `references/metrics.py`, `references/metrics_v2.py`, `references/baseline.json`, and `references/baseline_v2.json`.

## Mode Selection

- **Fast mode** is default for ordinary requests and text under about 5,000 Korean characters.
- **Strict mode** is required when the user says "정밀", "Strict", "5단계", asks for a redo/partial rewrite, provides long text over about 8,000 characters, or when content fidelity risk is high.
- If the user asks only for taxonomy maintenance or web architecture, do not run the humanizing pipeline; load the relevant reference and answer that task.

At the start of a run, tell the user:

```text
humanize-korean — {Fast|Strict} mode / run_id: {YYYY-MM-DD-NNN}
```

Create outputs under `_workspace/{run_id}/` relative to the repository or current working directory.

## Fast Mode

Use the single-skill path. Do not spawn subagents in Fast mode.

1. Save the original text to `_workspace/{run_id}/01_input.txt`.
2. Read `references/quick-rules.md`.
3. Detect S1/S2 AI-tell spans in memory.
4. Rewrite only detected spans, preserving meaning and genre.
5. Self-check the result against the Core Rules.
6. Write:
   - `_workspace/{run_id}/final.md`
   - `_workspace/{run_id}/summary.md`

`summary.md` should include mode, genre hint, change-rate estimate, before/after category summary, fidelity self-check, quality grade, and 3-5 short before/after highlights.

## Strict Mode

Strict mode keeps responsibilities separated. Use project-scoped Codex subagents only if the current Codex environment supports explicit subagent spawning. If subagents are unavailable, perform the same roles sequentially in the main Codex session and keep the same artifacts.

Only Strict mode may use role separation or subagents.

1. **Detection**: use `ai-tell-detector` role and `references/ai-tell-taxonomy.md`; write `_workspace/{run_id}/02_detection.json`.
2. **Rewrite**: use `korean-style-rewriter` role and `references/rewriting-playbook.md`; write `_workspace/{run_id}/03_rewrite.md` and `_workspace/{run_id}/03_rewrite_diff.json`.
3. **Content audit**: use `content-fidelity-auditor` role; write `_workspace/{run_id}/04_fidelity_audit.json`.
4. **Naturalness review**: use `naturalness-reviewer` role; write `_workspace/{run_id}/05_naturalness_review.json`.
5. **Arbitrate**:
   - `accept`: copy the approved rewrite to `final.md`.
   - `accept_with_note`: finalize with residual notes.
   - `rewrite_round_2`: rerun the rewrite only for target findings.
   - `rollback_and_rewrite`: roll back risky edits, then rerun targeted rewrite.
   - `hold_and_report`: stop and explain why human review is needed.
6. Write `summary.md` with the final verdict and evidence.

Limit rewrite loops to 3 rounds. After that, use `hold_and_report`.

## Redo And Partial Requests

Treat these as Strict mode:

- "이 문단만 다시"
- "번역투만 더 손봐줘"
- "관용구만 다시"
- "윤문 강도 낮춰줘"
- "2차 윤문해줘"
- "이 변경 되돌려줘"

Reuse the previous run when the user gives a run_id. Otherwise, find the latest `_workspace/*/final.md` only if it is clearly in the current project.

## Output Contract

Return the polished text to the user and keep durable files in `_workspace/{run_id}/`.

Fast mode minimum artifacts:

- `01_input.txt`
- `final.md`
- `summary.md`

Strict mode artifacts:

- `01_input.txt`
- `02_detection.json`
- `03_rewrite.md`
- `03_rewrite_diff.json`
- `04_fidelity_audit.json`
- `05_naturalness_review.json`
- `final.md`
- `summary.md`

## Quality Grades

- **A**: S1 residual 0, S2 residual <= 2, major score improvement, no over-polish.
- **B**: S1 residual 0, S2 residual <= 4, acceptable improvement.
- **C**: S1 residual 1-2 or over-polish signals; recommend targeted rewrite.
- **D**: S1 residual 3+ or serious fidelity/over-polish risk; use `hold_and_report`.
