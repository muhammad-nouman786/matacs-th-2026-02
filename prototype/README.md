# Prototype

A small Python script that runs the design end to end. The AI is used for one step only: extracting facts from each proposal. Everything after that is plain code.

| File | What it is |
|---|---|
| [triage.py](triage.py) | Pre-scan → AI extraction → quote validation → gates → scoring → ranking → audit log |
| [criteria.json](criteria.json) | What the admin screens would save: fields, mandatory gates, weights, formulas, version |
| [fixtures/facts.json](fixtures/facts.json) | Facts for A–E as plain values (no proposal text), for running without an API key |

## Run offline (no API key needed)

Scores the pre-extracted facts. The output matches [../ranking.json](../ranking.json).

```bash
python3 prototype/triage.py --facts prototype/fixtures/facts.json
```

## Run with Claude (real extraction)

The brief says not to commit the proposal text, so the `samples/` folder is git-ignored. To run it:

1. Put each proposal's text in `prototype/samples/A.txt` … `E.txt`.
2. Set up the key and install the SDK (Python 3.10+):

   ```bash
   cp .env.example .env
   # open .env and replace your-key-here with your real key
   set -a; source .env; set +a
   pip install -r prototype/requirements.txt
   python3 prototype/triage.py --samples prototype/samples
   ```

The script writes these files:
- `prototype/out/ranking.json`: the result, in the brief's format
- `prototype/out/audit.jsonl`: an append-only log containing the file hash, model, prompt version, raw AI output, validation issues, gates, points and status
- `prototype/out/cache/`: AI extractions saved by file hash, so a re-run gives the same answer and costs nothing

## What the code checks

- **Injection:** text is matched against the admin's patterns, and the AI also reports any instructions it saw. Either one flags the proposal.
- **Made-up facts:** a value is thrown away if its quote is not found word for word in the proposal.
- **Wrong numbers:** each number is checked against the digits in its quote.
- **Refusal:** if the model declines, the run stops with an error instead of guessing. Server-side fallback to another model is turned on.
