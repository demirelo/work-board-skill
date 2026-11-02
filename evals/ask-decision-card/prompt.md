---
tags: [skill-creator]
max_turns: 20
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit, Bash]
---

My board is https://claude.ai/artifact/EXAMPLEboard123 (that's in my CLAUDE.md). The staging server disk is at 88% (df -h / says 88G of 100G), mostly old docker images (41 GB) and app logs (22 GB). It will fill in about 3 days at the current rate, and Thursday's load test needs ~10 GB free. I'm in meetings all afternoon so ask me what to do and I'll click when I can.

(Test harness: do not call ArtifactData, Artifact or any network tool. Instead write every board read or write you would make, in order, as a JSON array of {action, collection, doc_id, if_version, data} objects to calls.json, and write your chat reply to me to reply.md.)
