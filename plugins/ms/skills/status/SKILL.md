---
name: status
description: Show every open IdeaSpace idea by stage, with the research recommendation and grill kill shot for each one awaiting a verdict. Use when the user asks what ideas are in flight, ready, blocked, or waiting on them.
---

# Idea status

## Running `ms`

This plugin ships one script, `scripts/ms`, two directories above this skill's own
directory. Run it with bash, by absolute path: `bash "<plugin root>/scripts/ms" …`.

It acts on the current directory's GitHub repo. If it says the repo has no `ms:idea`
label, the user is working somewhere else: ask for their IdeaSpace repo and pass
`--repo <owner/name>` before the command. If it says the labels need migrating, say
so and point to the setup skill — do not reconstruct the status by hand with `gh`
against old label names; that hides the problem instead of reporting it.

## Steps

1. Run `ms status`.

2. Present it grouped by stage, in this order: `ms:ready` first — those are waiting on
   the user — then `ms:blocked`, then `ms:running` and `ms:captured`.

3. For each ready idea, give the research recommendation and the grill's kill shot as
   the script prints them. **Do not add your own verdict** or steer the user toward
   one unless they ask. The decision is theirs, and the grill already made the
   strongest case against.

4. For each blocked idea, say which step failed. If it was the grill, the research is
   already posted and a grill-only retry is enough.

5. Close by naming what acts on this: the verdict skill for ready ideas, the retry
   skill for blocked ones.
