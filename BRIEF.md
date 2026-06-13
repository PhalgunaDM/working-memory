# OpenLoops — Product Brief

**Build Day:** Claude Fable 5 Build Day, June 13, 2026 · Solo team · Submission due 5:00 PM PT

## 1. The problem

People who context-switch heavily — especially people with ADHD — constantly open "loops": a flight search, a half-written reply, a form, a comparison-shopping session. When attention jumps, the loop doesn't close; it silently drops. The cost is real: missed deadlines, broken promises, re-doing research, late fees. Todo apps don't help because the failure happens *before* the task is ever written down.

## 2. The product

**OpenLoops is a working-memory prosthetic for macOS.** It ambiently observes what the user is doing, infers the *intent* of an activity across multiple windows and minutes, detects when a coherent activity is **abandoned before completion**, and captures it as a structured **Open Loop** with enough state to resume instantly. It is a productivity tool. It is not an advisor, coach, or chatbot of any kind.

The user experience has three tiers:

1. **Remember.** Abandoned loops land automatically on a centralized "Shelf" (menu bar popover + read-only web view). The user never wrote anything down.
2. **Schedule.** A polite, low-frequency notification offers to schedule the loop: one click writes it to Apple Reminders / Calendar via EventKit (which syncs to iPhone via iCloud).
3. **Advance.** Where possible, the next step is pre-prepared when the user returns: an unfinished draft completed and awaiting approval; researched options assembled into a comparison; a deadline extracted and attached. OpenLoops never *sends, submits, or purchases* anything on its own — it tees work up for one-click human approval.

## 3. The core object

An **Open Loop** is defined by `schema/open_loop.schema.json`. Read it before writing any code. Key fields: `intent`, `category`, `progress_state`, `artifacts` (URLs, draft text, key facts), `next_action`, `deadline` (+ `deadline_source`), `resume_payload`, `confidence`, `evidence` (observation indices).

## 4. Architecture (constraints, not suggestions)

```
ContextSource (protocol)
 ├── ReplayDriver   — feeds fixture sessions from fixtures/*.json   [build & test FIRST]
 └── LiveDriver     — NSWorkspace app-switch events + AXUIElement text extraction,
                      ScreenCaptureKit screenshot fallback           [build LAST]
        │
        ▼
ThreadTracker — maintains a set of active activity threads across a rolling
  observation window; accumulates evidence per thread; emits a candidate when a
  thread is (a) coherent and (b) abandoned (focus moved away, dwell threshold
  passed, no completion signal observed)
        │
        ▼
IntentEngine — calls the Claude API (model: claude-fable-5) with thread history;
  returns Open Loop JSON conforming to the schema, or null. Prompt lives in a
  versioned file (Prompts/intent_engine.md) so the eval harness can iterate it.
        │
        ├──▶ Shelf (menu bar popover, SwiftUI) + local persistence (JSON/SQLite)
        ├──▶ Notifier (UNUserNotificationCenter) → EventKit on accept
        └──▶ Web Shelf — tiny read-only viewer (static site or minimal server)
             deployed to a live URL; renders the user's loops + a demo replay mode
```

**Hard rules:**
- Everything above LiveDriver must be fully buildable and testable headlessly via the ReplayDriver — `swift test` plus the eval harness must run with no GUI, no permissions, no human.
- The LiveDriver is a thin adapter. Do not let TCC-permission-dependent code leak into the core. Permission grants are the *only* planned human touchpoints.
- Screen text is sent to the Claude API. Show a clear first-run consent screen stating exactly that. Never persist raw screen text longer than the rolling window; persist only the structured Open Loop.
- Tier 3 "Advance" must produce drafts/preparations only. Any action visible to another human or vendor (send, submit, purchase) requires an explicit user click.
- No "mental health" framing anywhere in code, UI copy, or README. The framing is: task capture for heavy context-switchers.

## 5. What "done" looks like (machine-verifiable)

Done = ALL of the following, verified without a human:

1. `swift test` passes (unit tests for ThreadTracker state machine, schema validation, EventKit adapter behind a mock).
2. The eval harness (`scripts/run_eval`) replays every fixture in `fixtures/` through the full pipeline and grades per `RUBRIC.md`, and the scores meet the thresholds in RUBRIC §4.
3. The macOS app builds (`xcodebuild -scheme OpenLoops build`) with the LiveDriver compiled in.
4. The Web Shelf responds 200 at a deployed URL and renders loops from the demo dataset.
5. A verifier sub-agent, in a fresh context, has graded the final state against `RUBRIC.md` and returned PASS.

Items requiring a human (do these LAST, and announce when ready): granting Screen Recording / Accessibility / Reminders / Notifications permissions for a live end-to-end run.

## 6. Out of scope (do not build)

iOS app · agentic completion of purchases/sends · browser extension · accounts/auth on the web shelf · analytics · onboarding beyond the consent screen · settings beyond a goals text field and capture on/off toggle.

## 7. Deliverables

Public GitHub repo containing: the app, the eval harness, `Prompts/` (versioned), this brief, `RUBRIC.md`, fixtures, the session log, and a `DEMO.md` with the exact on-stage script. Plus the deployed Web Shelf URL.
