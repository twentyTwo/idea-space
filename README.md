# IdeaSpace

A personal idea pipeline. You capture an idea as a GitHub issue — often straight from
a phone conversation. A research agent finds out what already exists in that space and
posts its findings as a comment. A grill agent then argues against the idea, using
that research as ammunition, in a second comment. You read both and decide: turn it
into a real project, or kill it with a recorded reason.

There is no database and no tracking file. **The issue is the idea, its labels are the
stage, its comments are the work, and closing it is the verdict.** Everything in this
guide reads or writes those four things and nothing else.

This guide covers installing the `ms` plugin and using the pipeline day to day. The
exact rules the pipeline enforces — the full label state machine, what research and
the grill are and aren't allowed to claim — live in [CLAUDE.md](CLAUDE.md). This guide
won't repeat them; it'll point to them.

## Quick start

```
# Claude Code
/plugin marketplace add twentyTwo/idea-space
/plugin install ms@idea-space

# Codex
codex plugin marketplace add twentyTwo/idea-space
# then install "ms" from /plugins and start a new session
```

Inside your IdeaSpace repo, first thing, either tool:

```
/ms:setup          (Claude Code)
$ms:setup          (Codex)
```

That creates the labels the pipeline runs on. Then capture an idea and you're running.

## How it works

```
   you file it                  automatic                     you decide
  ┌────────────┐   ┌──────────────────────────────┐   ┌──────────────────────┐
  │  ms:idea   │──▶│  ms:captured ──▶ ms:running   │──▶│      ms:ready         │
  │ + captured │   │  (research, then the grill)   │   │  (both comments in)  │
  └────────────┘   └──────────────────┬────────────┘   └───────────┬───────────┘
                                       │ a step fails                │ ms verdict
                                       ▼                              ▼
                                  ms:blocked              closed, ms:approved
                                                              or ms:killed
```

Filing the idea and giving the verdict are the only two things a human has to do. The
`ms` plugin exists to make those two things — plus checking status and recovering from
a failure — commands instead of hand-edited labels.

## Requirements

- An authenticated [`gh`](https://cli.github.com) — run `gh auth status` to check.
- `bash` on your `PATH`. On Windows this means Git for Windows (`git-bash`), which the
  Git for Windows installer sets up for you.
- Claude Code or Codex, for the plugin. Not strictly required — the script underneath
  it works with no AI tool at all; see [Working without the plugin](#working-without-the-plugin).
- If you're standing up a *new* IdeaSpace repo rather than using an existing one: the
  workflows in `.github/workflows/`, the prompts in `.github/prompts/`, and two
  repository secrets — `CLAUDE_CODE_OAUTH_TOKEN` for research, `GLM_API_KEY` for the
  grill.

## Installing the plugin

**Claude Code**

```
/plugin marketplace add twentyTwo/idea-space
/plugin install ms@idea-space
```

**Codex**

```
codex plugin marketplace add twentyTwo/idea-space
```

then open `/plugins`, install `ms`, and start a new session — Codex only picks up a
plugin's skills on the next session after install.

Both read the same five skills and run the same script
([plugins/ms/scripts/ms](plugins/ms/scripts/ms)), so the two tools behave identically.
See [plugins/ms/README.md](plugins/ms/README.md) for the plugin's own reference.

## First use in a repo

Run setup once per repo, from either tool:

```
/ms:setup          (Claude Code)
$ms:setup          (Codex)
```

It creates the seven labels the pipeline needs: `ms:idea`, the four stages
(`ms:captured` `ms:running` `ms:ready` `ms:blocked`), and the two verdicts
(`ms:approved` `ms:killed`). Safe to run more than once — a label already in place is
left alone, and if this repo predates the `ms:` prefix, an old bare label (`captured`)
is *renamed* rather than recreated, so nothing already filed under it loses its stage.

It refuses to touch labels while a run is in flight, or if the default branch's
workflow hasn't been updated to look for the new names yet — either would strand an
issue mid-run. If it refuses, it says which of the two is blocking it.

## Day to day

### Capture an idea

```
/ms:capture         (Claude Code)
$ms:capture         (Codex)
```

or just say it in conversation — "save this idea", "I want to capture something" —
capture is one of the two skills that can trigger without being asked by name.

Tell it the idea in your own words. It checks killed and in-flight issues for anything
that looks like the same idea first, and surfaces what it finds rather than filing a
duplicate silently — you decide whether to proceed. Once filed, research and the grill
start on their own; nothing else to do.

### Check what's waiting on you

```
/ms:status          (Claude Code)
$ms:status          (Codex)
```

Lists every open idea by stage. For anything `ms:ready`, you get the research
recommendation and the grill's kill shot right there — usually enough to decide
without opening the issue.

### Give a verdict

```
/ms:verdict          (Claude Code)
$ms:verdict          (Codex)
```

Explicit only — it won't trigger from ordinary conversation, since a verdict is a
decision you make on purpose. Give it the issue number, `approved` or `killed`, and
your reason. **The reason has to be yours.** The skill won't invent one from the
research or the grill, however obvious the call looks — a killed idea's reason is the
entire point of keeping it around, so the next time this idea comes up, `ms:capture`
can tell you why it already died.

Behind the scenes this is three steps done as one: set the verdict label, post the
reason as a comment, then close. That order matters — a repo-side guard checks every
close for exactly one verdict label and a human comment, and reopens anything missing
either.

### When something breaks

```
/ms:retry            (Claude Code)
$ms:retry            (Codex)
```

Explicit only. An issue lands on `ms:blocked` when a step fails, with a comment naming
which one. Two moves, and it's picked for you based on what failed:

- Research failed → full retry, back to `ms:captured`, research and grill both run again.
- Only the grill failed → grill-only retry, reusing the research that's already posted.

It won't touch an issue with a run in flight — that would just be refused by the
workflow anyway — and it won't quietly rerun research on something already `ms:ready`.

## Command reference

| Claude Code | Codex | Can trigger from conversation? | Does |
|---|---|---|---|
| `/ms:capture` | `$ms:capture` | Yes | Dedupe-check, then file an idea |
| `/ms:status` | `$ms:status` | Yes | List open ideas, grouped by stage |
| `/ms:verdict` | `$ms:verdict` | No | Label, comment the reason, close |
| `/ms:retry` | `$ms:retry` | No | Re-run a blocked idea, or re-grill one |
| `/ms:setup` | `$ms:setup` | No | Create or migrate the `ms:` labels |

## Working without the plugin

Every command is a thin wrapper around one standalone script — no AI tool required.
Clone this repo, then from inside it (or with `--repo owner/name`, from anywhere):

```bash
bash plugins/ms/scripts/ms status
bash plugins/ms/scripts/ms dupes usage tracker
bash plugins/ms/scripts/ms capture --title "Idea title" <<'BODY'
The idea, in as much detail as you have.
BODY
bash plugins/ms/scripts/ms verdict 5 killed "Reason it's dead, in one sentence."
bash plugins/ms/scripts/ms retry 12 --grill
bash plugins/ms/scripts/ms setup
```

On a shell without heredocs (PowerShell), use `--body-file <path>` in place of piping
the body on stdin. Set `IDEASPACE_REPO=owner/name` once, or pass `--repo owner/name`
before the command, to run it from outside the ideas repo.

`ms help` prints all of this from the script itself.

### Dropping to raw `gh`

For anything the script and the plugin don't cover — inspecting a specific issue,
re-reading the full research comment, dispatching a grill-only retry by hand — this is
plain `gh`, working against the labels documented in [CLAUDE.md](CLAUDE.md):

```bash
gh issue view <n> --comments
gh issue list --state open --label ms:ready
gh workflow run research.yml -f issue=<n>     # re-grill alone
```

The one rule that matters here: never set a stage label yourself unless you mean to
force a transition CLAUDE.md's state machine allows. The workflow finds stages by
exact name, and two stage labels on one issue is a state the pipeline doesn't expect.

## Troubleshooting

**"`<repo>` has no ms:idea label."** Either you're in the wrong repo — pass
`--repo owner/name` or set `IDEASPACE_REPO` — or `ms setup` hasn't been run here yet.

**An idea has been `ms:running` for a long time.** Check the workflow run itself:
`gh run list --workflow=research.yml`. If it's actually finished (crashed, or GitHub
silently dropped it), the issue is stuck holding a label no step will ever clear —
fix it by hand: `gh issue edit <n> --remove-label ms:running --add-label ms:blocked`,
then `ms retry <n>`.

**An idea reopened itself right after you closed it.** That's the verdict guard: it
reopens any close missing a verdict label, or carrying both. Read its comment for
which, fix it, and close again.

**`ms setup` refuses to rename anything.** It's protecting you from stranding a run
mid-flight, or from renaming out from under a workflow that's still matching the old
label names. Its error names which; the fix is either to wait for the run in flight,
or to merge the `ms:`-aware workflow to the default branch first.

## Repo layout

```
CLAUDE.md                      the rules — read this for the exact state machine
.github/workflows/
  research.yml                 claims an idea, runs research, then the grill
  verdict-guard.yml            reopens a close with no verdict label or reason
.github/prompts/
  research.md                  what the research agent is told to do
  grill.md                     what the grill agent is told to do
plugins/ms/                    the plugin — skills, and the one script behind them
  scripts/ms                   the actual logic; everything else calls this
  skills/                      capture, status, verdict, retry, setup
.claude-plugin/marketplace.json   this repo as a Claude Code marketplace
.agents/plugins/marketplace.json this repo as a Codex marketplace
```
