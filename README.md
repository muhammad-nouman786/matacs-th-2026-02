# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Muhammad Nouman
- **Reference:** MATACS-TH-2026-02
- **Repository:** https://github.com/muhammad-nouman786/matacs-th-2026-02
- **Time actually spent:** 50 minutes

## Files

| File | What it contains |
|---|---|
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow from upload to ranked output, showing AI steps, rule steps and human checkpoints |
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, plus how the sample set is scored |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt, exactly as it would be sent, and its output schema |
| [ranking.json](ranking.json) | Expected ranking of proposals A–E |
| [prototype/](prototype/README.md) | Optional: Python script that runs the prompt and the scoring rules ([.env.example](.env.example) for the API key) |

## Design in one line

The AI only reads the proposals and pulls out facts, each with a quote as evidence. Plain code checks the mandatory criteria and calculates the scores. A reviewer confirms or overrides the result, and a person makes the award.

## How I used AI tools

I used Claude (in Claude Code) to break down the brief and to draft the write-up, the prompt, the diagram and the prototype script. For the tricky cases (D with no finish date, C with the hidden instruction, and how to score price), I asked it for the options and the reasons for each, then chose which approach to use. One place I corrected the AI: its setup instructions loaded the API key with `export $(cat .env | xargs)`. I questioned whether that was the right way, and on review it is fragile (it breaks on values with spaces or quotes), so it was replaced with `set -a; source .env; set +a`.
