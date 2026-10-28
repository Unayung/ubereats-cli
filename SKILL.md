---
name: ubereats
description: Use when checking Uber Eats Taiwan stores, menus, prices, or item options.
---

# ubereats — Uber Eats Taiwan from a shell (read-only)

`~/.local/bin/ubereats` → `~/Projects/ubereats-cli/ubereats`. Needs Edge on CDP 9222
(see memory for launch flags). Opens a background tab only; never focuses windows.

- `ubereats store <url> --json` — `is_open`, `eta`, `items[]` (title, price TWD, sold_out, has_options, section)
- `ubereats item <url> <name>` — required/optional option groups with prices
- `ubereats search <q>` — stores near the Edge session's saved address
- Store URL last segment is the store UUID in url-safe base64.

Pitfalls: Edge is not signed in to Uber Eats, so getUserV1 / draft orders fail.
Never add ordering without explicit user confirmation — checkout spends real money.
Pomerado (headless) is blocked by Uber Eats' verification page; don't retry it.
