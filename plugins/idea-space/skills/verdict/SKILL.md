---
name: verdict
description: Close a IdeaSpace idea with a verdict — approved or killed — and a one-sentence reason. Use only when the user explicitly gives a verdict on an idea.
disable-model-invocation: true
---

# Record a verdict

Closing an idea takes three things together: exactly one verdict label, a comment
giving the reason, and the close itself. `is verdict` does all three, in the order the
repo's verdict guard checks them. Always use it; never close an idea issue by hand.

## Running `is`

This plugin ships one script, `scripts/is`, two directories above this skill's own
directory. Run it with bash, by absolute path: `bash "<plugin root>/scripts/is" …`.

It acts on the current directory's GitHub repo. If it says the repo has no `is:idea`
label, the user is working somewhere else: ask for their IdeaSpace repo and pass
`--repo <owner/name>` before the command. When the script refuses, relay its message —
never work around it with `gh` directly. Its refusals are the pipeline's rules.

## Steps

1. You need three things: the issue number, the verdict (`approved` or `killed`), and
   the reason.

2. **The reason is the user's, never yours.** Use what they said — tidied into one
   sentence if it needs it, substance unchanged. If they gave no reason, ask for one.
   Do not infer it from the research or the grill, however obvious it looks. A killed
   idea's reason is what stops it being researched twice, and only the user knows why
   they decided.

3. Run:

       bash "<plugin root>/scripts/is" verdict <n> <approved|killed> "<reason>"

4. Report the result. The script refuses an idea that is still running, an approval of
   a blocked idea that was never researched, and anything already closed; relay which.
