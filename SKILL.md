---
name: work-board
description: Run a personal work board, one live page where the user answers Claude's decision questions with one click and sees every running job and whom it waits for (a claude.ai artifact with a small database). Use it whenever you need a decision from the user during multi-step work, a scheduled "board pickup" runs, a job (PR, review, CI run, deploy step, fix round, long task) starts, changes stage or finishes, the user mentions "the board", "a card", "put it on the board", "what needs me", answers with a bare option number such as "1", or asks you to set up a board for their work. Use it before you ask the user a work decision in chat, because every decision should also be a card.
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
   `{"db": {"rules": [{"path": "", "read": "view", "write": "admin"}]}, "user": {}}`.
   This lets anyone with access read the board, and only the owner (editor)
   and you write. Load the `artifact-capabilities` skill first if it is
   available.
3. Write `meta/board` with `{title, subtitle}` (for example
   `"Acme platform board"`, `"What needs Sam, then every job."`).
4. Record the URL where future sessions will find it (project instructions
   or memory), and say that the page also needs a recurring pickup (below).

Every later publish of the page passes `url=<board URL>` and omits
`capabilities` (that keeps the stored ones). A publish without `url` creates
a second board.

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
- `nextAction`: the next concrete step, one line. When done, the result
  ("Merged as 05f2a62f").
- `updatedAt`: ISO time. Open jobs older than 12 hours show as stale.
- `link`, `machine`: optional.

Use one `batch` write when you touch more than two records.

### 4. Change the page

Edit the project's copy of the page, keep it working on a phone and in
light and dark mode, make it render correctly when a field is missing (old
records lack new fields), and publish with `url=`.

## Writing for the user

Follow the user's own style rules for chat (their CLAUDE.md wins). Cards are
read in seconds: short sentences, active voice, no padding. Never put
secrets, keys, passwords or private file contents on the board, because
everyone with access can read it.
