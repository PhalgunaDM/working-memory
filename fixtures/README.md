# Fixture format

Each fixture is one recorded (synthetic) user session plus the expected grading targets.

```jsonc
{
  "fixture_id": "01_flight_booking",
  "description": "What this fixture tests",
  "session": {
    "user_goals": ["optional strings the user typed into Settings"],
    "observations": [
      {
        "i": 0,                          // index, referenced by evidence arrays
        "t": "2026-06-13T10:02:00Z",     // timestamp
        "event": "app_focus",            // app_focus | window_focus | text_snapshot | input_burst | idle
        "app": "Safari",
        "window_title": "Google Flights",
        "url": "https://...",            // when known (browser)
        "screen_text": "condensed AX/OCR text of the focused window"
      }
    ]
  },
  "expected": {
    "open_loops": [
      {
        "intent_gold": "reference answer (for the verifier; not string-matched)",
        "intent_must_include": ["SFO", "LAX"],
        "category": "purchase_or_booking",
        "progress_must_include": ["Southwest", "$178"],
        "artifacts_must_include_urls": ["https://..."],
        "draft_must_be_preserved": null,         // or exact substring
        "deadline": null,                        // or "2026-08-03"
        "next_action_keywords": ["checkout", "book"]
      }
    ],
    "must_not_detect": ["descriptions of activities that must NOT become loops"]
  }
}
```

Notes for the eval harness:
- Sessions end when observations end; grade the pipeline's final state.
- `screen_text` is deliberately condensed but realistic; the live AX driver will produce
  noisier text. Do not overfit prompts to fixture phrasing — the verifier checks for this.
- Timestamps matter: dwell time and gaps are signal for the ThreadTracker.
