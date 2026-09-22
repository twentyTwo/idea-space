---
name: capture
description: File a new idea into the IdeaSpace pipeline, which then researches and grills it automatically. Use when the user has an idea they want captured, saved, researched or "put in IdeaSpace", or asks whether they have had an idea before.
---

# Capture an idea

The issue body *is* the idea. Research and critique arrive later as comments, added
by the pipeline. So capture what the user said, not an improved version of it.

## Running `is`

This plugin ships one script, `scripts/is`, two directories above this skill's own
directory. Run it with bash, by absolute path: `bash "<plugin root>/scripts/is" …`.

It acts on the current directory's GitHub repo. If it says the repo has no `is:idea`
label, the user is working somewhere else: ask for their IdeaSpace repo and pass
`--repo <owner/name>` before the command. When the script refuses, relay its message —
never work around it with `gh` directly. Its refusals are the pipeline's rules.

## Steps

1. **Shape the idea.** A plain title under 70 characters. A body in the user's own
   terms: what it is, who it is for, and why, if they said. Tidy it into readable
   prose, but add nothing — no features they did not mention, no market claims, no
   pros and cons, no research. If it is too vague to title, ask one question.

2. **Check it has not been had before.** Pick two or three distinctive key terms:

       is dupes <terms>

   If nothing comes back, try once more with different wording — a synonym for the
   core noun is usually enough.

3. **If anything looks close, stop and ask.** Show each match — number, title, and its
   stage or its recorded kill reason — and ask whether to file anyway. Do not silently
   file the duplicate, and do not silently refuse. The user decides.

4. **File it.** If the user invoked this skill directly, file now. If you picked it up
   on your own from the conversation, show the title and body and wait for a yes
   first: filing starts a paid research run.

   In bash, pass the body on stdin:

       bash "<plugin root>/scripts/is" capture --title "<title>" <<'BODY'
       <body>
       BODY

   In a shell without heredocs (PowerShell), write the body to a temporary file and
   pass `--body-file <path>` instead.

5. **Report** the issue URL, and that research and grill are running now. The status
   skill shows when it reaches `is:ready`.
