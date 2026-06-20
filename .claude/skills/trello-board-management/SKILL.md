---
name: trello-board-management
description: Full Trello board management for any team member — read, create, update, archive cards, manage labels, checklists, and lists. Use when someone asks to add, update, move, label, or review anything on their Trello board.
---

# Trello Board Management

A reusable skill for managing any team member's Trello board through the Trello REST API.
Nothing here is tied to one person: each employee supplies their own credentials and board.

## Setup (per employee — one time)

Credentials are read from **environment variables**. Never paste a real key or token into
this file, into committed code, or into card content.

| Variable | What it is | Where to get it |
|----------|------------|-----------------|
| `TRELLO_API_KEY` | Your personal Trello API key | <https://trello.com/power-ups/admin> → your app → API key |
| `TRELLO_TOKEN` | A token authorizing that key against your account (grants read/write to your boards) | Generate from the same API key page |
| `TRELLO_BOARD_ID` | The board this skill manages | See **Find your board ID** below |

- **In a Claude Code cloud environment / routine:** set these under the environment's
  **Environment variables**. They are injected at runtime and never committed.
- **Locally:** export them in your shell profile or a git-ignored `.envrc`. Do not commit them.

### Find your board ID

```bash
curl -s "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN&fields=id,name,shortUrl"
```

Match by name to get the board `id`, then set it as `TRELLO_BOARD_ID`.

## Base URL pattern

```
https://api.trello.com/1/{resource}?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN
```

## Board-first rule (important)

List IDs and label IDs are **different on every board**. Always resolve them live from the
board before acting — never reuse IDs from another person's board. If the user uploads a
screenshot of their board, read it to resolve ambiguity. Never ask a question that the live
board or a screenshot can answer.

### Resolve list IDs by name

```bash
curl -s "https://api.trello.com/1/boards/$TRELLO_BOARD_ID/lists?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN&fields=id,name"
```

### Resolve label IDs by name

```bash
curl -s "https://api.trello.com/1/boards/$TRELLO_BOARD_ID/labels?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN&fields=id,name,color"
```

## Recommended label system (team standard)

Labels are prefixed with numbers so a daily Butler sort orders them by priority
(alphabetical sort = priority order). Apply this standard on each board, creating any that
are missing (see **Create a new label**). Label **IDs** are per-board — resolve them live;
the **names + colors** below are the shared contract.

### Type labels (primary sort) — apply exactly one

| Name | Color |
|------|-------|
| 1 🔵 Client Work | blue |
| 2 🟣 Internal / Ops | purple |
| 3 🤖 Tech / AI | sky |
| 4 🟢 Recurring | green |
| 5 ⚫ Delegated | black |
| 6 📢 Digital Marketing | pink |

### Status labels (tiebreaker) — apply exactly one

| Name | Color |
|------|-------|
| 1 🔴 Urgent | red |
| 2 🟠 Active | orange |
| 3 🟡 Waiting On | yellow |

**Tagging rule:** every card gets one Type label **and** one Status label.

- **Status meanings:** `Urgent` = time-critical / due imminently. `Waiting On` = blocked by
  an external party (client, vendor, teammate). `Active` = in your court, nothing blocking
  (the default when nothing else clearly fits).
- **Delegated (Type):** the card is assigned to a team member; the owner is monitoring only.

## Common operations

Each example assumes `TRELLO_API_KEY`, `TRELLO_TOKEN`, and `TRELLO_BOARD_ID` are set.

### Get all open cards

```bash
curl -s "https://api.trello.com/1/boards/$TRELLO_BOARD_ID/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN&filter=open&fields=id,name,idList,idLabels"
```

### Create a card

```bash
curl -s -X POST "https://api.trello.com/1/cards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Card Title",
    "desc": "Card description",
    "idList": "LIST_ID",
    "idLabels": "LABEL_ID_1,LABEL_ID_2",
    "pos": "top"
  }'
```

### Update a card

```bash
curl -s -X PUT "https://api.trello.com/1/cards/CARD_ID?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated Title", "desc": "Updated description"}'
```

### Archive a card

```bash
curl -s -X PUT "https://api.trello.com/1/cards/CARD_ID?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"closed": true}'
```

### Move a card to a different list

```bash
curl -s -X PUT "https://api.trello.com/1/cards/CARD_ID?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"idList": "TARGET_LIST_ID"}'
```

### Add a label to a card

```bash
curl -s -X POST "https://api.trello.com/1/cards/CARD_ID/idLabels?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"value": "LABEL_ID"}'
```

### Remove a label from a card

```bash
curl -s -X DELETE "https://api.trello.com/1/cards/CARD_ID/idLabels/LABEL_ID?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN"
```

### Create a checklist on a card

```bash
curl -s -X POST "https://api.trello.com/1/checklists?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"idCard": "CARD_ID", "name": "Checklist Name"}'
```

### Add an item to a checklist

```bash
curl -s -X POST "https://api.trello.com/1/checklists/CHECKLIST_ID/checkItems?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Item text"}'
```

### Create a new label

```bash
curl -s -X POST "https://api.trello.com/1/labels?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "1 🔵 Client Work", "color": "blue", "idBoard": "'"$TRELLO_BOARD_ID"'"}'
```

### Rename a label

```bash
curl -s -X PUT "https://api.trello.com/1/labels/LABEL_ID?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "New Name"}'
```

### Archive a list

```bash
curl -s -X PUT "https://api.trello.com/1/lists/LIST_ID/closed?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"value": true}'
```

## Bulk operations (Python pattern)

For tagging or updating many cards at once, use Python with `urllib`. Read credentials from
the environment — never hardcode them.

```python
import json, os, urllib.request, urllib.error

KEY = os.environ["TRELLO_API_KEY"]
TOKEN = os.environ["TRELLO_TOKEN"]
BOARD_ID = os.environ["TRELLO_BOARD_ID"]


def add_label(card_id, label_id):
    url = f"https://api.trello.com/1/cards/{card_id}/idLabels?key={KEY}&token={TOKEN}"
    data = json.dumps({"value": label_id}).encode()
    req = urllib.request.Request(
        url, data=data, headers={"Content-Type": "application/json"}, method="POST"
    )
    try:
        urllib.request.urlopen(req)
    except urllib.error.HTTPError as e:
        # Surface failures instead of swallowing them silently.
        print(f"Failed to label {card_id}: {e.code} {e.read().decode()}")


# Get all open cards
url = f"https://api.trello.com/1/boards/{BOARD_ID}/cards?key={KEY}&token={TOKEN}&filter=open&fields=id,name,idLabels"
with urllib.request.urlopen(url) as r:
    cards = json.loads(r.read())

# Loop and tag
for card in cards:
    if "keyword" in card["name"].lower():
        add_label(card["id"], "LABEL_ID")
```

## Butler daily sort automation (optional)

Butler can sort lists by label name ascending. Because labels are prefixed with numbers,
alphabetical sort = priority order:
`1 Client Work → 2 Internal → 3 Tech/AI → 4 Recurring → 5 Delegated → 6 Digital Marketing`.

To set up or recreate it:

- Butler → **Scheduled** → **Create automation**
- Trigger: **Every day at <time>**
- Action: **Sort** → "sort the cards in list [LIST NAME] by label name ascending"
- Repeat for each list you want sorted

## Guidelines

1. When creating or categorizing a card, always apply both a **Type** label and a
   **Status** label.
2. **Never create duplicate cards** — search the board first before adding.
3. **Delegated** label = assigned to a team member; the owner is monitoring only.
4. **Waiting On** label = blocked by an external party — flag these in reviews.
5. **Board-first rule:** resolve live list/label IDs before acting; read an uploaded
   screenshot to resolve ambiguity; never ask what the board can answer.
6. **Never hardcode API keys or tokens** in this file, in committed code, or in card
   content. Always read them from environment variables, and rotate any credential that
   has been exposed.
