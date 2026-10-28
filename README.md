# ubereats-cli

Agent-native, read-only CLI for Uber Eats Taiwan. Zero dependencies (Python 3.8+ stdlib).

Portable: reads go straight to the `/_p/api/*V1` JSON endpoints over HTTPS (no browser).
Only the HTML pages are bot-walled; if an API call ever gets a 403/HTML reply it is retried
in a **background** tab of a CDP browser (`UBEREATS_CDP`, default `127.0.0.1:9222`).

## Login (mayo/nidin-style state, no password)

Uber signs in with one-time codes + bot checks, so the CLI never takes credentials.
Sign in once in any browser, then save that session:

```
ubereats login --from-browser [--cdp http://127.0.0.1:9222] [--keyring]
ubereats login --cookie-stdin [--keyring]   # paste DevTools "Cookie:" header
ubereats whoami | logout
```

The session is checked with `getUserV1` before saving. Stored in
`~/.config/ubereats/state.json` (0600, `UBEREATS_HOME` to move it), or the OS keyring with
`--keyring` (macOS `security`, Linux `secret-tool`). `UBEREATS_COOKIE` overrides both.
An expired `jwt-session` gives a warning and exit code 3.

## Address

```
ubereats address from-url '<store URL containing pl=...>'
ubereats address set '民生東路三段128號' --lat 25.0576 --lng 121.5460
```
Sent as the `uev2.loc` cookie; changes ETA, delivery range and search ranking.

```
ubereats doctor [--via auto|http|browser]
ubereats doctor                       # browser, API, login state
ubereats search 烏弄 [--suggest]       # find stores
ubereats store <url|uuid|b64id>       # open/ETA + full menu (alias: menu)
      [--available] [--grep 紅茶] [--mode PICKUP]
ubereats item <store> <item name|uuid> # option groups (sweetness, ice, add-ons)
ubereats raw getStoreV1 --body '{...}'
```

Every command takes `--json` → `{ok, data|error}`. Exit codes: 0 ok, 1 api, 3 auth, 4 net/blocked.

No cart / checkout commands by design. Unofficial private API — may break anytime.
