# Write-up

The full flow is in [diagram/process-flow.png](diagram/process-flow.png).

**The idea in one line:** the AI reads the proposals, plain code applies the rules and does the maths, and people check the result and make the final decision.

## 1. AI vs. rules

| Step | Done by | Why |
|---|---|---|
| Get text out of the PDF | Code | Mechanical task, no judgement needed |
| Look for hidden instructions | Code | Simple word patterns, cheap and predictable |
| Pull facts out of each proposal | **AI** | Every bidder writes differently; reading free text is what AI is good at |
| Check the AI's facts | Code | Every fact must come with a quote that really exists in the proposal |
| Must-have checks (licence, WSIB, insurance) | Code | These are yes/no; rules never change their mind |
| Points, total and ranking | Code | Maths must be exact and the same every time |
| Reason for each position | Code | Built from the real points, so the reason always matches the score |

I drew the line at **reading vs. deciding**. The AI only turns messy text into facts, each with evidence. Anything that decides an outcome is plain code that a reviewer can check by hand.

## 2. Consistency and explainability

- Scores come from fixed formulas, not AI opinion. The same facts always give the same score.
- The AI's answer is saved per file. If the same proposals are ranked again, the saved facts are reused, so **the ranking is the same.** If a proposal is re-read, any fact that changed shows up as a difference and is checked again.
- The model version and prompt version are fixed, and the AI must answer in a fixed format.
- The formulas (set by the admin):
  - **Experience, price, timeline:** the best bid gets full points, and the others get points in proportion. For price: cheapest ÷ their price × 25, the usual public-tender formula.
  - **Warranty:** 5 points for the panel warranty (25 years = full) plus 10 points for workmanship (best in the batch = full).
  - **Team:** 3.75 points each for a named lead, a crew, a safety plan and a monitoring plan.
- **What the reviewer sees:** the points for each criterion next to the exact sentence and page they came from, e.g. "Experience 30/30 — *14 commercial installations… since 2021* (p.1)".

## 3. Human in the loop

- **Admin** sets the criteria and weights. Saving is blocked unless the weights add up to 100%.
- **Reviewer** sees all five results. Flagged and ineligible ones are shown first. For each, the reviewer can:
  - fix any fact the AI got wrong (the code then re-scores it),
  - clear or keep a flag,
  - override a must-have check or a rank.

  Every change needs a written reason.
- **Award:** a person decides, outside the tool. The ranking is only advice.
- **Audit log** records: who and when, the file fingerprint (hash), criteria version, model and prompt version, the AI's raw answer, check results, points, flags, and every override (old value, new value, reason). Nothing in the log can be edited or deleted.

## 4. Changing criteria

- Every set of criteria is saved as a **version**. Editing creates v2; v1 never changes.
- Finished results stay linked to v1, so they can always be explained later.
- If only **weights or formulas** change, the tool re-scores from the saved facts straight away, with no new AI call, and shows the reviewer old vs. new.
- If a **new criterion** is added, the AI only reads that one new fact.
- A ranking is never built from a mix of v1 and v2.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| **AI reads a fact wrong** | Quote must match the proposal word for word; code re-checks numbers; reviewer sees the quote |
| **AI makes up a fact** | Rule: "if it's not written, leave it empty". No quote means no value. Missing facts are flagged, never guessed (Proposal D) |
| **Cheating / hidden instructions** | Proposal text is treated as data only. Code searches for suspicious phrases and the AI must report any instructions it sees. Anything found is flagged for a person (Proposal C). The AI can't change scores anyway, because it doesn't score |
| **Tricky wording on a must-have** | The licence must be held *by the bidder*. "A subcontractor, to be confirmed" fails (Proposal B). Unclear wording is flagged, not passed |
| **Bias** (nice writing, long proposals, famous names) | The AI only pulls out facts; formulas ignore style and length; bidder names are not used in scoring |
| **Scores shift when a bid is added** (price, experience, timeline compare bids to each other) | The whole batch is re-scored and logged; the admin can switch to fixed scales |
| **Cost** | One AI call per proposal; answers saved and reused; weight changes need no new AI call |
| **People trusting the tool too much** | Clearly labelled as advice; a person must confirm before anything is exported |

## How the five sample proposals come out

| | Must-haves | Exp /30 | Price /25 | Time /15 | Warranty /15 | Team /15 | Total | Result |
|---|---|---|---|---|---|---|---|---|
| A | Pass | 30.0 | 24.3 | 15.0 | 13.3 | 15.0 | **97.6** | Rank 1 |
| E | Pass | 17.1 | 23.8 | 13.8 | 15.0 | 15.0 | **84.8** | Rank 2 |
| C | Pass | 19.3 | 23.7 | 12.9 | 13.3 | 7.5 | (76.6) | Flagged: hidden instruction |
| D | Pass | 23.6 | 25.0 | 0 | 13.3 | 7.5 | (69.4) | Flagged: no finish date |
| B | **Fails licence** | – | – | – | – | – | – | Ineligible |

Bids are only compared with the four that passed the must-have checks, so B (cheapest, but not licensed) doesn't affect anyone's points.

**My decisions on the tricky cases, and why:**
- **D: flagged.** The 16-week limit is a hard requirement and D gave no date. Ranking D would be a guess, and rejecting it could be unfair, so a person should ask D for a date.
- **C: paused until a person checks it.** A bidder who tries to trick the system deserves a human look. C's score is already worked out, so it can be added back quickly.
- **C's "weekly progress reports"** are not monitoring of the finished system, so they earn no monitoring points.
- **C's "since 2020"** may include a year outside the 5-year window. The tool marks it for the reviewer to confirm.
