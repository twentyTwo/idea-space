# IdeaSpace

IdeaSpace is a personal idea pipeline. An idea is captured as a GitHub issue from a
phone conversation. An agent researches what already exists in that space and posts
its findings as a comment. A second agent then grills the idea — challenging its
assumptions and weaknesses — in a separate comment. A human reads both and decides:
spin the idea into a real project, or kill it with a recorded reason. The point is not
automation for its own sake — it is that ideas stop dying in chat logs, and that killed
ideas stay findable so the same idea isn't researched twice.

## Issues are the only state store

There is no database, no JSON, no markdown file tracking an idea. Do not create one.

- The issue body is the idea.
- Labels are the stage.
- Comments are the work.
- Closing the issue is the verdict.

If you find yourself about to write a file to record an idea's status, that is a bug.
Put it in the issue.

## The label state machine

Every label this pipeline reads or writes is namespaced `is:` — the repo also carries
GitHub's stock labels, and the prefix is what separates machine state from a label a
human picked off the list. A new pipeline label takes the prefix too.

Every idea issue carries `is:idea` plus **exactly one** stage label:

- `is:captured` — filed, not yet worked on. Adding it starts the pipeline.
- `is:running` — the research or grill job is in flight.
- `is:ready` — research and grill comments are posted; waiting for a human verdict.
- `is:blocked` — a job failed; its comment names the step.

Stage labels are mutually exclusive: whatever adds one must remove the one it replaces
in the same operation. The workflow finds them by exact name (the `STAGES` pattern in
`research.yml`), so a new stage must be added there too.

Legal transitions, and nothing else:

- *(none)* → `is:captured` — a human or the `idea-space` plugin's capture skill, when the idea
  is filed.
- `is:captured` → `is:running` — the research job, at start, before the model is
  called. The issue stays `is:running` through the grill job that follows in the same
  run.
- `is:running` → `is:ready` — the grill job, after the critique comment is posted.
- `is:running` → `is:blocked` — either job, on failure of any step.
- `is:blocked` → `is:captured` — a human, deliberately, to re-run research and grill.
- `is:ready` or `is:blocked` → `is:running` — a human dispatching the workflow to retry
  the grill alone: `gh workflow run research.yml -f issue=<n>`.
- `is:ready` → closed with `is:approved` or `is:killed` — a human.
- `is:blocked` → closed with `is:killed` — a human, abandoning it.

Two guards matter:

- Never add `is:captured` to, or dispatch a grill for, an issue carrying `is:running`.
  A job is in flight, and the workflow will refuse the second run.
- Never re-run anything on a `is:ready` issue unless a human has deliberately chosen to.

## Closing an issue

Only a human closes an idea issue, and always with both:

1. Exactly one verdict label — `is:approved` or `is:killed`.
2. A closing comment giving the reason in one sentence.

An issue closed without a verdict label and a stated reason is a lost idea. The reason
is the entire point of keeping killed ideas around.

This one is enforced. `verdict-guard.yml` watches every close of an `is:idea` issue: a
missing or doubled verdict label reopens it with a comment saying so, and a verdict
with no human comment after it draws a comment but no reopen — that second check is a
heuristic (it reads "newest comment is the bot" as "nobody gave a reason") and so it
never fights you over a guess.

## The `idea-space` plugin

`plugins/idea-space/` is a Claude Code and Codex plugin for the human side of this state
machine: capture, status, verdict, retry, and label setup. This repo is its
marketplace for both tools (`.claude-plugin/marketplace.json` and
`.agents/plugins/marketplace.json`).

Every rule the plugin enforces lives in one script, `plugins/idea-space/scripts/is`. The skills
are thin wrappers around it, so the two tools cannot drift apart: change a rule in the
script, never in a skill. The script stores nothing — it reads and writes issues
through `gh` — so it does not break the rule that issues are the only state store.

When you change a transition in this file, change it in the script too.

## Before creating a new idea issue

Search closed, killed ideas for a near-duplicate first:

```bash
gh issue list --state closed --label is:killed --search "<key terms>"
```

If anything looks close, surface it to the human — issue number, title, and the
recorded kill reason — and ask whether to proceed. Do not silently create the
duplicate, and do not silently refuse: the human decides. The same courtesy applies to
open ideas already in flight.

## Research output

Research comments have to be checkable. Every factual claim carries a URL and the date
it was fetched; a claim that cannot be linked is dropped, not softened into a hedge.
Never estimate revenue, user counts, or market size — those are not observable and any
number produced for them is invented.

The research prompt lives in `.github/prompts/research.md`. Edit it there; the workflow
reads it, so prompt changes never require touching YAML.

## Grill output

The grill comment argues against the idea using the research findings as ammunition —
it is not a second pass at research. The grill runs GLM-5.3 through Claude Code against
z.ai's Anthropic-compatible endpoint, billed to the GLM Coding Plan. It has no web
access, so every fact it cites must come from the research comment with that row's URL;
anything else is argument and must read as argument.

The grill prompt lives in `.github/prompts/grill.md`. Edit it there; the workflow reads
it, so prompt changes never require touching YAML.
