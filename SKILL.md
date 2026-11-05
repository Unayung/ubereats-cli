---
name: ubereats
description: Use when checking Uber Eats Taiwan stores, menus, prices, or item options, starting (開團) or cancelling an Uber Eats group order, or joining one (團購, eats.uber.com/group-orders/... link) and adding items to it.
---

# ubereats — Uber Eats Taiwan from a shell (no checkout)

`~/.local/bin/ubereats` → `~/Projects/ubereats-cli/ubereats`. Portable stdlib Python:
reads go over plain HTTPS; a CDP browser (Edge 9222 here) is only a fallback and the
source for `login --from-browser`. Run `ubereats doctor` first on a new machine.

- `ubereats address set '<地址>'` (or `address from-url '<store url with pl=>'`) once, so
  ETA/range match the user; check the printed `matched:` line is the right place
- `ubereats login --from-browser` after the user signs in themselves; never ask for codes
- Without a CDP browser, prefer `ubereats login --from-clipboard` (user copies the DevTools
  `Cookie` request-header value, then runs it without copying anything else). `--cookie-stdin`
  reads a raw tty because the Linux line editor caps a line at 4 KB and Uber cookies are
  5–8 KB (Enter does nothing). The CLI rejects clipboard text that is not a cookie header
  (e.g. a copied shell prompt with Powerline glyphs). Tell the user not to paste cookies
  into chat.

- `ubereats store <url> --json` — `is_open`, `eta`, `items[]` (title, price TWD, sold_out, has_options, section)
- `ubereats item <url> <name>` — required/optional option groups with prices
- `ubereats search <q>` — stores near the saved `address` (else IP guess)
- `--via http|browser` forces a transport; default HTTP with CDP fallback on 403/HTML
- Session: `~/.config/ubereats/state.json` (0600) or `--keyring`; `UBEREATS_COOKIE` overrides; `whoami` / `logout`. Never copy a saved session between machines: treat it as a login credential.
- Store URL last segment is the store UUID in url-safe base64.

Group orders (link `eats.uber.com/group-orders/<uuid>/join`, often posted in Slack):
1. `ubereats group peek <link> --json` — store, `you_pay`, `joined`; once joined also `closed`
   (null = open), members, items. Non-members cannot see if it is open. `group list` = groups you are in
2. `ubereats store <store_uuid> --grep ..` / `ubereats item <store_uuid> <name>` to pick options
3. Show the user the store, item, options and price; only after they say yes:
   `ubereats group join <link> --yes`, then `ubereats group add <link> <item> --opt .. --yes`
4. `group rm <link> <item> --yes` undoes an add; `group leave <link> --yes` leaves (ask first). Never pass `--i-pay` unless the user asked to pay.

Hosting (開團) when the user asks to start a group order:
1. Pick the store with the user (`search`, `store`); confirm name, deadline, who pays
2. `ubereats group create <store> [--deadline 12:30] [--split] [--limit 200]` previews; after the
   user says yes add `--yes` and give them the printed `link` to share. Default: they pay for everyone
3. `group peek <link>` shows who joined and what they added; `group cancel <link> --yes` (ask first)
   drops the group. Placing the order happens in the Uber Eats app; the CLI never checks out

Pitfalls: without a saved session getUserV1 / draft orders fail (exit 3). Without an
address, stores may say 目前不接受訂單 because Uber Eats guesses the location from IP.
Never add ordering without explicit user confirmation — checkout spends real money.
Pomerado (headless Chromium) is blocked by Uber Eats' verification page; don't retry it.

## Lightpanda fallback experiment (2026-10-29)

`lightpanda` 1.0.2 is installed at `~/.local/bin/lightpanda`; M5 has its arm64 build too. It is a workable read-only browser fallback:

- Its page-context `fetch()` successfully called `getStoreV1`, `getMenuItemV1`, `getSearchSuggestionsV1`, and `getSearchFeedV1` (all HTTP 200).
- For a signed-in Lightpanda fallback on the same machine, read the local state cookie and inject its individual name/value pairs into a fresh CDP context with `Network.setCookies`, then verify with the read-only `getUserV1`. This also reaches `getDraftOrdersByEaterUuidV1`; treat returned drafts as read-only unless the user explicitly asks to change a cart. Do not print or transfer cookie values.
- It self-identifies as `Lightpanda/1.0`. It deliberately refuses `Mozilla` User-Agent impersonation; do not try to bypass that guard. A site may later reject this UA.
- Default telemetry posts aggregate usage to `telemetry.lightpanda.io`; always launch it with `LIGHTPANDA_DISABLE_TELEMETRY=true`. It reports an install ID, OS/arch/version, command/mode and aggregate tool/navigation events, not page URLs/content.
- `lpnet` at `~/.local/bin/lpnet` records CDP Network events and can save XHR/fetch response bodies. Start Lightpanda with `LIGHTPANDA_DISABLE_TELEMETRY=true lightpanda serve --host 127.0.0.1 --port 9333`, then use `lpnet <url> --filter /_p/api --bodies`.
