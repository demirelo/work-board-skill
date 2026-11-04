---
name: work-board
description: |
  Personal work board: the user's live claude.ai page that holds open decision cards and every running job. Use this skill even when the user never says "board". It covers three intents:
  - The user owes a decision. You need their call on a choice (drop a version, pick a time, approve a plan), or they say "ask async", "don't block on me", "keep going" or "park it". Post a decision card first, then continue other work.
  - The user asks about their own queue: what waits on them, what is blocked on their answer, what needs them, what is in flight. This also covers a reply that is only an option number, such as "2".
  - A job changes state. A PR opens, merges or fails. CI, a review or a deploy starts or ends. A long task stalls. This also covers the user reporting such news, saying the board is stale, or a "board pickup" run.
  Do not use it to build kanban apps or dashboards, or for advice on Jira, Trello or GitHub project boards.
---

# Work board

A work board is one page that answers two questions at a glance: **what
needs me** and **what is every job waiting for**. The user answers decision
cards there with one click and an optional note, so they do not have to type
in chat, and they can see what you are doing without asking. A board only
helps if it is current: a question asked only in chat, or a job left at
"working" after it finished, breaks that trust.

The page template is `assets/board.html`. Its data lives in the artifact's
database, which you read and write with the `ArtifactData` tool (load it with
ToolSearch `select:ArtifactData` if it is deferred). Field shapes are in
`references/schema.md`; read it before your first write in a session.

## Find the board (or create one)

Look for the user's board URL in project instructions (CLAUDE.md, AGENTS.md)
or memory. Use that board; do not make a second one.

If there is none and the user wants one:

1. Copy `assets/board.html` into the project (for example
   `docs/work-board.html` or a private notes folder) so later edits have a
   source file.
2. Publish it with the Artifact tool and these capabilities:
   `{"db": {"rules": [{"path": "", "read": "view", "write": "admin"}]}, "user": {}, "comments": {}}`.
   This lets anyone with access read the board, and only the owner (editor)
   and you write. Load the `artifact-capabilities` skill first if it is
   available. `comments` is the instant trigger: when the user confirms an
   answer, the page also sends a short "Send to Claude" comment ("Board
   decision on ... (card <id>): <answer>"), which wakes a session that
   watches the board with auto-replies armed. Treat that comment exactly like
   a pickup of that one card, then update the card as usual. Every session
   that watches the board gets the comment: act only on cards whose id starts
   with your own project prefix, and leave other cards (and their threads) to
   their session. A session watches only the boards its project uses.
   The platform delivers the comment to every watching session; the page
   cannot pick one. So every card you create carries `session` (your session
   name, as other sessions address you), and the page starts the message
   with "For session <name>:". Act only on a message addressed to you, or on
   a card with no `session` whose prefix is yours. Otherwise do nothing: no
   reply in the thread, no card update. The scheduled
   pickup stays the fallback for when no session is watching.
3. Write `meta/board` with `{title, subtitle}` (for example
   `"Acme platform board"`, `"What needs Sam, then every job."`).
4. Record the URL where future sessions will find it (project instructions
   or memory).
5. Set up the recurring checks (next section) and tell the user they run.

Every later publish of the page passes `url=<board URL>` and omits
`capabilities` (that keeps the stored ones). A board published before the
trigger existed needs one publish with the full capability set above to add
`comments`. A publish without `url` creates
a second board.

## Recurring checks (set them up, keep them alive)

A board only works hands-free if something reads it while the user is away.
Set up a recurring **board pickup** when you create a board, and again at
the start of every session that uses an existing board, because session
schedules end with the session.

- **Cadence:** every 10–15 minutes while work is running. Offer a slower
  cadence (hourly) for quiet periods; never faster than every 5 minutes.
- **Prompt:** use `references/pickup-prompt.md` with the board URL filled
  in. It is the move-2 pickup plus the heartbeat, and it replies only
  `board: nothing new` when nothing waits, so quiet ticks stay short.
- **Scheduler:** use what the host offers, in this order:
  1. a durable scheduled task or routine (survives restarts), when the user
     agrees to one;
  2. a session cron (for example `CronCreate` in Claude Code), re-created in
     every new session;
  3. a `/loop` with the pickup prompt, as a last resort.
  Check first whether one already exists (list the schedules) so you never
  run two pickups against the same board.
- **Long jobs:** a schedule only wakes you on its tick. For background jobs
  (CI runs, builds, agent jobs), also start a waiter that ends when the job
  ends, so a finished job is handled within minutes, not at the next tick.
- **Stop:** when the user pauses the work or the board is retired, delete
  the schedule and say so.

Record in project instructions or memory that the pickup must be re-armed
in each session; the heartbeat line on the page shows when it stopped.

## Times

Take every timestamp from the clock (`date -u +%FT%TZ`), never from memory
or an estimate. A time in the future shows as "just now" and hides a stale
job; a guessed past time marks a fresh job stale.

## The four moves

### 1. Ask the user (a decision card)

Every decision the user must make becomes a `needs` card, even when you also
ask in chat. The chat question and the card say the same thing.

- `doc_id`: short kebab-case topic (`db-migration-window`).
- `question`: one line ending in "?". Name the choice, not the background.
- `context`: 2–5 short sentences: what you found, the evidence (numbers with
  the command that produced them), why it matters now.
- `options`: 2–4 choices, your recommendation **first** with
  `recommendFirst: true`. Each option must be actionable as written: when the
  user picks it, you can start without asking again. Use `{label, detail}`
  when an option needs a one-line consequence.
- `priority`: `now` (blocks running work), `today`, or `later`.
- `blocks`: what waits on the answer, if anything.
- `status: "open"`, `askedAt` (ISO time).

One decision per card. The page also offers "Other answer…", so you do not
need a catch-all option.

### 2. Pick up answers

Run this whenever a scheduled pickup fires, and at natural pauses. Query
`needs` where `status == "answered"`, `answeredVia == "board"`,
`pickedUp == false`. For each card:

1. Treat `answer` (and `note`) exactly like the user typing that choice in
   chat, inside the card's question and context. Your normal safety and
   review rules still apply.
2. If the note asks something, or a free-text answer is unclear, ask in chat
   and add a new card. Do not guess.
3. Start the work, then update the card with `if_version`: `pickedUp: true`,
   `pickedUpAt`, `outcome` (one line: what you started or did). The page
   shows `outcome` under "Recent decisions".
4. Tell the user one line per card.

End every pickup by writing the heartbeat `meta/claude` →
`{checkedAt: <ISO>, note: "<one line>"}`, even when nothing was waiting; the
page shows "Claude checked the board N min ago". If nothing was waiting,
reply only `board: nothing new`.

**Chat answers count too.** If the user answers a card in chat ("1", "go
with the second"), map it to that card's option and write `status:
"answered"`, `answer` (the full option text), `answeredVia: "chat"`,
`answeredAt`, `pickedUp: true`, `pickedUpAt`, `outcome`. Otherwise the card
stays open on the board.

### 3. Keep jobs current

A job is anything that runs longer than one turn: a PR, a review round, a
CI run, a deploy step, a long task on another machine. Update its `jobs`
record **in the same turn as the change**. Change points: started, sent to
review, CI started, CI green or red, waiting for someone, merged, failed,
blocked, done.

- `stage`: one of the stages in `references/schema.md`; an unknown stage
  lands in the wrong group.
- `waitingOn`: who or what, by name ("Dana's approval on 1b2bee29",
  "CI on #177", "you: card db-migration-window").
- Use a `needs-*` stage only when the user must act **and** an open `needs`
  card says what to do; write `waitingOn: "you: card <id>"`. The page lists
  only cards under "Needs you" (a job cannot take an answer), links the job
  to its card, and flags a `needs-*` job without an open card. If someone
  else must act (a reviewer, CI), use `waiting-others` and name them.
- `nextAction`: the next concrete step, one line. When done, the result
  ("Merged as 05f2a62f").
- `updatedAt`: ISO time. Open jobs older than 12 hours show as stale.
- A job that waits on you (`waitingOn: "Claude: ..."`) is your own next
  step. Start it in the same turn, or name what it waits for instead. The
  page marks it "Claude has not acted" after 60 minutes without a change.
- `link`, `machine`: optional.

Use one `batch` write when you touch more than two records.

### 4. Change the page

Edit the project's copy of the page, keep it working on a phone and in
light and dark mode, make it render correctly when a field is missing (old
records lack new fields), and publish with `url=`.

## Jira, Linear, GitHub Projects and other trackers

The board does not replace a team tracker. The tracker holds the team's
tasks; the board holds the conversation between one person and Claude:
the decisions Claude needs from them, and what Claude's own running work
waits for.

- Keep the tracker the source of truth for tasks. Do not copy a team's
  backlog onto the board.
- A board job that belongs to a ticket puts the ticket URL in `link` and the
  key in the title ("PROJ-142: rate limiter"). Update the ticket in the
  tracker as usual (with its own tool or connector, if available); the board
  only mirrors the stage of the part Claude is doing.
- A decision that the team must see (scope, priority, a design choice) goes
  to the tracker or the team channel as well. The card's `context` names
  where, so the answer can be posted back there.
- If the user wants no second place at all, skip the jobs half and use only
  decision cards.

## Writing for the user

Follow the user's own style rules for chat (their CLAUDE.md wins). Cards are
read in seconds: short sentences, active voice, no padding. Never put
secrets, keys, passwords or private file contents on the board, because
everyone with access can read it.
