# OpenLoops — Rubric

This file is the grading contract. The eval harness grades §1–§4 in code. The verifier
sub-agent grades §5 by inspection. The builder may not stop until §4 thresholds are met
AND the verifier returns PASS on §5.

## 1. Loop detection (per fixture)

For each fixture, compare loops produced by the pipeline against `expected.open_loops`.

- **True positive:** a produced loop whose `intent` semantically matches an expected
  loop (operationalized: contains ALL strings in that loop's `intent_must_include`,
  case-insensitive).
- **False positive:** a produced loop matching no expected loop, OR any loop produced
  for a fixture whose `expected.open_loops` is empty, OR a loop matching an entry in
  `expected.must_not_detect`.
- **False negative:** an expected loop with no matching produced loop.

## 2. Field accuracy (per true-positive loop)

| Field            | Check (code)                                                              | Weight |
|------------------|---------------------------------------------------------------------------|--------|
| category         | exact match to `category`                                                  | 1      |
| progress_state   | contains ≥ 80% of `progress_must_include` strings                          | 2      |
| artifacts.urls   | contains every URL in `artifacts_must_include_urls`                        | 2      |
| artifacts.draft  | if `draft_must_be_preserved`, produced draft contains that exact substring | 2      |
| deadline         | exact ISO-date match to `deadline` (or both null)                          | 3      |
| deadline_source  | non-empty whenever deadline is non-null                                    | 1      |
| next_action      | contains ≥ 1 of `next_action_keywords`                                     | 1      |
| schema validity  | loop validates against `schema/open_loop.schema.json`                      | gate*  |

\* Schema-invalid output scores 0 for the entire loop.

Field score per loop = weighted fraction of checks passed.

## 3. Statefulness checks

- Fixture `09`: the loop is opened AND closed within the session (email sent).
  Producing a loop here is a false positive.
- Fixture `10`: a loop is abandoned mid-session, then completed later in the same
  session. Producing a loop at end-of-session is a false positive. (A transient
  candidate that is retracted before session end is acceptable.)
- Fixture `08`: rapid app-flipping with no coherent thread. Any loop = false positive.

## 4. Pass thresholds (eval harness must print these)

- Detection: **precision ≥ 0.90** and **recall ≥ 0.80** across all fixtures pooled.
- False positives on negative fixtures (08, 09, 10): **zero**.
- Mean field score across true positives: **≥ 0.80**.
- Deadline extraction: **100%** exact on fixtures where a deadline exists (04, 06, 07).
- Eval harness runtime is deterministic in structure: same fixtures, same report format
  (JSON report written to `eval/report.json` + human-readable summary).

If thresholds are not met: inspect failures in the report, revise
`Prompts/intent_engine.md` and/or ThreadTracker logic, re-run. Log each iteration's
scores in `eval/history.md` (this is demo material — do not skip it).

## 5. Product quality (verifier sub-agent, fresh context)

Grade each 1–5; PASS requires all ≥ 4. Judge from code, copy, and screenshots/recordings.

1. **Notification restraint.** At most one notification per loop; nothing fires for
   low-confidence (< 0.75) loops; copy is ≤ 2 sentences, names the loop specifically,
   offers exactly one action. No nagging language.
2. **Shelf clarity.** A loop card answers, at a glance: what was I doing, where did I
   stop, what's the one-click next step. Resume link works.
3. **Tier-3 safety.** Prepared drafts are clearly marked as drafts awaiting approval;
   no code path sends/submits/purchases without a user click.
4. **Privacy posture.** Consent screen states screen text goes to the Claude API; raw
   observations are not persisted beyond the rolling window; a visible pause toggle
   exists.
5. **Honest framing.** No mental-health/medical claims anywhere. README and UI describe
   a productivity tool for heavy context-switchers.

## 6. Build/deploy gates (code)

- `swift test` exit 0 · `xcodebuild -scheme OpenLoops build` exit 0 ·
  deployed Web Shelf URL returns HTTP 200 and renders the demo dataset.
