# Contribution [2]: [Improve "number of ntapes differs from restart file" error msg #3593]

**Contribution Number:** 2
**Student:** Ena Salazar  
**Issue:** [GitHub issue link](https://github.com/ESCOMP/CTSM/issues/3593)
**Status:** Phase IV Complete — PR opened and under review

---

## Why I Chose This Issue

I chose this issue because it is open, unassigned, labeled `good first issue` and `size: small`, and has no linked pull requests. The requested change is narrow: improve an unclear error message so users know they should use a branch run instead of changing history options during a restart. This matters because better error messages help users understand not only what went wrong, but what to do next. Since I have already completed one pull request, this issue seems like a manageable next step that still lets me practice working in a real source-code file.

---

## Understanding the Issue

### Problem Description

When a user changes the number of history files in the middle of a restart run with `CONTINUE_RUN=TRUE`, CTSM gives an error saying that the number of `ntapes` differs from the restart file. The current message explains that history options cannot be changed on restart, but it does not explain what the user should do instead.

### Expected Behavior

The error message should tell the user that if they need to change history-file options, they should use a branch run instead of trying to change those options during a restart run.

### Current Behavior

The current error message says:

```text
ENDRUN: ERROR in histFileMod.F90 at line 4886
ERROR: number of ntapes differs from restart file. You can NOT change history
options on restart.
```

This tells the user what went wrong, but it does not tell them the recommended next step.

### Affected Components

The issue points to `histFileMod.F90`, specifically the error message related to history-file settings and restart files. During reproduction I found this is not a single message — there are **three sibling error messages** in the same restart-read block that all share the same "you can NOT change history options on restart" phrasing without telling the user what to do (see Reproduction below).

---

## Reproduction Process

### Environment Setup

**A note on why this is a code-level reproduction, not a live model run.**

CTSM is a large Fortran land model that runs through the CIME case-management framework. It is designed to run on supported HPC machines (e.g. NCAR's Derecho) or in the CESM-Lab Docker container. My development machine is an Apple M1 (arm64) with 8 GB RAM and ~17 GB free disk, and:

- A full source build/port on macOS is out of scope for a one-line message change.
- The CESM-Lab container images are built for x86_64, so on Apple Silicon they run under emulation, and 8 GB RAM / ~17 GB free disk is below what is needed to build and run even a small single-point case plus store input data.

Because triggering this specific error requires running a case, writing a restart file, then changing history options and continuing the run, a live reproduction on this hardware is not feasible. I confirmed this constraint with the maintainer (samsrabin), who asked whether I had built and run CTSM. I therefore reproduced the issue at the **source-code level** — locating the exact failing code path and confirming the conditions that trigger it — which is sufficient to verify a message-text-only change. This is also consistent with CTSM's contribution model, where "a software engineer responsible for CTSM will do additional testing and bring the changes to CTSM master."

**What I actually set up:**

1. Forked `ESCOMP/CTSM` on GitHub → `enu-emu/CTSM`.
2. Cloned my fork locally (shallow, single-branch, no externals to keep it small — ~50 MB):
   ```bash
   git clone --depth 1 --single-branch https://github.com/enu-emu/CTSM.git
   cd CTSM
   ```
3. Added the upstream remote:
   ```bash
   git remote add upstream https://github.com/ESCOMP/CTSM.git
   ```
4. Created a working branch:
   ```bash
   git checkout -b fix-issue-3593
   ```

**Challenge & how I solved it:** the real friction here was setup feasibility, not a dependency error. I researched CTSM's setup paths (quickstart docs, CESM-Lab container, forum threads) and concluded a local run wasn't viable on my hardware. Rather than sink days into a build that was unlikely to succeed, I confirmed the constraint with the maintainer and shifted to a code-inspection reproduction.

### Steps to Reproduce

The error is deterministic and lives on a known code path. Anyone can confirm it by inspection, and the runtime scenario that triggers it is well-defined:

1. Open `src/main/histFileMod.F90` in the CTSM source.
2. Go to the `restart` subroutine's `read` branch (`else if (flag == 'read')`), inside `if_restart1: if (is_restart())`.
3. Observe the guard at line **4881**: `if (ntapes_onfile /= ntapes)`. When the number of history tapes on the restart file differs from the number configured for the current run, execution reaches the `endrun` at line **4883**.
4. **Expected:** the error should tell the user *what to do* — that changing history options isn't allowed on a continuation run, and that a branch run is the supported way to change them.
5. **Actual:** the message stops at *"You can NOT change history options on restart."* — it states the restriction but gives no next step.
6. Confirm the same pattern repeats for two sibling checks in the same block:
   - line **4911** — `history_tape_in_use` differs from restart file
   - line **4972** — number of fields differs from restart file

**Runtime scenario that triggers it (for reference):** run a case, let it write a restart file, change a history-file option (e.g. the number of history tapes) in the namelist, then continue the run with `CONTINUE_RUN=TRUE`. The `ntapes_onfile /= ntapes` guard fires and CTSM calls `endrun` with the message above.

### Reproduction Evidence

- **Branch link:** [enu-emu/CTSM — fix-issue-3593](https://github.com/enu-emu/CTSM/tree/fix-issue-3593)
- **My findings:** The issue names one message, but the same unhelpful phrasing appears in **three** places in the restart-read block (lines 4883, 4911, 4972). All three tell the user what is *not* allowed without pointing to the branch-run remedy. Fixing all three keeps the messages consistent.
- **Style note:** lines 4883 & 4911 pass the location as a separate `additional_msg=errMsg(sourcefile, __LINE__)` argument, while line 4972 concatenates `errMsg(...)` directly into `msg` with `//`. I preserved each message's existing structure rather than "fixing" the inconsistency.

---

## Solution Approach

### Analysis

The root cause is a usability gap in the error text, not a logic bug. The guards correctly detect that history options changed across a restart and correctly stop the run. The problem is purely that the message tells the user *what is forbidden* ("You can NOT change history options on restart") without telling them *what to do instead* (use a branch run). A user hitting this has no way to know the supported path forward from the message alone.

### Proposed Solution

Append an action sentence to each of the three messages so they point the user to a branch run, and clarify that the restriction applies to a *continuation run* (which is what `CONTINUE_RUN=TRUE` is). No conditions, variables, or control flow change — this is message text only.

New wording (applied to all three):

```text
... You can NOT change history options on a continuation run.
To change history options, use a branch run instead.
```

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** When history options differ from what's stored in the restart file, CTSM stops with an error. The error explains the restriction but not the remedy, leaving users stuck. It should tell them to use a branch run instead.

**Match:** The three `endrun` calls in the restart-read block of `histFileMod.F90` are the pattern to match. They already use CTSM's standard `endrun(msg=..., additional_msg=errMsg(sourcefile, __LINE__))` convention (and one variant that concatenates `errMsg` into `msg`). I followed the existing structure of each call so the change blends in with surrounding code.

**Plan:**
1. In `src/main/histFileMod.F90`, edit the `endrun` message at line 4883 (`ntapes` differs) to add: "You can NOT change history options on a continuation run. To change history options, use a branch run instead."
2. Apply the same wording to the sibling message at line 4911 (`history_tape_in_use` differs).
3. Apply the same wording to the sibling message at line 4972 (number of fields differs), keeping its inline `// errMsg(...)` structure.
4. Keep the change message-text-only; do not alter any conditions or logic.

**Implement:** [enu-emu/CTSM — fix-issue-3593](https://github.com/enu-emu/CTSM/tree/fix-issue-3593) — the three edits are committed on this branch.

**Review:** I self-reviewed against CTSM's [CONTRIBUTING.md](https://github.com/ESCOMP/CTSM/blob/master/CONTRIBUTING.md), which asks contributors to open an issue/discuss first (done — I claimed the issue and confirmed scope with the maintainer), work on a fork + branch, and expect a CTSM software engineer to do additional testing before merge. I kept the commit subject short and imperative. I did not change logic, so no coding-standard concerns beyond matching the existing message style.

**Evaluate:** Because this is a text-only change, correctness is verified by inspection (the diff touches only string literals inside `msg=`) and by CTSM's existing automated regression/CI suite, which the maintainers run. A live run to trigger the message isn't feasible on my hardware; I flagged this to the maintainer so verification can happen via CI / their side.

---

## Testing Strategy

### Unit Tests

- [ ] Not applicable in the usual sense — this is a change to error-message string literals with no new logic to unit-test. CTSM's Fortran unit-test suite (pFUnit) does not test message wording.

### Integration Tests

- [ ] CTSM's regression test system (the maintainers' testing infrastructure) builds the model and runs standardized cases. A text-only change should be bit-for-bit identical and pass unchanged. This is the verification path noted in CONTRIBUTING.md ("a software engineer responsible for CTSM will do additional testing").

### Manual Testing

- Verified by inspection that the diff changes only string content inside `endrun(msg=...)` calls and touches no conditions, variables, or control flow.
- Confirmed the edited Fortran still uses valid line-continuation (`//` + `&`) syntax matching the surrounding code.
- A full manual run to trigger the message is not feasible on my hardware (see Environment Setup); flagged to the maintainer.

---

## Implementation Notes

### Week 1 Progress (Phase I — Understand & Choose)

Selected the issue, confirmed it was open/unassigned/`good first issue`, and wrote up the problem, expected vs. current behavior, and affected components. Commented on the issue to claim it and asked a scoping question (fix only this message, or similar ones too).

### Week 2 Progress (Phase II — Reproduce & Plan)

- Forked and cloned CTSM, added upstream, created the `fix-issue-3593` branch.
- Investigated setup feasibility and concluded a local build/run isn't viable on an M1/8 GB/17 GB machine; confirmed this with the maintainer and moved to a code-level reproduction.
- Located the failing code path and discovered **three** sibling messages, not one.
- Wrote the UMPIRE solution plan and made the actual code edits on the branch (see below).

### Code Changes

- **Files modified:** `src/main/histFileMod.F90` (3 error messages, text-only).
- **Key commit:** `Improve restart history-option error messages (#3593)` on branch `fix-issue-3593`.
- **Approach decisions:** fixed all three sibling messages for consistency; changed "on restart" → "on a continuation run" for precision; preserved each `endrun` call's existing structure.

### Week 3 Progress (Phase III — Build & PR)

Completed. Made the three text-only edits on the `fix-issue-3593` branch, pushed to my fork, and opened [PR #4113](https://github.com/ESCOMP/CTSM/pull/4113) against `ESCOMP/CTSM`. The PR description explains the change, notes that I fixed all three sibling messages for consistency (offering to narrow it), and discloses that I verified by code inspection rather than a local run because CTSM can't build on my hardware.

### Week 4 Progress (Phase IV — Review & Iterate)

PR is open and in the maintainers' review queue. Since this is a message-text-only change on a large scientific model, review is expected to move on the maintainers' cadence and be validated through their regression/CI system rather than a local run on my side. No maintainer comments have come in yet. My plan for the review phase:

- Monitor the PR for reviewer comments and CI results.
- Respond quickly to any wording preferences or a request to narrow the change to just the `ntapes` message.
- Rebase onto `master` if the maintainers ask for it.
- If there's no response after ~5–7 business days, leave a single polite follow-up asking whether anything is needed to move it forward.

I'm treating the contribution as complete on my end: the fix is written, pushed, and submitted as a PR, with the reflection captured below. Any further revisions will be driven by maintainer feedback.

---

## Pull Request

**PR Link:** https://github.com/ESCOMP/CTSM/pull/4113

**PR Description (draft):**

> **Improve restart history-option error messages (#3593)**
>
> Fixes #3593. The error raised when history options change across a restart told the user what was forbidden ("You can NOT change history options on restart") but not what to do about it. This adds guidance to use a branch run instead, and clarifies that the restriction applies to a continuation run.
>
> I noticed the same phrasing appears in three sibling `endrun` calls in the restart-read block of `histFileMod.F90` (the `ntapes`, `history_tape_in_use`, and number-of-fields checks), so I updated all three for consistency — happy to narrow it to just the `ntapes` message if you'd prefer.
>
> This is a message-text-only change (no logic touched), so it should be bit-for-bit. Note: I wasn't able to build/run CTSM locally (M1 Mac, limited RAM/disk), so I verified by code inspection — happy to have this checked via your regression testing.

**Maintainer Feedback:**

- _None yet — PR is open and in the review queue; no reviewer comments have been posted. Will document any feedback and my responses here as it arrives._

**Status:** PR opened ([#4113](https://github.com/ESCOMP/CTSM/pull/4113)) — under review

---

## Learnings & Reflections

### Technical Skills Gained

- How to navigate a very large unfamiliar Fortran codebase to a specific code path using search and the issue's line-number hint.
- CTSM/CESM's setup model (CIME, supported machines, CESM-Lab container) and why scientific-model setup is fundamentally heavier than a `pip install`.
- Reading Fortran line-continuation syntax and matching an existing `endrun`/`errMsg` convention.

### Challenges Overcome

- The hard part of this issue was not the code — it was recognizing early that setting up the model locally wasn't feasible on my hardware, and handling that honestly with the maintainer instead of burning days on a build. I also caught that the issue was really three sibling messages, not one.

### What I'd Do Differently Next Time

- When scoping an issue in Phase I, check *whether the project can actually run on my machine* before committing — for a scientific model like CTSM, "can I build/run it?" is the real gate, not "is the fix small?". For my next contribution I'll prioritize projects that run cleanly on my laptop.

---

## Resources Used

- [CTSM issue #3593](https://github.com/ESCOMP/CTSM/issues/3593)
- [CTSM CONTRIBUTING.md](https://github.com/ESCOMP/CTSM/blob/master/CONTRIBUTING.md)
- [CTSM Users Guide — Quickstart](https://escomp.github.io/CTSM/users_guide/overview/quickstart.html)
- [CESM-Lab container basics (NCAR NEON tutorial)](https://ncar.github.io/ncar-neon-books/quick_start_docker.html)
- [DiscussCESM — running CTSM on a personal computer](https://bb.cgd.ucar.edu/cesm/threads/how-to-run-ctsm-offline-on-my-own-computer.9304/)
