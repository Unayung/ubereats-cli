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
An expired `jwt-session` prints a warning; calls that need login then fail with exit code 3.

## Address

```
ubereats address from-url '<store URL containing pl=...>'
ubereats address set '台北市松山區民生東路三段128號'   # looked up via Uber Eats; prints the match
ubereats address set '民生東路三段128號' --lat 25.0576 --lng 121.5460
```
Sent as the `uev2.loc` cookie; changes ETA, delivery range and search ranking.

```
ubereats doctor [--via auto|http|browser]   # browser, API, login state
ubereats search 烏弄 [--suggest]       # find stores
ubereats store <url|uuid|b64id>       # open/ETA + full menu (alias: menu)
      [--available] [--grep 紅茶] [--mode PICKUP]
ubereats item <store> <item name|uuid> # option groups (sweetness, ice, add-ons)
ubereats raw getStoreV1 --body '{...}'    # get* endpoints only
```

## Group orders (團購)

Someone shares `https://eats.uber.com/group-orders/<uuid>/join`:

```
ubereats group list                    # group orders you are in, open or why not
ubereats group peek <link>             # store, deliver-to, who pays, joined?; members + items once joined
ubereats group join <link> --yes       # [--nickname NAME]
ubereats group add <link> 一品紅茶 --opt 熱 --opt 小 --opt 微糖 [--opt 椰果] [--qty 2] [--note ..] --yes
ubereats group rm <link> 一品紅茶 --yes
ubereats group leave <link> --yes
```

Without `--yes`, join/leave/add/rm only print what they would do. `--opt` matches an option title
exactly if that title exists anywhere in the item, else by substring; nested options
(size under temperature) work; unset required groups take the store's default. `add` refuses
groups that split the bill (`you_pay`) unless `--i-pay`. `peek` reads `getJoinGroupOrderMetaV1`,
which also answers for closed or discarded groups: whether a group is still open shows only
after joining (`joined, open`; else `LOCKED`, `EXPIRED` or the order state). state stays
`UNORDERED` after the host locks the cart, so add/rm check the lock and expiry too. Uber Eats
answers `leave` with success on a locked group yet keeps you in; `leave` re-checks and says so.

Every command takes `--json` → `{ok, data|error}`. Exit codes: 0 ok, 1 api, 3 auth, 4 net/blocked.

No checkout by design: the only writes are joining a group order and adding/removing your own
items in it. The group host places and pays for the order. Unofficial private API — may break anytime.
