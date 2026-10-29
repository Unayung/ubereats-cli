---
name: ubereats
description: Use when checking Uber Eats Taiwan stores, menus, prices, or item options, or joining an Uber Eats group order (團購, eats.uber.com/group-orders/... link) and adding items to it.
---

# ubereats — Uber Eats Taiwan from a shell (no checkout)

`~/.local/bin/ubereats` → `~/Projects/ubereats-cli/ubereats`. Portable stdlib Python:
reads go over plain HTTPS; a CDP browser (Edge 9222 here) is only a fallback and the
source for `login --from-browser`. Run `ubereats doctor` first on a new machine.

- `ubereats address set '<地址>'` (or `address from-url '<store url with pl=>'`) once, so
  ETA/range match the user; check the printed `matched:` line is the right place
- `ubereats login --from-browser` after the user signs in themselves; never ask for codes

- `ubereats store <url> --json` — `is_open`, `eta`, `items[]` (title, price TWD, sold_out, has_options, section)
- `ubereats item <url> <name>` — required/optional option groups with prices
- `ubereats search <q>` — stores near the saved `address` (else IP guess)
- `--via http|browser` forces a transport; default HTTP with CDP fallback on 403/HTML
- Session: `~/.config/ubereats/state.json` (0600) or `--keyring`; `UBEREATS_COOKIE` overrides; `whoami` / `logout`
- Store URL last segment is the store UUID in url-safe base64.

Group orders (link `eats.uber.com/group-orders/<uuid>/join`, often posted in Slack):
1. `ubereats group peek <link> --json` — store, `you_pay`, `joined`; once joined also `state`, members, items
2. `ubereats store <store_uuid> --grep ..` / `ubereats item <store_uuid> <name>` to pick options
3. Show the user the store, item, options and price; only after they say yes:
   `ubereats group join <link> --yes`, then `ubereats group add <link> <item> --opt .. --yes`
4. `group rm <link> <item> --yes` undoes an add. Never pass `--i-pay` unless the user asked to pay.

Pitfalls: without a saved session getUserV1 / draft orders fail (exit 3). Without an
address, stores may say 目前不接受訂單 because Uber Eats guesses the location from IP.
Never add ordering without explicit user confirmation — checkout spends real money.
Pomerado (headless) is blocked by Uber Eats' verification page; don't retry it.
