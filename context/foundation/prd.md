---
project: "GymBuddy"
version: 1
status: draft
created: 2026-10-25
context_type: greenfield
product_type: web-app
target_scale:
  users: small
  qps: null
  data_volume: null
timeline_budget:
  mvp_weeks: 3
  hard_deadline: 2026-12-06
  after_hours_only: true
---

# GymBuddy — Product Requirements

## Vision & Problem Statement

A lifter training on a plan from a personal trainer receives only exercises and rep counts — no loads. Right before each exercise at the gym, they try to recall what weight they used last week and scroll through the WhatsApp conversation with the trainer or their Excel files to find it. This costs time and breaks training rhythm between sets; when the number can't be found they guess and pick a wrong load (too light — stagnation, too heavy — missed reps); and because the data is scattered they can't see whether they are progressing.

Insight: not yet established. The user has not tried existing workout trackers, so it is open whether they already solve this pain (see Open Questions).

# TODO: insight — see Open Questions

## User & Persona

**Primary persona — the lifter (the user themself).** Trains strength regularly following a plan from a trainer that specifies exercises and reps but not loads. Reaches for the product at the gym, just before starting an exercise, to know what weight to use; and occasionally to check whether they are progressing.

### Secondary persona

**The trainer.** Provides the plan (exercises + reps). Does not use the product in the MVP; may be considered later.

## Success Criteria

### Primary

- For 2 weeks, every workout is done with the app, without scrolling WhatsApp or Excel to find loads.
- The proposed load is accepted without change in the majority of sets (exact threshold not decided).
- Loads in the main exercises increase over several weeks.

# TODO: threshold for "majority" of proposed loads accepted — see Open Questions

### Secondary

- The trainer can see the user's progress (outside the app, e.g. a screenshot).

### Guardrails

- Logging a set at the gym takes seconds and does not interrupt training.
- No completed set is lost (e.g. with poor signal at the gym).
- A user's data is visible only to that user.
- The proposed load never jumps by an unrealistic amount between sessions.

## User Stories

MVP flow (as locked during shaping):

0. (Before first session) The user enters the trainer's plan manually: workouts → exercises with sets and reps.
1. User opens the app and picks a workout from the list (no "today's workout" schedule in MVP).
2. User sees the list of exercises with sets and reps.
3. User taps "Start" and gets the current exercise with a proposed load for the given set. If the exercise has no history yet, there is no proposal and the user enters the actual load.
4. After each set the user confirms it (or corrects the actual load / reps).
5. After each exercise the user may rate how hard it was — the rating feeds the decision whether to increase load next time; skipping keeps the load unchanged.
6. (Nice-to-have) User sees a workout summary.
7. (Separate) Statistics screen from the main view: list of exercises with max results; workout history.

### US-01: Workout with proposed load

- **Given** a signed-in user with a plan in which the exercise already has history from a previous session
- **When** they pick a workout from the list, start it and reach that exercise
- **Then** they see a proposed load for each set; after each set they confirm it (or correct the load/reps), and after the exercise they may rate its difficulty

# TODO: user stories for the remaining MVP flow steps (plan entry, first session without history, history & statistics) — see Open Questions

## Functional Requirements

### Account

- FR-001: User can sign up and sign in. Priority: must-have
  > Socrates: Counter-argument considered: "signing in at the gym slows down the start of a workout." Resolution: kept; the session is remembered for a long time so sign-in is rare.

### Plan

- FR-002: User can manually create, edit and delete workouts in their plan (exercises, sets, reps). Priority: must-have
  > Socrates: Counter-argument considered: "the plan changes every few weeks — editing vs. a new plan could lose load history." Resolution: kept; load history belongs to the exercise, not the plan — editing or deleting a workout never erases it, and the same exercise in a new plan reuses prior results.

### Workout session

- FR-003: User can pick a workout from the list of workouts in their plan. Priority: must-have
  > Socrates: Counter-argument considered: "a list is enough; a 'today's workout' view needs a schedule." Resolution: modified; no "today's workout" in MVP — the user always picks from the list.
- FR-004: User can start a workout and get a proposed load for each set. Priority: must-have
  > Socrates: No counter-argument; it stands as written.
- FR-005: User can confirm a set as done. Priority: must-have
  > Socrates: No counter-argument; it stands as written.
- FR-006: User can record the actual load and reps for a set when they differ from the proposal, or when there is no proposal because the exercise has no history yet. Priority: must-have
  > Socrates: Former separate FR "enter the load manually when the exercise has no history" was challenged as "just this FR with an empty proposal." Resolution: merged into this FR. This FR itself: no counter-argument; it stands as written.
- FR-007: User can optionally rate how hard each exercise was. Priority: must-have
  > Socrates: Counter-argument considered: "a whole-workout rating says nothing about a specific exercise." Resolution: modified; rating is per exercise and optional — a skipped rating keeps the load unchanged.
- FR-008: User can see a workout summary (e.g. total weight lifted). Priority: nice-to-have
  > Socrates: Counter-argument considered: "duplicates the statistics screen." Resolution: demoted to nice-to-have; statistics cover the "am I progressing" need.

### History & statistics

- FR-009: User can browse the history of completed workouts. Priority: must-have
  > Socrates: No counter-argument; it stands as written.
- FR-010: User can see a list of exercises with their max results. Priority: must-have
  > Socrates: Counter-argument considered: "'max result' is undefined / records don't show a trend." Resolution: kept; a list of max results is enough for MVP, can be extended later.

## Non-Functional Requirements

- Confirming a set shows acknowledgement to the user in < 1 s.
- A set confirmed without network connectivity is not lost and is saved once connectivity returns.
- The product is usable one-handed on a phone, on current mainstream mobile browsers.
- A user's data is never visible to another account, and the user can delete all of their data completely.

## Business Logic

After each exercise, the proposed load for the next session equals the load actually used plus the change the user selects with a 7-step difficulty rating (bad −10, not okay −5, not yet −2.5, okay 0, maybe +2.5, easy +5, more +10 kg); if not all planned reps were completed the load cannot increase (it can only stay or decrease), and a skipped rating keeps it unchanged.

Inputs (user-facing): the load and reps actually performed in each set of the exercise (confirmed or corrected during the session), the planned reps from the trainer's plan, and the optional difficulty rating given after the exercise. Load history belongs to the exercise, so it carries over when the plan changes.

Output: a proposed load for that exercise, shown to the user at the start of each set the next time the exercise appears in a workout. When the exercise has no history there is no proposal and the user enters the load.

How the user meets it: during the workout the user sees the proposal, does the set, confirms or corrects it, and after the exercise picks a rating — that choice is how the user decides how much to add or remove. The step sizes are the same in kg for every exercise; the jump between sessions is never larger than what the user explicitly chose via the rating.

## Access Control

Login required. Open sign-up: anyone can create an account. Flat model — no roles; every user sees and manages only their own data. The trainer has no access in the MVP. The session is remembered for a long time so the user rarely has to sign in at the gym.

Not yet decided: specific sign-in method (email + password / OAuth / passwordless).

# TODO: sign-in method — see Open Questions

## Non-Goals

- No import from Excel or WhatsApp — plan and history are entered manually; imports are a second stage.
- No trainer access or trainer features — no roles, sharing, or plan entry by the trainer.
- No schedule / "today's workout" and no trend charts — only the workout list and the max-results list.
- No automatic step sizing (AI, percentage-based, or per-exercise) — the user chooses the step via the rating; fixed kg steps.
- FR-008 (workout summary) is nice-to-have and outside the binding MVP scope.

## Open Questions

1. **Do existing workout trackers (e.g. Strong, Hevy) already solve this pain?** The user has not tried them — the product's insight is unestablished. — Owner: user.
2. **Which sign-in method (email + password / OAuth / passwordless)?** — Owner: user.
3. **Which load counts as "actually used" when sets in one exercise used different loads (last set, heaviest, most frequent)? What happens when a decrease would go below zero / the lightest available load?** (The load-progression rule itself was resolved during shaping — see Business Logic.) — Owner: user.
4. **Fixed kg steps for every exercise vs. the guardrail "no unrealistic jump":** on light exercises (e.g. 8 kg dumbbells) +10 kg more than doubles the load. Currently interpreted as "the jump never exceeds what the user chose" — confirm or introduce per-exercise steps later. — Owner: user.
5. **FR-004 says "proposed load for each set", but the rule produces one load per exercise — is the same load proposed for every set?** — Owner: user.
6. **How is progress measured?** (from seed) — partially: MVP shows max results per exercise; definition of "max result" not decided. — Owner: user.
7. **What are the expected Excel format(s) for plan and history?** (Excel import itself was resolved: not in MVP, deferred to a second stage.) — Owner: user.
8. **What threshold counts as "majority" of proposed loads accepted without change?** — Owner: user.
9. **What scale does the per-exercise difficulty rating use?** — Owner: user. Note: the Business Logic section already specifies a 7-step scale (bad −10 … more +10 kg); confirm and close this entry.
10. **User stories for the remaining MVP flow steps** — only US-01 has a Given/When/Then block; plan entry (flow step 0), the first session of an exercise without history, and history & statistics (FR-009, FR-010) have none. — Owner: user.
11. **Data deletion has no FR** — the NFR "the user can delete all of their data completely" implies a capability not listed among FR-001…FR-010. Should it become an FR, and with what priority? — Owner: user.
12. **`target_scale.qps` and `target_scale.data_volume` are unset (null)** — confirm a ballpark or leave as not relevant for a single-digit user base. — Owner: user.
