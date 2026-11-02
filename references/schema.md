# Board data schema

Artifact: the user's board URL (see SKILL.md, "Find the board").
Access rule: everyone with access reads; only editors (the owner) and
Claude (via ArtifactData) write. Every write to an existing document needs
`if_version` (the `version` you last read).

## `needs/<id>` — owner decisions

| field | type | notes |
|---|---|---|
| question | string | one line, ends with "?" |
| context | string | 2–5 sentences; the page shows it behind "Why" |
| options | array | strings, or `{label, detail}` objects; recommendation first |
| recommendFirst | bool | true when options[0] is the recommendation |
| priority | `now` \| `today` \| `later` | sort order; missing = `today` |
| blocks | string | what waits on the answer (optional) |
| link | string | evidence file or URL (optional) |
| status | `open` \| `answered` | |
| askedAt | ISO string | older cards may have "2 Nov" text |
| answer | string | the chosen option label, or free text from "Other" |
| note | string | optional owner note |
| answeredVia | `board` \| `chat` | |
| answeredAt | ISO string | |
| pickedUp | bool | false until Claude acts on a board answer |
| pickedUpAt | ISO string | |
| outcome | string | one line: what Claude started or did |

The page writes only: `status, answer, note, answeredVia: "board",
answeredAt, pickedUp: false`. Claude writes everything else.

## `jobs/<id>` — one record per job

| field | type | notes |
|---|---|---|
| title | string | short name |
| stage | see below | |
| waitingOn | string | who or what, by name |
| nextAction | string | next concrete step, or the result when done |
| machine | string | where it runs, e.g. laptop, build server, CI (optional) |
| updatedAt | ISO string | older than 12 h shows as stale |
| link | string | PR, run or evidence (optional) |
| order | number | lower first within a group (optional) |

Stages and the group the page shows them in:

| stage | group |
|---|---|
| needs-answer, needs-review, needs-merge | Needs you |
| blocked, failed | Blocked |
| working, ci | Working |
| review | Waiting for others (shown as "In review") |
| pr | Waiting for others (shown as "PR open") |
| queued, waiting-others | Waiting for others |
| backlog | Backlog (collapsed) |
| done | Recently done (collapsed) |

## `meta/claude` — heartbeat

`{checkedAt: ISO, note: string}` — written at the end of every pickup. The
page shows "Claude checked N min ago" and turns it red after 45 minutes.

## `meta/board` — page header

`{title, subtitle}` — shown as the page heading and subtitle. Missing fields
keep the defaults ("Work board", "What needs you, then every job…").
