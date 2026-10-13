# Aflac CRM Tracker

A simple, installable (PWA) field CRM for Bedrock Benefits — track businesses through the
prospecting-to-client cycle, log touches, and see your territory on a map.

## What's here

| File | What it is |
|---|---|
| `index.html` | The app (React via CDN, no build step). List / Map / Settings tabs. |
| `data/businesses.json` | **The source of truth for your businesses.** The skill edits this; the app seeds + syncs from it. |
| `manifest.json`, `sw.js`, `icon-*.png` | PWA wrapper so it installs to your phone and works offline. |
| `.claude/skills/aflac-crm-tracker/` | The skill Claude uses to update businesses in the tracker. |

## The business cycle

```
Cold → Approached → Warm → Meeting Set → Presented → Enrolled → Client     (+ Dead = closed/off-cycle)
```

## Each business tracks

Business name · owner / decision maker · address · ZIP (its own row) · phone · stage ·
touch number · initial touch date · last contact · follow-up date · partner · next action · notes.

## Using it

- **In the app (phone):** add/edit a business, tap **+1 Touch**, **Advance →** to move it along the
  cycle, or set a follow-up date. Overdue follow-ups pulse red.
- **With Claude (the skill):** just say what happened — "move Gorilla Demolition to Warm", "touched
  Pioneer again today", "go back to Rocky Mountain Friday", "add Summit Plumbing". The skill updates
  `data/businesses.json` and pushes it.

### Keeping phone ⇄ repo in sync

Your phone holds the live copy. The skill edits the repo file. To reconcile:

- **Pull skill updates onto the phone:** Settings → **Sync from repo** (newest edit per business wins).
- **Send phone edits back:** Settings → **Export** (copies + downloads the JSON), then hand it to Claude.

## Map setup (Google Maps)

1. In Google Cloud, create an API key with **Maps JavaScript API** and **Geocoding API** enabled.
2. In the app: Settings → paste the key → **Save key**.
3. Open the **Map** tab. Businesses are geocoded from address (or ZIP center) and pinned, colored by
   stage. Coordinates are cached so geocoding only runs once per business.

The key is stored only on your device. Restrict it to your app's domain in Google Cloud.

## PIN lock

The app is behind a 4-digit PIN (default `5280`) so field data isn't open on an unlocked phone.
Change `CORRECT_PIN` near the top of the `<script>` in `index.html`.

## Coming later: AI prospecting

Intentionally not built yet. The plan is an AI layer that reads this same `data/businesses.json` and
suggests who to follow up with and how to keep each business moving — focused on prospecting. Keeping
stages and dates accurate now makes that drop-in later.
