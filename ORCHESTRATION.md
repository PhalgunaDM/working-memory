# ORCHESTRATION.md — how this build runs

This file is both my plan and a judging artifact (Orchestration, 15%). Goal: another
team could rerun this setup tomorrow on a new problem.

## Kickoff prompt (paste into Claude Code at ~11:15 AM)

> Read BRIEF.md, RUBRIC.md, schema/open_loop.schema.json, and fixtures/README.md in
> full before writing any code. Ask me your questions now, in one batch — after that I
> intend to intervene only when you request permissions or new information.
>
> Then build OpenLoops per the brief. Order of work: (1) Swift package with core types
> + schema validation, (2) ReplayDriver + ThreadTracker + IntentEngine, (3) the eval
> harness `scripts/run_eval` that replays all fixtures and grades per RUBRIC.md §1–§4,
> writing eval/report.json and appending each run's scores to eval/history.md,
> (4) hillclimb the IntentEngine prompt (Prompts/intent_engine.md) and ThreadTracker
> until RUBRIC §4 thresholds pass, (5) Shelf UI + Notifier + EventKit behind a mock,
> (6) Web Shelf + deploy, (7) LiveDriver last, then tell me it's time to grant
> permissions. Commit at every milestone with descriptive messages.

## /goal

> All RUBRIC.md §4 thresholds met by `scripts/run_eval` AND `swift test` green AND
> `xcodebuild -scheme OpenLoops build` green AND Web Shelf URL returns 200 AND a
> verifier sub-agent run per below returns PASS on RUBRIC §5.

## Verifier sub-agent (fresh context, after builder claims done — and at milestones 4 and 6)

> You are a skeptical reviewer. Read RUBRIC.md only, then the repo. Grade §5 items 1–5
> (1–5 each) with evidence quotes/paths. Also check for fixture overfitting: do
> Prompts/intent_engine.md or ThreadTracker contain strings copied from fixtures
> (airline names, dollar amounts, fixture URLs)? If yes, FAIL with locations. Output:
> PASS/FAIL + per-item scores + required fixes.

## Workflow loop (describe to Claude Code in these words if coordination outgrows one session)

> Builder works toward /goal. Every milestone: run eval, append scores to
> eval/history.md, commit. After milestones 4 and 6: spawn verifier (above); builder
> must address FAIL items before proceeding. Builder may not declare done; only a
> PASS verdict from a final fresh verifier closes the goal.

## Intervention policy (Autonomy, 15% — judged from the session log)

I intervene ONLY to: (a) answer the kickoff question batch, (b) grant macOS permissions
when the LiveDriver is ready, (c) provide deploy credentials/URL target, (d) supply
genuinely new information if asked. I do NOT debug, point out failures, or steer
mid-task — failures must be caught by tests, the eval harness, or the verifier.

## Timeline (submission 5:00 PM sharp)

- 10:30–11:15 — repo public on GitHub, this kit committed, credits wired, kickoff.
- 11:15–3:00 — autonomous run. Me, in parallel: DEMO.md script, social post, 3-min story.
- ~3:00 — permissions granted; live end-to-end on my Mac; record 1-min video during it.
- 4:00–4:45 — verify deploy URL, repo public, session log exported into repo, submit.
- 4:45–5:00 — buffer. Nothing is scheduled here on purpose.

## Demo beats (3 minutes, if finalist)

1. (0:00) Team slide → "Todo apps fail people who context-switch; the task is lost
   before it's written down."
2. (0:20) Show BRIEF + fixtures + eval/history.md — point at the run where precision
   jumped after the model rewrote its own prompt off a failing fixture.
3. (1:05) LIVE: abandon a flight booking on stage → Shelf card knows route, dates,
   price → accept the nudge → reminder appears on iPhone via iCloud.
4. (2:20) Open the unsent-reply card → completed draft waiting for approval (Tier 3).
5. (2:45) "Session log: N hours autonomous, two human interventions, both permission
   grants." Close on the Web Shelf URL.
