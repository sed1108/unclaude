---
name: unclaude
description: Rewrite text into plain English before it leaves this session — a Jira or Linear comment, a GitHub or GitLab issue, a PR description or review, a commit message, a Slack or Telegram message, an email, a doc. Use when the user says /unclaude, or asks to post, comment, file, send, summarise or write something up "with /unclaude" or "in plain English". Manual invocation only.
---

## What this is for

Text that stays in the terminal can read however it likes. Text that leaves — into a
ticket, a review, a message, a commit — is read by people who did not watch the work
happen, and often are not engineers. This skill is the rewrite that happens before it
goes.

It is **not** the UN-CLAUDE block shown under each answer on screen. That is a display
hook: it never touches what gets sent anywhere, and it cannot do this job. This skill
is the one that changes what is actually written.

## When it applies

Anything about to be handed to another system or another person: an issue or pull
request comment, a ticket description, a PR body, a review, a commit message, a Slack
or Telegram message, an email, a doc someone else will read.

If the user names the destination ("post the task summary to the Jira ticket with
/unclaude"), do the work first, then rewrite the outgoing text, then send it. The
rewrite is the last step before the write, never a separate message back to the user.

## The rewrite

Write what happened, for someone who was not here.

**Cut these outright**
- Openers that say nothing: "Great question", "You're absolutely right", "Let me…",
  "I'll go ahead and…", "Perfect!"
- Self-narration: "I investigated", "I noticed", "It looks like", "I've gone ahead and".
  Say what is true, not that you found it out.
- Hedges stacked on hedges: "it seems like it might potentially" → say it, or say you
  do not know.
- Closers that add nothing: "Let me know if you'd like me to…", "Hope this helps!"
- Praise for the reader, apologies, and enthusiasm about the work.

**Keep, always**
- Every fact that carries information: names, numbers, file paths, versions, commands,
  error text, ticket and PR ids.
- The decision and the reason for it.
- What is still broken, still unknown, or still needs a person. Never soften a blocker
  into "minor" or drop it because the rest went well.

**Shape**
- Lead with the outcome. What is true now, in one sentence. Then the detail.
- Short sentences. Everyday words. One idea per sentence.
- Active voice, past tense, plain subject: "the migration fails on empty rows", not
  "it was observed that failures may occur".
- Concrete over abstract: "3 of 47 tests fail" beats "some tests are failing".
- No em-dash pile-ups, no "not just X but Y", no rule-of-three flourishes, no bold
  scattered through prose for emphasis.
- As short as it can be while still complete. Cutting a fact is not shortening, it is
  losing information — cut words, never content.

**Formatting for the destination**
- A ticket comment: prose and short lists. No headings unless it is genuinely long.
- A commit message: imperative subject under ~70 chars, blank line, why-not-what body.
- A chat message: one paragraph, no headings, no bullets unless there is a real list.
- Markdown only where the destination renders it. Jira and Telegram do not render
  GitHub-flavoured markdown; do not send it there.

## Before sending

Read the rewrite once as the person receiving it. If any sentence tells them something
about how the work was done rather than what is true, cut it. If a blocker got quieter
in the rewrite than it was in the work, put it back.
