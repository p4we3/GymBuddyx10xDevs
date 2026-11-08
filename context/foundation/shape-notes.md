---
project: "GymBuddy"
context_type: greenfield
created: 2026-10-22
updated: 2026-10-23
product_type: web-app
target_scale:
  users: small
  qps: null
  data_volume: null
timeline_budget:
  mvp_weeks: 3
  hard_deadline: 2026-12-06
  after_hours_only: true
checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  gray_areas_resolved:
    - topic: "pain cost"
      decision: "lost time and training rhythm; guessing the wrong load; no visibility of progress"
    - topic: "insight vs existing apps"
      decision: "unknown — user has not tried existing trackers (Open Question)"
    - topic: "primary persona scope"
      decision: "the user themself; trainer is a secondary persona, out of MVP"
    - topic: "access model"
      decision: "login; open sign-up for others; flat model, no roles; each user sees only own data"
    - topic: "plan entry"
      decision: "manual entry in MVP; Excel import deferred"
    - topic: "starting load with no history"
      decision: "entered manually during the first session of that exercise"
    - topic: "MVP scope"
      decision: "scoped down — both Excel imports (plan, history) deferred to stage 2"
    - topic: "today's workout"
      decision: "no schedule in MVP — user always picks a workout from the list"
    - topic: "load history ownership"
      decision: "load history belongs to the exercise, not to the plan; plan edits never erase it"
    - topic: "difficulty rating"
      decision: "per exercise, optional; skipped rating keeps the load unchanged"
    - topic: "rating scale"
      decision: "7 steps: bad −10, not okay −5, not yet −2.5, okay 0, maybe +2.5, easy +5, more +10 kg"
    - topic: "proposal base"
      decision: "load actually used + rating delta"
    - topic: "missed planned reps"
      decision: "increase blocked; negative rating can still decrease"
    - topic: "step size"
      decision: "same kg steps for every exercise"
  frs_drafted: 10
  quality_check_status: accepted
---

# Shape notes

## Seed idea (verbatim)

Osobiste narzędzie do śledzenia treningów siłowych. Zapisuję treningi (ćwiczenie, serie, powtórzenia, obciążenie), widzę postęp w czasie, a aplikacja proponuje obciążenie na następną sesję na podstawie mojej historii danego ćwiczenia. Jeden użytkownik (ja), z logowaniem. Start od zera. Historia treningów jest w plikach Excel i w rozmowie z trenerem na WhatsAppie.

Otwarte pytania:

- Według jakiej reguły dobierać obciążenie (zakresy powtórzeń, krok zwiększania ciężaru, co robić, gdy nie zrobię zaplanowanej liczby powtórzeń)?
- Jak mierzyć postęp?
- Czy import z Excela wchodzi do MVP?

## Vision & Problem Statement

A lifter training on a plan from a personal trainer receives only exercises and rep counts — no loads. Right before each exercise at the gym, they try to recall what weight they used last week and scroll through the WhatsApp conversation with the trainer or their Excel files to find it. This costs time and breaks training rhythm between sets; when the number can't be found they guess and pick a wrong load (too light — stagnation, too heavy — missed reps); and because the data is scattered they can't see whether they are progressing.

Insight: not yet established. The user has not tried existing workout trackers, so it is open whether they already solve this pain (see Open Questions).

## User & Persona

**Primary persona — the lifter (the user themself).** Trains strength regularly following a plan from a trainer that specifies exercises and reps but not loads. Reaches for the product at the gym, just before starting an exercise, to know what weight to use; and occasionally to check whether they are progressing.

### Secondary persona

**The trainer.** Provides the plan (exercises + reps). Does not use the product in the MVP; may be considered later.

## Access Control

Login required. Open sign-up: anyone can create an account. Flat model — no roles; every user sees and manages only their own data. The trainer has no access in the MVP. The session is remembered for a long time so the user rarely has to sign in at the gym.

Not yet decided: specific sign-in method (email + password / OAuth / passwordless).

## MVP flow (locked in phase 3, adjusted in phase 4)

0. (Before first session) The user enters the trainer's plan manually: workouts → exercises with sets and reps.
1. User opens the app and picks a workout from the list (no "today's workout" schedule in MVP).
2. User sees the list of exercises with sets and reps.
3. User taps "Start" and gets the current exercise with a proposed load for the given set. If the exercise has no history yet, there is no proposal and the user enters the actual load.
4. After each set the user confirms it (or corrects the actual load / reps).
5. After each exercise the user may rate how hard it was — the rating feeds the decision whether to increase load next time; skipping keeps the load unchanged.
6. (Nice-to-have) User sees a workout summary.
7. (Separate) Statistics screen from the main view: list of exercises with max results; workout history.

Scope decision: scoped down. Excel import of the plan and of load history is out of the MVP and deferred to a second stage (user wants both eventually; Excel files would follow a defined format).

## Success Criteria

### Primary

- For 2 weeks, every workout is done with the app, without scrolling WhatsApp or Excel to find loads.
- The proposed load is accepted without change in the majority of sets (exact threshold not decided).
- Loads in the main exercises increase over several weeks.

### Secondary

- The trainer can see the user's progress (outside the app, e.g. a screenshot).

### Guardrails

- Logging a set at the gym takes seconds and does not interrupt training.
- No completed set is lost (e.g. with poor signal at the gym).
- A user's data is visible only to that user.
- The proposed load never jumps by an unrealistic amount between sessions.

## Timeline budget

mvp_weeks: 3 (user's own estimate for the flow; scope was then reduced by deferring both Excel imports).
hard_deadline: 2026-12-06. after_hours_only: true.

## Product framing

- Product type: web app.
- Target scale: just the user or a handful of people.
- Scale challenge (Socrates, "how would the rule change at 100× scale?"): user's answer — the MVP rule is sufficient; it will be revisited after the first real workouts.

## Non-Goals

- No import from Excel or WhatsApp — plan and history are entered manually; imports are a second stage.
- No trainer access or trainer features — no roles, sharing, or plan entry by the trainer.
- No schedule / "today's workout" and no trend charts — only the workout list and the max-results list.
- No automatic step sizing (AI, percentage-based, or per-exercise) — the user chooses the step via the rating; fixed kg steps.

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

## User Stories

### US-01: Workout with proposed load

- **Given** a signed-in user with a plan in which the exercise already has history from a previous session
- **When** they pick a workout from the list, start it and reach that exercise
- **Then** they see a proposed load for each set; after each set they confirm it (or correct the load/reps), and after the exercise they may rate its difficulty

## Business Logic

After each exercise, the proposed load for the next session equals the load actually used plus the change the user selects with a 7-step difficulty rating (bad −10, not okay −5, not yet −2.5, okay 0, maybe +2.5, easy +5, more +10 kg); if not all planned reps were completed the load cannot increase (it can only stay or decrease), and a skipped rating keeps it unchanged.

Inputs (user-facing): the load and reps actually performed in each set of the exercise (confirmed or corrected during the session), the planned reps from the trainer's plan, and the optional difficulty rating given after the exercise. Load history belongs to the exercise, so it carries over when the plan changes.

Output: a proposed load for that exercise, shown to the user at the start of each set the next time the exercise appears in a workout. When the exercise has no history there is no proposal and the user enters the load.

How the user meets it: during the workout the user sees the proposal, does the set, confirms or corrects it, and after the exercise picks a rating — that choice is how the user decides how much to add or remove. The step sizes are the same in kg for every exercise; the jump between sessions is never larger than what the user explicitly chose via the rating.

## Non-Functional Requirements

- Confirming a set shows acknowledgement to the user in < 1 s.
- A set confirmed without network connectivity is not lost and is saved once connectivity returns.
- The product is usable one-handed on a phone, on current mainstream mobile browsers.
- A user's data is never visible to another account, and the user can delete all of their data completely.

## Quality cross-check

Run 2026-10-23. Access Control, Business Logic (one-sentence rule), project artifacts, timeline-cost (mvp_weeks 3) and Non-Goals all present; preserved behavior n/a (greenfield). Status: accepted. Remaining uncertainty is tracked in Open Questions below.

## Open Questions (running)

1. Do existing workout trackers (e.g. Strong, Hevy) already solve this pain? The user has not tried them — the product's insight is unestablished.
2. Which sign-in method (email + password / OAuth / passwordless)?
3. ~~Load-progression rule~~ Resolved in phase 5 (see Business Logic). Still open: which load counts as "actually used" when sets in one exercise used different loads (last set, heaviest, most frequent)? What happens when a decrease would go below zero / the lightest available load?
4. Fixed kg steps for every exercise vs. the guardrail "no unrealistic jump": on light exercises (e.g. 8 kg dumbbells) +10 kg more than doubles the load. Currently interpreted as "the jump never exceeds what the user chose" — confirm or introduce per-exercise steps later.
5. FR-004 says "proposed load for each set", but the rule produces one load per exercise — is the same load proposed for every set?
6. How is progress measured? (from seed) — partially: MVP shows max results per exercise; definition of "max result" not decided.
7. ~~Is Excel import in the MVP?~~ Resolved in phase 3: no — deferred to a second stage. Open: the expected Excel format(s) for plan and history.
8. What threshold counts as "majority" of proposed loads accepted without change?
9. What scale does the per-exercise difficulty rating use?
