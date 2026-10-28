# Write-up

**Assumptions:** the reference date is the submission date; "last 5 years" is measured from it. Flagged
proposals are not ranked until a human resolves them.

## 1. AI vs. rules
**AI (one step):** reading messy proposal text and extracting facts with evidence quotes. This is
language work that rules cannot do well.

**Rules / plain code:** PDF text extraction, loading the admin's criteria, quote verification, mandatory
pass/fail checks, all score maths, ranking and the audit log. These must be exact and repeatable, and I
don't want a model making money-affecting decisions. The line: AI *reads*, code *decides*.

## 2. Consistency and explainability
- AI runs at temperature 0 with a fixed prompt version and JSON schema, so the same text gives the same
  facts in practice. Extraction results are stored, so re-ranking never re-calls the model.
- Scores come from fixed formulas on stored facts. **Same stored facts + same criteria version = identical
  ranking, always.**
- Formulas used for the sample: price = (ceiling - price) / (ceiling - lowest eligible price); timeline =
  (16 - weeks) / (16 - fastest weeks); experience = installs / 12 (capped at 100%, the 12 being an admin
  setting); warranty = average of panel years / 25 and workmanship years / 12; team = 25% each for named
  lead, crew size, safety plan, monitoring plan.
- Each result shows a table: criterion, extracted value, evidence quote, points. The reviewer sees exactly
  why a proposal ranked where it did.

## 3. Human in the loop
- **Review queue:** every flagged proposal (missing data, unclear dates, suspicious text, failed quote
  check) goes to a reviewer first. They can request clarification, correct a value (with a reason), or
  confirm.
- **Ineligible proposals** can be reinstated by a reviewer with a written reason.
- **Final award** is always a human decision. The ranked list is advice only.
- **Audit log** records: who uploaded and reviewed, file hash, criteria version, prompt version, model
  name, raw AI output, verified evidence, every override (old value, new value, who, when, why), and the
  final decision.

## 4. Changing criteria
Criteria are versioned. Edits create a new version; old scores stay linked to the version that produced
them and are labelled so. Because extracted facts are stored separately from scoring, applying new
weights is a cheap re-calculation with no new AI calls. If the admin adds a *new* criterion that needs new
facts, only extraction is re-run for that field. The UI shows "scored under v2, current is v3" and offers
a re-rank. Rankings already used for an award are never changed.

## 5. Risks and safeguards
| Risk | Safeguard |
|---|---|
| Wrong extraction | Evidence quote must match source text exactly; schema checks; low-confidence goes to reviewer |
| Made-up facts | "Extract only what is written", null for gaps; missing data flagged, never guessed |
| Prompt injection (as in Proposal C) | Proposal text treated as data; model told to ignore it and report it; code ignores any model "score"; suspicious text flags for human review |
| Bias (large vendors, wording style) | Scoring uses only numeric facts and checklists, never "quality of writing"; same formulas for all |
| Gaming (e.g. B's unconfirmed subcontractor) | Mandatory criteria require the bidder to state it holds the licence |
| Cost | One extraction call per proposal, cached; re-ranking costs nothing; size limits on uploads |
| Scanned/poor PDFs | If extracted text is too short, flag for manual entry instead of guessing |
| Privacy | Only fictional/consented data sent to the model; no personal data in logs beyond reviewer IDs |

## Sample set result
- **A (92.7)** ranked 1, **E (76.3)** ranked 2.
- **B** ineligible: licence is not held by the bidder.
- **C** flagged: embedded instruction ignored and reported. I chose not to rank it automatically; its real
  content would score about 65, below A and E anyway.
- **D** flagged: no completion time, so the 16-week limit cannot be checked. Lowest price, so a reviewer
  should ask the bidder for a schedule.
