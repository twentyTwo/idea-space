# idea-space — IdeaSpace for Claude Code and Codex

The human half of the IdeaSpace idea pipeline, from your coding agent. The GitHub
Actions workflows research and grill every idea on their own; this plugin covers the
parts that are yours: filing an idea, seeing what is waiting on you, and closing an
idea with a recorded verdict.

One plugin, both tools. The skills are shared `SKILL.md` files, and every rule of the
label state machine lives in one script, `scripts/is`, so Claude Code and Codex cannot
drift apart.

## Install

**Claude Code**

```
/plugin marketplace add twentyTwo/idea-space
/plugin install idea-space@twentyTwo
```

**Codex**

```
codex plugin marketplace add twentyTwo/idea-space
```

then install `idea-space` from `/plugins` and start a new session.

Both need `bash` and an authenticated [`gh`](https://cli.github.com) on your PATH. On
Windows, `bash` comes with Git for Windows.

## Use

| Claude Code | Codex | What it does |
|---|---|---|
| `/idea-space:capture` | `$idea-space:capture` | File an idea — after checking killed and in-flight ideas for a near-duplicate |
| `/idea-space:status` | `$idea-space:status` | Every open idea by stage; ready ones show the research call and the grill's kill shot |
| `/idea-space:verdict` | `$idea-space:verdict` | Close an idea as approved or killed, with your reason |
| `/idea-space:retry` | `$idea-space:retry` | Re-run a blocked idea, or re-grill one |
| `/idea-space:setup` | `$idea-space:setup` | Create the `is:` labels, or migrate old bare ones |

`capture` and `status` can also trigger from conversation ("save this idea", "what's
waiting on me?"). `verdict`, `retry` and `setup` run only when you invoke them: a
verdict is your decision, and the other two spend model runs or rename labels.

## Using it from any project

Inside your IdeaSpace repo, the commands just work. Anywhere else, the agent will ask
which repo holds your ideas. To skip the question, set it once:

```bash
export IDEASPACE_REPO=<owner>/<name>
```

The script also takes it per command as `--repo <owner>/<name>`, which is what the
agent uses in shells where setting a variable inline is awkward.

## New IdeaSpace repo

`/idea-space:setup` creates the labels. The repo also needs the workflows in
`.github/workflows/`, the prompts in `.github/prompts/`, and two repository secrets:
`CLAUDE_CODE_OAUTH_TOKEN` for research and `GLM_API_KEY` for the grill.
