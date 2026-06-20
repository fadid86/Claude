# Trello automation

Shared Claude Code tooling for managing team Trello boards.

## Contents

| Path | What it is |
|------|------------|
| [`.claude/skills/trello-board-management/SKILL.md`](.claude/skills/trello-board-management/SKILL.md) | Reusable skill for reading/creating/updating cards, labels, checklists, and lists on any member's board via the Trello REST API. |
| [`routines/daily-backlog-labeling.md`](routines/daily-backlog-labeling.md) | Prompt + setup for a scheduled routine that labels the BACKLOG list every morning. |

## Credentials

This tooling never stores Trello API keys or tokens in the repo. Each person supplies their
own via environment variables — `TRELLO_API_KEY`, `TRELLO_TOKEN`, and `TRELLO_BOARD_ID` — set
in their Claude Code cloud environment or local shell. See the skill's **Setup** section.

If a key or token is ever pasted into a file, chat, or screenshot, **revoke it and generate a
new one** — exposure is not undone by deleting the text.
