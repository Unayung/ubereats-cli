# ubereats-cli

Agent-native, read-only CLI for Uber Eats Taiwan. Zero dependencies (Python 3.8+ stdlib).

Uber Eats blocks curl (403) and fresh headless browsers (verification page). This CLI
instead opens one **background** tab in your own Edge on CDP `127.0.0.1:9222`, calls the
same `/_p/api/*V1` JSON endpoints the web app uses, then closes the tab. No focus stealing.

```
ubereats doctor                       # browser, API, login state
ubereats search 烏弄 [--suggest]       # find stores
ubereats store <url|uuid|b64id>       # open/ETA + full menu (alias: menu)
      [--available] [--grep 紅茶] [--mode PICKUP]
ubereats item <store> <item name|uuid> # option groups (sweetness, ice, add-ons)
ubereats raw getStoreV1 --body '{...}'
```

Every command takes `--json` → `{ok, data|error}`. Exit codes: 0 ok, 1 api, 3 auth, 4 net/blocked.

Delivery address, ETA and range come from the Edge profile's Uber Eats session.
No cart / checkout commands by design. Unofficial private API — may break anytime.
