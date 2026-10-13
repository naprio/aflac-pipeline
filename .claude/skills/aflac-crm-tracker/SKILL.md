---
name: aflac-crm-tracker
description: >
  Updates businesses in Nate's Aflac CRM Tracker (the "Aflac CRM" PWA). Use whenever Nate
  wants to change a business in the tracker — moving it along the business cycle (e.g. "move
  Gorilla Demolition to Warm", "Gold Standard is a client now"), logging another touch ("touched
  Rocky Mountain Concrete again today"), setting a follow-up date ("remind me to go back to Pioneer
  Friday"), adding a new business, or editing owner/address/notes. Also use when he debriefs a walk
  and wants the structured tracker updated (complements aflac-voice-crm, which keeps the prose log).
  Triggers: "move X to <stage>", "advance X", "log a touch on X", "touched X again", "mark X as
  dead/client", "set follow-up for X", "add X to the tracker", "update X's owner/address/phone".
---

# Aflac CRM Tracker — update skill

**Job:** read, change, and save businesses in the CRM Tracker's data file so the changes show up
in Nate's phone app. One file is the source of truth; every change is a small edit to it followed
by a commit + push.

---

## The data file

- **Repo:** `naprio/aflac-pipeline`
- **File:** `data/businesses.json`
- **Served branch:** the repo's **default branch** (`main`). The phone app's "Sync from repo" button
  fetches this file from the deployed site, so changes must land on `main` to reach the phone (unless
  Nate says otherwise).

If the repo isn't already in the working directory, add/clone it first, then operate on
`data/businesses.json`.

### Shape

```json
{
  "version": 1,
  "updatedAt": "2026-10-13T00:00:00Z",
  "businesses": [ { ...business... }, ... ]
}
```

Each business:

| Field | Meaning |
|---|---|
| `id` | stable id, never change it |
| `business` | business name |
| `dm` | owner / decision maker name |
| `address` | street address (own row in the app) |
| `zip` | ZIP code (own row in the app) |
| `phone` | phone |
| `stage` | one of the cycle stages below, or `Dead` |
| `touchCount` | number of touches (integer) |
| `initialTouch` | `YYYY-MM-DD` of the first touch — set once, never overwrite |
| `lastContact` | `YYYY-MM-DD` of the most recent touch |
| `followUp` | `YYYY-MM-DD` of the next planned follow-up (or "") |
| `partner` | who he walked with, or `Solo` |
| `nextAction` | short next step |
| `notes` | free text; append, don't clobber |
| `lat` / `lng` | map coords, usually `null` (the app geocodes) |
| `updatedAt` | ISO timestamp — bump on every change |

### The business cycle (in order)

```
Cold → Approached → Warm → Meeting Set → Presented → Enrolled → Client
```

Plus `Dead` (off-cycle: closed-lost or do-not-contact). "Advance" / "move it forward" / "move it
along" means the **next** stage in that list. Jumping straight to a named stage is fine too.

---

## How to make any change

1. **Read** `data/businesses.json`.
2. **Find the business** by name (case-insensitive, allow partial match; if more than one matches,
   ask which one). If it doesn't exist and Nate is clearly adding it, create a new entry (see below).
3. **Apply the change** (see operations).
4. **Stamp timestamps:** set that business's `updatedAt` to now (ISO), and set the top-level
   `updatedAt` to now.
5. **Write** the file back (keep it valid, pretty-printed JSON, 2-space indent).
6. **Commit & push** to the default branch:
   `git add data/businesses.json && git commit -m "<what changed>" && git push`.
7. **Confirm** in one line what changed, and remind Nate to tap **Sync from repo** in the app (or
   that it'll pull next time he opens Settings → Sync).

Keep edits minimal — change only the fields the operation touches.

---

## Operations

### Log a touch
"Touched Pioneer again today", "stopped by Karpov, owner was out".
- `touchCount` += 1
- `lastContact` = today
- if a stage change is implied (e.g. "and the owner said come back" → still Warm/Approached), apply it
- if he gives a return date, set `followUp`
- append any note to `notes` as `"<today>: <note>"` (pipe-separated from existing)

### Move along the cycle
"Move Gorilla Demolition to Warm", "advance Red Hawk", "Gold Standard is a client now".
- set `stage` to the target (next stage if he just says "advance/move forward")
- `lastContact` = today
- if moving to `Dead`, clear `followUp` and note why

### Set / change a follow-up
"Go back to Rocky Mountain Friday", "follow up with Miss Jen's on the 17th".
- set `followUp` to that date (resolve relative dates; if vague like "next week", pick a specific
  weekday and say you estimated it)
- optionally set `nextAction`

### Add a business
"Add Summit Plumbing, owner Dave, 80120, approached today".
- create an entry with a fresh `id` (`"b" + <epoch ms>`), the stage he gives (default `Approached`),
  `touchCount` 1, `initialTouch` and `lastContact` = today, `lat`/`lng` `null`, `partner` `Solo`
  unless stated. Leave unknown fields as `""`.

### Edit details
"Pioneer's owner is Linda Pho", "add the address for Color House".
- update just those fields.

---

## Rules

- **Never invent data.** Only write what Nate gives you or what's already there. Leave unknowns blank.
- **Never overwrite `initialTouch`** once set. Append to `notes`, don't replace.
- **One source of truth.** This file is it. Don't duplicate the business into another store.
- **Valid stages only.** `stage` must be one of the eight values above — don't improvise new ones.
- If Nate's debrief is a whole walk with several businesses, process each one, then give a single
  summary (e.g. "Updated 4: Karpov +1 touch, Pioneer → Meeting Set, added Summit Plumbing, Gold
  Standard → Client"). Pair with **aflac-voice-crm** if he also wants the prose approach log updated.

---

## Future: AI prospecting (not built yet)

This skill is deliberately manual for now. The planned next step is an AI layer that looks at the
tracker and suggests who to follow up with and how to keep each business moving through the cycle —
especially around prospecting. When that's added, it will read this same `data/businesses.json`, so
keep the data clean and the stages accurate now and it will pay off later.
