# Codex Project Guide

## Project

`trans-korean` is a Codex-first Korean AI-tell remover. It rewrites Korean text that reads like AI output while preserving meaning, facts, claims, numbers, proper nouns, and direct quotations.

## Product Rules

- Preserve meaning: do not change facts, claims, numbers, dates, proper nouns, legal text, formulas, or quoted text.
- Keep edits span-grounded: only revise passages tied to detected AI-tell findings.
- Preserve genre and register: do not turn reports into essays, columns into literary prose, or formal writing into casual speech.
- Avoid over-polishing: warn above 30% change rate and stop or roll back above 50%.
- Use taxonomy/playbook assets: keep `.agents/skills/humanize-korean/references/ai-tell-taxonomy.md`, `.agents/skills/humanize-korean/references/rewriting-playbook.md`, `.agents/skills/humanize-korean/references/scholarship.md`, and `.agents/skills/humanize-korean/references/web-service-spec.md` as the source of truth.

## Codex Layout

- `.agents/skills/humanize-korean/SKILL.md`: repo-local Codex skill for Korean AI-tell removal.
- `.agents/skills/humanize-korean/references/`: taxonomy, playbook, scholarship, web spec, metrics, and baselines.
- `.codex/agents/*.toml`: optional project-scoped role prompts for Strict mode only.
- `plugins/trans-korean-codex/`: distributable Codex plugin wrapper.
- `plugins/trans-korean-codex/skills/humanize-korean/`: bundled copy of the repo skill.
- `_workspace/{YYYY-MM-DD-NNN}/`: runtime outputs. Do not commit generated runs unless a maintainer explicitly asks.

## Build And Test

- Metrics tests: `python3 -m unittest discover -s tests`
- v1 metrics only: `python3 tests/test_metrics.py`
- v2 metrics only: `python3 tests/test_metrics_v2.py`
- Monolith input shim smoke: `python3 scripts/prepare_monolith_input.py --text "오늘은 비가 온다." --genre essay`
- Manifest parse check: parse `.codex/agents/*.toml`, `plugins/trans-korean-codex/.codex-plugin/plugin.json`, and `.agents/plugins/marketplace.json`.
- Skill drift check: `diff -ru .agents/skills/humanize-korean plugins/trans-korean-codex/skills/humanize-korean`
- Optional plugin validator dependencies: `python3 -m pip install -r requirements.txt`

The runtime code uses the Python standard library for metrics. Thumbnail scripts require Pillow and use local Pretendard fonts when available, with macOS Korean font fallback; they are not part of the core test path.

## Work Rules

- Keep documentation paths synchronized with actual files.
- Keep the skill id `humanize-korean`; only the repo/plugin brand should use `trans-korean`.
- Keep `.agents/skills/humanize-korean` and `plugins/trans-korean-codex/skills/humanize-korean` in sync when either copy changes.
- Do not reintroduce Claude Code slash commands, `.claude` user paths, or Claude-specific install instructions.
- Do not change product semantics while moving files or renaming wrappers.
- Do not invent new taxonomy rules without preserving evidence and version notes in the reference files.
- Prefer small, local edits. Avoid broad refactors unrelated to Codex migration.
- Before release, grep for personal absolute paths, file URLs, Codex deeplinks, and stale old-name paths in README, AGENTS, skills, plugin manifests, scripts, and tests.
