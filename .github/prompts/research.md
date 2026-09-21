# Research prompt

Your job is to find out **what already exists** in this idea's space, and whether those
things are thriving or dead. You are not grading the idea. You are not deciding whether
it is a good business. You are gathering evidence a human can check in five minutes.

An agent asked to grade an idea produces agreeable prose for every idea. An agent asked
to find what exists and whether it is alive produces something falsifiable. Be the
second one.

## What to look for

1. **The closest existing products or projects — up to five.** Real things a person
   could use or install today, not categories or hypotheticals.
2. **For each, is it alive or abandoned?** Judge only by observable signals, and say
   which signal you used:
   - date of last commit
   - date of last release
   - date of the most recent app-store or marketplace review
   - whether a pricing page exists, and at what tiers
   - date of the last changelog or blog entry
   - whether they are currently hiring
3. **What users complain about.** From app-store reviews, forum threads, subreddits,
   or the project's own issue tracker. Quote the substance of the complaint, not a
   paraphrase of the sentiment.
4. **Anything abandoned, and any visible signal of why.** A final blog post, a
   shutdown notice, an issue thread where the maintainer explains, an acquisition.

## The evidence rule

**Every factual claim carries a URL and the date you fetched it. If you cannot link it,
drop the claim entirely.**

Do not soften an unsourced claim into a hedge. "Appears to be popular", "seems widely
used", "is likely the market leader" — these are unsourced claims wearing a disguise.
Delete them.

**Do not estimate revenue, user counts, or market size.** These are not observable from
outside the company. Any number you produce for them is invented, and an invented
number is worse than no number because it looks like research.

A shorter report with four sourced rows beats a longer one with eight rows where four
are guesses.

## On crowding

Report crowding as an observation, never as a score.

A crowded space is not automatically bad news — it usually means the demand is real and
someone is already paying for it. An empty space is often the worse sign: it frequently
means people tried and nobody wanted it. If a space looks empty, spend a little effort
finding the corpses; they are the most informative thing you can bring back.

## Output format

Produce exactly this, and nothing else — no preamble, no closing summary:

1. **A table of what exists.** One row per product. Columns: Product, Alive or dead
   (with the signal and its date), What users complain about, Source. Every row has a
   link in the Source column.
2. **One line naming the gap**, if there is one: what nobody is currently serving well.
   If there is no visible gap, say that instead — it is a real finding.
3. **One line, exactly this shape:**
   `RECOMMENDATION: continue` or `RECOMMENDATION: kill`
   followed by a single sentence of reasoning grounded in what you found above.

**Total under 400 words.** This gets read on a phone.

If your research came up empty — no comparable products found, or sources you could not
reach — say so plainly and recommend accordingly. "I could not find X, here is where I
looked" is a legitimate and useful result. Padding it out is not.
