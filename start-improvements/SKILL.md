---
name: start-improvements
description: Build the improvements one app's team has moved to status Start. Reads each Start record with its notes, builds it in the app's repo, verifies, commits, and closes the record. Run by hand with /start-improvements; never on a commit or a push.
disable-model-invocation: true
argument-hint: "[app name] [improvement numbers]"
---

Build the improvements in status `start` for one app: Stitchhub, CJ hub,
Rushe or Lifted. Start means the team has picked the record to be worked on
next and nobody has begun it. Each app has its own improvements MCP server
with the same tools (`list_improvements`, `get_improvement`,
`add_improvement`, `update_improvement`, `add_improvement_note`,
`complete_improvement`).

Moving a record to Start is the go-ahead to build it. Do not ask again
before building. Stop to ask only where step 4 says so.

This skill does not tidy the list. Rewriting, splitting and closing records
that have already shipped is `/sweep-improvements`.

## 1. Pick the app

- An argument names the app. Numbers after it limit the run to those
  improvements, which must be in Start; say so for any that is not, and
  leave it.
- With no argument, use the repo you are in: `stitchhub` is Stitchhub,
  `cj-hub` and `cj-next-web` are CJ hub, `rushe-brain-ui` and `rushe-ops-ios`
  are Rushe, `lifted/*` is Lifted. Anywhere else, ask which app.
- Server ids change between sessions. Find the right one with a tool search
  for `list_improvements` and read each description. If the app's server is
  not connected, say so and stop.
- If the server's `status` filter does not offer `start`, that app has no
  Start status, or the session connected before it was added. Say so and
  stop. Do not fall back to `new`.

## 2. Read the Start list

Call `list_improvements` with status `start`. If it is empty, say so and
stop.

Call `get_improvement` on every record, whatever its notes count. Notes come
oldest first and a later note can change, narrow or drop part of the ask;
Markdown files attached to a note are part of the request. The ask is the
details as the notes leave them.

Where the app's entries carry `started` (CJ hub does), a record already
started is one someone else is building now: leave it, and name it in the
report.

Order the work: starred records first, then oldest first. Where one record
depends on another in the list, build the one it depends on first.

Show the order as a short list (number as a link, title, one line on what
you will build), then begin. This is a statement, not a question.

## 3. Prepare each record

Before writing code for a record:

- Work in the repo the record is about. For Rushe and Lifted that may be the
  web repo, the iOS repo or both. Read that repo's `CLAUDE.md` and the rules
  file for the area before planning.
- Check the app as it is now. If the whole ask is already built and usable,
  write no code: complete the record (step 6) and move on. If part is built,
  build the rest.
- Use the record's `page` field and its screenshots to find where the team
  was looking when they filed it.

## 4. When to stop and ask

Build without asking when the ask is clear enough that a wrong guess costs a
small follow-up change. Stop on a record, and carry on with the others, when:

- the ask can be read two ways that lead to different things being built;
- it needs a decision that belongs to the team (a policy, wording a customer
  will see, what happens to existing data);
- it needs something irreversible or outside the repo: deleting or
  rewriting live data, sending to customers, spending money, a new secret or
  a new external account;
- it is far larger than its record suggests (a new section of the app, not a
  change to one).

For a stopped record, add a note with the question in plain words a teammate
can answer without reading code, and set the record to `hold`. Put the same
question in the final report.

## 5. Build, verify, commit

One record at a time, one commit per record:

- Before writing code for a record, turn its switch on:
  `update_improvement` with `started: true`, where the app has it. The
  team's board then shows the card glowing while you work. Turn it on for
  one record at a time, never the whole list up front.

- Build the whole ask, every task under What. Follow the repo's own rules
  for style, tests, error handling and migrations.
- Verify the way the repo's `CLAUDE.md` says (typecheck, lint, tests, build,
  a live read). Fix what fails before committing. No browser check unless
  the repo asks for one.
- Commit and push as the repo's `CLAUDE.md` says. The commit message names
  the improvement number.
- A failure in one record does not stop the run. Revert what does not work,
  note what happened, and go on to the next.

## 6. Close the record

Every record you started leaves this step with its switch off.
`complete_improvement` and a move to `hold` turn it off by themselves; a
record set to `partial` or left in Start needs `update_improvement` with
`started: false`, in the same call as the status where there is one.

Write every note for the people who filed the record. They do not read
code: no file names, no code, the app's own words.

- **Built in full:** `complete_improvement` with one or two plain sentences
  saying what changed for the team and where to find it. Tick the built
  tasks in the details first (`- [ ]` to `- [x]`).
- **Built in part:** only when the built part is usable on its own. Tick the
  built tasks, add a note saying what is in the app now and what is left,
  and set the record to `partial`.
- **Nothing usable built:** add a note saying what was tried and what is in
  the way. Leave it in Start, unless step 4 applies, in which case `hold`.
- **Needs a step from a person** (a setting to turn on, a key to add, a
  device to test on): complete it only if the feature works without that
  step. Otherwise note the step and set `partial`.

Do not change the owner or the star. Do not file new improvements; name
anything you noticed along the way in the report instead.

## 7. Report

List every Start record with its link and what happened: completed, partly
built, put on hold with its question, or left (say so for one left because
someone else had it started). For each one built, give the
commit and say what was verified and what no one has seen in a browser or
on a device.
