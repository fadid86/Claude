# Routine: Daily BACKLOG labeling (8 AM)

A self-contained prompt for a scheduled [Claude Code routine](https://code.claude.com/docs/en/routines).
Every day it ensures each card in the **BACKLOG** list carries one Type label + one Status
label, using the team standard defined in the `trello-board-management` skill.

Each employee creates their own routine with their own `TRELLO_*` environment variables.

---

## 1. Routine prompt (paste this into the routine's **Instructions**)

> Use the `trello-board-management` skill in this repo.
>
> Goal: ensure every open card in the **BACKLOG** list of my Trello board carries exactly
> one Type label and one Status label, following the skill's label standard.
>
> Steps:
> 1. Read `TRELLO_API_KEY`, `TRELLO_TOKEN`, and `TRELLO_BOARD_ID` from the environment.
>    If any is missing, stop and report which one.
> 2. Resolve the BACKLOG list ID and all label IDs live from the board (board-first rule).
> 3. Fetch all open cards in BACKLOG with their current labels.
> 4. For each card:
>    - If it already has both a Type and a Status label, leave it unchanged.
>    - Otherwise, assign the best-fit Type label and best-fit Status label from the card
>      title and description. Make a confident best guess for **every** card — do not leave
>      any card without a full Type + Status pair.
>    - Apply labels additively (POST `idLabels`); never remove an existing correct label.
> 5. Status guidance: `Urgent` = time-critical / due imminently; `Waiting On` = blocked by
>    an external party; `Active` = in my court / nothing blocking (the default).
> 6. Do not create, move, or archive cards. Labeling only.
>
> Definition of done: every open BACKLOG card has exactly one Type label and one Status
> label. No per-card write-up is required — make the best-guess call on every card and move on.

---

## 2. Create the routine (one time)

Routines are created at <https://claude.ai/code/routines> (**New routine**), or with
`/schedule daily backlog labeling at 8am` from the desktop/terminal app. `/schedule` is
disabled **inside** a Claude Code web session, so use the web UI there.

| Field | Value |
|-------|-------|
| **Prompt** | The block in section 1 above. Select your model. |
| **Repository** | `fadid86/claude` (this repo — makes the skill available) |
| **Environment** | One whose **Environment variables** include `TRELLO_API_KEY`, `TRELLO_TOKEN`, `TRELLO_BOARD_ID` |
| **Network access** | **Custom** → add `api.trello.com` (the default Trusted allowlist does not include it) |
| **Schedule trigger** | **Daily at 8:00 AM** — entered in your local zone; runs may start a few minutes late due to stagger |
| **Connectors** | None required — the skill calls the Trello REST API directly over HTTPS |

After creating, click **Run now** once to confirm it works end to end, then open the run to
read the summary (a green status only means the session started, not that the task succeeded).

## 3. Notes

- **Idempotent:** cards already carrying a Type + Status pair are left untouched, so running
  daily only touches new or partially-labeled cards.
- **Per employee:** each person points the routine at their own board via their own
  `TRELLO_BOARD_ID` and their own key/token. Nothing is shared between boards.
- **Butler alternative:** the optional Butler daily sort (see the skill) re-orders cards by
  label; it does not assign labels. This routine assigns them.
- **Want a daily report?** The prompt is set to best-guess every card with no write-up. To
  get a summary instead, append this to the prompt: *"Then post a short summary — total
  cards, how many were already labeled, how many you labeled, and any genuine judgment calls."*
