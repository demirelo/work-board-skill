# work-board

A Claude Code skill for a personal **work board**: one live page where you
answer Claude's decision questions with one click, and see every running
job and whom it waits for.

The board is a claude.ai artifact with a small database. Claude writes
decision cards and job records; you click an option (or type your own
answer); Claude picks the answer up and acts on it.

## What the page shows

- **Needs you** — decision cards, urgent first, recommended option marked.
- **Answered, waiting for Claude** — your clicks Claude has not acted on yet.
- **Blocked / Working / Waiting for others** — every job and who it waits for.
- **Recent decisions, Recently done, Backlog** — collapsed by default.
- A heartbeat line ("Claude checked the board 5 min ago") and a stale mark on
  jobs not updated for 12 hours.

## Install

Copy this folder to `~/.claude/skills/work-board/` (or a project's
`.claude/skills/`). Then ask Claude: "set up a work board for this project".
Claude publishes `assets/board.html` as a private artifact and records its
URL in your project instructions.

For answers to be picked up while you are away, schedule a recurring prompt
(for example every 15 minutes) that says "board pickup".

## Files

- `SKILL.md` — the workflow Claude follows.
- `references/schema.md` — data fields and stages.
- `assets/board.html` — the page template.
- `evals/` — test cases (`claude plugin eval .` runs them).
