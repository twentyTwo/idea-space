---
name: retry
description: Re-run a IdeaSpace idea — research and grill again for a blocked idea, or a grill-only rerun. Use only when the user explicitly asks to retry or re-grill an idea.
disable-model-invocation: true
---

# Retry an idea

Two moves, and the script enforces which one is legal from which stage:

- `is retry <n>` — a blocked idea goes back to `is:captured`, which re-runs research
  and grill from scratch.
- `is retry <n> --grill` — a ready or blocked idea is re-grilled alone, reusing the
  research already posted. Cheaper, and the right move when only the grill failed.

## Running `is`

This plugin ships one script, `scripts/is`, two directories above this skill's own
directory. Run it with bash, by absolute path: `bash "<plugin root>/scripts/is" …`.

It acts on the current directory's GitHub repo. If it says the repo has no `is:idea`
label, the user is working somewhere else: ask for their IdeaSpace repo and pass
`--repo <owner/name>` before the command. When the script refuses, relay its message —
never work around it with `gh` directly. Its refusals are the pipeline's rules.

## Steps

1. If you do not know the idea's stage, run `is status` first.

2. **Blocked:** read the failure line. "Grill failed" means the research is already
   posted — suggest `--grill`. "Research failed" needs the full retry.

3. **Ready:** re-grilling costs a model run and posts a second critique beside the
   first. Confirm the user deliberately wants that before running `--grill`.

4. Run it and report. Anything carrying `is:running` is refused: a run is in flight
   and will finish on its own.
