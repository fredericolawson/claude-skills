---
name: sweep-improvements
description: Sweep one app's open improvements against the code. Close what has shipped, close the built part of a partly built record and move the rest to a new one, rewrite records that have gone stale or grown long, and split records that hold more than one ask. Run by hand with /sweep-improvements; never on a commit or a push.
disable-model-invocation: true
argument-hint: "[app name] [improvement numbers]"
---

Sweep the open improvements for one app: Stitchhub, CJ hub, Rushe or Lifted.
Each has its own improvements MCP server with the same tools
(`list_improvements`, `get_improvement`, `add_improvement`,
`update_improvement`, `add_improvement_note`, `complete_improvement`).

This skill reads code and writes improvement records. It changes no code. It
files a new improvement only when it splits an existing one (step 5).

## 1. Pick the app

- An argument names the app. Numbers after it limit the sweep to those
  improvements.
- With no argument, use the repo you are in: `stitchhub` is Stitchhub,
  `cj-hub` and `cj-next-web` are CJ hub, `rushe-brain-ui` and `rushe-ops-ios`
  are Rushe, `lifted/*` is Lifted. Anywhere else, ask which app.
- Server ids change between sessions. Find the right one with a tool search
  for `list_improvements` and read each description ("The team's improvements
  for Stitchhub"). If the app's server is not connected, say so and stop.

## 2. Read the open list

Call `list_improvements` once for each of `new`, `partial` and `hold`. Call
`get_improvement` on every item that has notes, because a note may have
changed or dropped part of the ask. Leave done improvements alone.

Where the app's entries carry `started` (CJ hub does), a record with
`started: true` is one someone is working on now, in `start` or `partial`.
The `start` list is `/start-improvements`' work and the sweep does not read
it; a started record in `partial` the sweep does read, with the limits in
steps 3 to 5.

## 3. Check each one against the code

The code is the evidence, not a commit message. For each item, check every
task and every part of "What" against the app as it is now, using
`git log` since the item was filed to find where to look. Give each item one
verdict:

| Verdict | Meaning |
|---|---|
| Shipped | Every part is built. |
| Partly built | Some parts are built, some are not. |
| Not started | Nothing is built, and the ask still makes sense. |
| Overtaken | The app has changed so that the ask no longer applies, or another improvement covers it. |

Built means the team can use it in the app today. Work that is in the code
but not usable on its own (a table with no page, a page nothing links to)
counts as not built; when that is all there is, the verdict is Not started.

The sweep leaves no record in `partial`. A partly built record is split
(step 5): the original is cut down to what is built and completed, and what
is left moves to a new improvement. This includes records already in
`partial` when the sweep starts, except a started one: someone is building
the rest now, so it stays in `partial` and the sweep only adds a note on
what is built.

When you cannot tell, say so and leave the item as it is.

## 4. Decide which records to rewrite

A record is worth rewriting when a teammate reading it today would be misled
or slowed down. Rewrite for these reasons only:

- **Stale content:** it describes a page, button or record as the app no
  longer has it, or uses a name that has since changed.
- **Dropped or settled parts:** a later note decided against part of the ask,
  or answered a question the details still pose.
- **Build instructions:** file names, URL parameters or how to build it. The
  record says what the app should do and why, in the app's own words.
- **Length:** repeated points, long preamble, or a title that is not one
  plain line naming the ask.
- **Missing fields:** no icon or no type.

Keep to these limits:

- Keep the requester's meaning. If you are unsure what they meant, leave the
  text and flag it.
- Keep the "Why" and the "Source" sections; shorten them, do not remove them.
- Follow `add_improvement`'s writing rule: "### What", "### Why", "### Notes"
  and "### Source" as they apply, with the separate things to build as a task
  list under What.
- Progress still goes in a note, never in the details. The one progress edit
  to details is ticking a built task (`- [ ]` to `- [x]`).
- Details keep no history and notes are permanent. When a rewrite removes
  anything a teammate might look for later (a dropped ask, a decision), add a
  note saying what was removed and why.
- Do not change the owner, the star, a hold, or the started switch, unless
  asked.
- Do not rewrite a started record. Someone is working from its text; flag
  what is stale in the report instead.

## 5. Decide which records to split

Split a record for one of two reasons:

- **It holds two or more asks that would ship separately:** they touch
  different pages, belong to different owners, or one has shipped and the
  rest is a distinct piece of work. One ask with several unbuilt tasks stays
  one record; a task list is not a reason to split.
- **It is partly built.** The built parts stay in the original, which is
  completed. What is left moves to one new improvement, or to several when
  the remainder holds asks that would ship separately.

How to split:

- **The original keeps its number and its notes.** It keeps the ask its title
  names and is rewritten down to that one ask. When that ask has shipped, the
  original is completed and the rest moves out. For a partly built record,
  the original keeps only the built parts, every task ticked, with the title
  rewritten when it no longer names what was built.
- **Each other ask becomes a new improvement,** written to
  `add_improvement`'s rule. It carries over the Why, the Source and the
  owner, takes its own icon and type, and names the original in Notes
  ("Split from #12"). Do not star it or set it started.
- **Leave a trail.** Notes cannot be moved. Add a note on the original saying
  what moved and to which numbers. Where an existing note on the original
  matters to a moved ask, restate its point in the new record's Notes.
- Do not split a record on hold or a started one. Do not split unbuilt asks that depend on
  each other so closely that one cannot ship without the other.

## 6. Apply without asking

Do not propose and wait. Running the sweep is the go-ahead for every
complete, note, rewrite and split it decides on (Freddie, 2026-11-13), so
apply each one as soon as the checks in steps 3 to 5 are done.

The records belong to the team, and `complete_improvement` and
`add_improvement` both post to their Slack, so a wrong write is seen at
once. Write only what the code or a note settles. Where you are unsure of
a verdict or of what the requester meant, write nothing on that record and
name it in the report with the question that would settle it.

## 7. How to apply each

- **Shipped:** `complete_improvement` with a note of one or two plain
  sentences saying what changed for the team, in the app's own words. No
  file names, no code.
  Completing a started record turns its switch off.
- **Partly built:** apply it as a split, below. The closing note on the
  original covers only what shipped. Never set status `partial`.
- **Overtaken:** complete it with a note saying why, naming the improvement
  that covers it when there is one.
- **Rewrite:** `update_improvement` with the new title, details, icon or
  type, plus the note from step 4 when something was removed.
- **Split:** `add_improvement` for each new record first, so their numbers
  are known. Then rewrite the original with `update_improvement` and add the
  note naming those numbers. Complete the original if its own ask shipped.

The people who read these do not read code. Write every note, every
rewrite and every new record for them.

## 8. Report

Show one table of every open item: number as a link, title, verdict,
whether it is started, and what was done (completed, noted, rewritten,
split, or left). Under it, give each closing note and progress note as
written, and for each rewrite and split the old title beside the new one,
with the new numbers for a split. Name any item you could not judge and
what would settle it.
