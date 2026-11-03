# Board pickup prompt

Use this text for the recurring check. Replace `<BOARD_URL>`.

```
Board pickup: query the work board <BOARD_URL>, collection "needs", where
status == "answered" and answeredVia == "board" and pickedUp == false.
Treat each answer (and its note) exactly like the user typing that choice
in chat, within the card's question and context, under the normal rules.
Act on it, then update the card with pickedUp: true, pickedUpAt (clock
time) and a one-line outcome, pinned with if_version, and tell the user one
line per card. If an answer is unclear or the note asks something, ask in
chat and add a new card. Also check that open jobs on the board are current
and no time is in the future. Any open job whose waitingOn starts with
"Claude" is your own next step: start it now or say why it waits. Always write the heartbeat meta/claude
{checkedAt, note}. If nothing was waiting, reply only "board: nothing new".
```
