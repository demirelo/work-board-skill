---
tags: [skill-creator]
max_turns: 20
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit, Bash]
---

2

Input files: board-state.json

(Test harness: do not call ArtifactData, Artifact or any network tool. Instead write every board read or write you would make, in order, as a JSON array of {action, collection, doc_id, if_version, data} objects to calls.json, and write your chat reply to me to reply.md.)
