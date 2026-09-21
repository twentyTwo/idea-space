---
name: setup
description: "Create or migrate the ms: labels the IdeaSpace pipeline runs on. Use only when the user explicitly asks to set up IdeaSpace or its labels."
disable-model-invocation: true
---

# Set up the labels

`ms setup` makes sure the repo has the seven labels the pipeline reads and writes:
`ms:idea`, the four stages `ms:captured` `ms:running` `ms:ready` `ms:blocked`, and the
two verdicts `ms:approved` `ms:killed`. It is safe to run any number of times:

- a label that is already there is left alone;
- a label that exists under its old bare name (`captured`) is **renamed**, which keeps
  it on every issue that carries it;
- a label that does not exist at all is created.

Before renaming anything it checks two things, and refuses if either fails: the
default branch's `research.yml` must already match the `ms:` names — otherwise the
rename would stop the deployed pipeline triggering — and no issue may be mid-run.

## Running `ms`

This plugin ships one script, `scripts/ms`, two directories above this skill's own
directory. Run it with bash, by absolute path: `bash "<plugin root>/scripts/ms" …`.
It acts on the current directory's GitHub repo, or on `--repo <owner/name>` given
before the command.

## Steps

1. Run `ms setup` and relay each line it prints.

2. If it refused, explain which check failed and what clears it.

3. Setup handles labels only. If the user is standing up a new IdeaSpace repo, tell
   them it also needs the workflows (`.github/workflows/research.yml` and
   `verdict-guard.yml`), the prompts in `.github/prompts/`, and the repository secrets
   `CLAUDE_CODE_OAUTH_TOKEN` and `GLM_API_KEY`.
