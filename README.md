# Tzero Bot

**Versi:** 1.0.7 · **Harga:** Rp 120.000 (~$8) · **Kontak pembelian:** [@rmndkyl](https://t.me/rmndkyl)

Automation for **Tzero**, the launchpad for **Zero** (LayerZero's L1), running on **Base Sepolia** (chain 84532). Season 0 **Pre-flight Points** is live now; Zero mainnet is expected Q4 2026, so every action below is early positioning on free testnet ETH.

## Kebutuhan

| Item | Detail |
|---|---|
| Runtime | Node.js 18+ |
| Wallet | EVM / sesuai chain proyek (kamu sendiri) |
| Proxy | opsional (1 proxy per akun) |
| Captcha | tidak perlu |
| Akses | akun proyek yang kamu kelola sendiri |

## Pembayaran

E-Wallet: DANA / GoPay / ShopeePay · Bank: BCA / Seabank · Crypto: contact for wallet address.
Pembeli menghubungi [@rmndkyl](https://t.me/rmndkyl), kirim bukti transfer, lalu terima license key.

---

Automation for **Tzero**, the launchpad for **Zero** (LayerZero's L1), running on **Base Sepolia**
(chain 84532). Season 0 **Pre-flight Points** is live now; Zero mainnet is expected Q4 2026, so every
action below is early positioning on free testnet ETH.

Built by [LYID — Layer Airdrop ID](https://t.me/layerairdrop) · Telegram [@rmndkyl](https://t.me/rmndkyl)

---

## Proof it runs

Every screenshot below is captured from this bot's own log files and live reads — not a mockup.

**Autonomous crew snipe.** A mission sitting at 98.64% is finished in one buy, and the liftoff is
confirmed inside the same transaction:

![Crew sniper run](docs/screenshots/crew-sniper.png)

**Official points, straight from the project's own API.** The local estimator is only a fallback;
eligibility is decided by these numbers:

![Site points](docs/screenshots/site-points.png)

**Capital recovery.** Every still-held position quoted on its correct venue (curve before liftoff,
TzeroSwap after), with nothing sold inside the 60 minute hold window:

![Holdings](docs/screenshots/holdings.png)

**On-chain receipt of the snipe above**, checked over RPC:

![On-chain proof](docs/screenshots/onchain-proof.png)

---

## The point rules decide the whole design

Copied from https://testnet.tzer0.fun/docs/points (read 2026-11-08) and enforced in `src/points.js`:

| Source | Points | Rule that actually bites |
|---|---|---|
| **Fuel** | 1,000 per 1 ETH bought | Must be **held 60 minutes**. Anything sold or sent inside 60 min does not count, and a sale is matched against your **most recent buys first**. Buys under **0.0005 ETH earn zero**. Max **1 ETH of buys per wallet per UTC day**. Buys on your own mission earn nothing. |
| **Crew** | +150, max 5/day | A mission you backed must **lift off**; you need **≥ 0.01 ETH bought on its curve**, held 60 min, still holding at liftoff. |
| **Liftoff** | +500 | Creator of a mission that lifts off with **≥ 5 other wallets** backing it (0.01 ETH+ each, held 60 min). Launching alone earns nothing. |
| **Staking** | 5,000/day shared | Pro-rata, capped at 1,000/day/wallet, denominator at least 10,000,000 T0. |
| **Signal** | +200 / +100 | Reply or quote the announcement post on X with your wallet code, paste the link, sign a message. Manual — the bot does not touch X. |
| **Eligibility** | 500 | Below 500 points a wallet gets nothing. |

Two consequences drove this version:

1. **Selling early destroys points.** The old daily loop bought 0.00025-0.00065 ETH and sold 40% seconds
   later - below the Fuel floor *and* inside the hold window. It earned zero, every run. Buys are now
   written to a ledger and only sold by a separate `--settle` step, 65 minutes later.
2. **Fuel is the expensive way to 500 points.** 500 Fuel points needs 0.5 ETH of held buys per wallet.
   Crew is 12x cheaper per point, so the bot's budget logic leans on Crew and Liftoff.

Testnet calibration measured on 2026-11-08: a fresh curve lifts off after **~0.28 ETH** of net buys
(`quoteBuy(1 ETH).refund` gives the exact remaining cost for any mission).

## What the bot does

| Mode | Command | What it does |
|---|---|---|
| Read + status | `--status` | tokenCount, creation fee, newest open missions |
| Quote | menu 2 | buy/sell estimate, no gas |
| Fuel run | `--daily` | Buy >=0.0006 ETH on 2-4 missions and **hold**; optional mission launch; claim creator fees |
| Settle | `--settle` | Sells only positions that already passed the 60 min hold |
| Crew sniper | `--crew` / `--crew-scan` | Finds missions where one buy finishes the curve, buys the last slice (3M gas) -> +150 Crew. Scans up to 3,000 recent missions through Multicall3 (~13 s for 2,000), so a mission that sits at 88% for hours is still visible. Since 1.0.7 the wallet priority reads the **site** numbers from `data/site-points.json` (local estimate is wrong in both directions) and skips wallets already over 500 |
| Crew loop | `--crew-loop` | One process instead of restarting by hand: scan -> snipe -> wait -> scan again, rotating wallets, until every wallet hits its 5/day Crew cap (or `--loop-runs N`). Wallets already at the cap are dropped from the pool, so a run never dies on a wallet that cannot earn anymore |
| Crew train | `--crew-train` | 5 wallets each back the same near-liftoff mission (>=0.01 ETH) -> 5 x +150 if it lifts |
| Group buy | `--gb --create` / `--back` / `--fill` / `--gb --gb-status` | 1 creator + 5 backers on our own mission -> +500 Liftoff and +150 per backer |
| Points | `--points` | Local estimate per wallet (fuel, crew, liftoff) + live site check |
| Site points | `scripts/site-points.js` | **Authoritative**: pulls `/api/points/<addr>` for every wallet into `data/site-points.json` (total, fuel, crew, liftoff, staking, signal, eligible, rank, per-action list). 5 wallets in parallel, 4 tries each, and a wallet the site refuses does not block the rest. `CEK-POIN.bat` now runs this instead of the local estimate, and the crew sniper reuses the cache for wallet priority |
| Sweep | `--sweep` | Consolidates dust into N wallets so a 0.01 ETH Crew buy is possible |
| Staking | `--stake`, `--stake --buy <eth>`, `--stake --claim-rewards`, `--stake --unstake`, `--stake --stake-report` | $T0 stake/unstake/claim with an honest yield readout |
| Launch / claim | `--launch`, `--claim` | Create a mission, claim creator fees |
| Helper | `scripts/balance.js` | Balance snapshot of all 21 wallets (source + fleet) |
| Helper | `scripts/holdings.js` | Quotes every still-held token on its correct venue -> recoverable ETH |
| Helper | `scripts/fuel-topup.js <addr> <eth>` | Buys an exact amount on a random non-own mission and records it (used to push a wallet over 500) |
| Helper | `scripts/send.js <from> <to> <eth>` | Manual one-off transfer. Kept for rescues only: every transfer between your own wallets is one more on-chain link |
| Helper | `scripts/sepolia.js` | Relay bridge Sepolia -> Base Sepolia (`info` / `quote` / `bridge --execute`) |
| Helper | `scripts/eligibility-catchup.js` | Buys just enough Fuel to push a 460-499 point wallet over 500 (max 1 per run) |
| Helper | `scripts/check-position.js <SYMBOL>` | Age + sell quote + pool reserves for one held position |
| Helper | `scripts/exit-position.js <SYMBOL> [pct]` | Sells a symbol now, bypassing the per-persona settle window (use when the pool price spikes) |

Everything is wrapped in per-wallet try/catch, a balance guard, randomized delays and a slippage cap.

## Features

| Feature | Status |
|---|---|
| Read layer (stats, missions, quotes, remaining-to-liftoff) | working |
| Buy + HOLD with a per-buy ledger (`data/positions.json`) | working, live-tested |
| Settle that refuses immature positions | working, live-tested |
| Daily Fuel cap (1 ETH/day/wallet) | working |
| Crew sniper (buy the last slice, 3M gas) | working, live-tested; now prefers wallets still under 500 points |
| Crew loop (one process until every wallet is at 5/5) | working, dry-run tested 2026-11-13; capped wallets are skipped instead of ending the run |
| Crew train (5 wallets on one mission) | implemented, needs 5 funded wallets |
| Group-buy state machine with 65 min hold | implemented, needs ~0.29 ETH |
| Points estimator + site API probe | working; the site went live 2026-11-10, so `scripts/site-points.js` is now the authoritative view |
| Staking + honest yield math | working, yield is negligible on testnet |
| Dust sweep into N wallets | working, executed 14 transfers |
| Multi-wallet, `--limit N`, `--dry-run`, license gate | working |

## Requirements

| Item | Required? | Cost |
|---|---|---|
| Node.js 18+ (tested on v26) | yes | free |
| Base Sepolia test ETH | yes | free (faucet) |
| One private key per wallet | yes | free |
| Real money | **no** | Rp 0 |

Faucets that worked in this project: `console.optimism.io/faucet` (GitHub login, ~0.1 ETH per claim).
The CDP Coinbase faucet works with an account. Most other Base Sepolia faucets were dead or gated.

## Point economics, honestly

Measured on 2026-11-08 with 0.09 ETH of testnet ETH in the fleet:

| Play | Cost | Points | Notes |
|---|---|---|---|
| Fuel | 0.001 ETH | 1 | 500 points = 0.5 ETH held per wallet. Not affordable at fleet scale. |
| Crew sniper | 0.012-0.05 ETH | 150 | Only when a mission is already near liftoff. Windows last minutes. |
| Crew train | ~0.0525 ETH | 750 (5 x 150) | Best points per ETH, needs 5 wallets holding >=0.011 ETH each. |
| Group buy (own mission) | ~0.286 ETH | 1,250 (500 + 5 x 150) | Guaranteed liftoff, most expensive. |
| Staking | ~213 ETH for 1M T0 | ~5/day | Dead on testnet: 935M T0 already staked. |

Realistic path: **crew train whenever a near-liftoff mission exists, one group-buy cycle when the fleet
can afford it, and never sell inside the hold window.**

## Setup

Cara termudah di Windows: klik dua kali **JALANKAN.bat** (menu berbahasa Indonesia), atau
**CEK-POIN.bat** untuk melihat poin saja. Panduan klik-per-klik ada di `PANDUAN-MANUAL.md`.

Lewat terminal:

```bash
cd tzero-bot
npm install
cp config.json.example config.json
node index.js --status
node index.js --daily --dry-run --limit 1
node index.js --daily
node index.js --points
```

Interactive menu (`node index.js`) exposes 18 actions (`0` exits), including settle, crew, crew loop,
group buy, staking and a proxy checker.

## Usage (cron-friendly)

```bash
node index.js --daily                 # fuel run, buys and holds
node index.js --settle                # sell only positions older than 65 min
node index.js --crew-scan             # is anything about to lift off?
node index.js --crew --limit 1        # snipe one
node index.js --crew-loop             # keep sniping until every wallet is 5/5 (Ctrl+C to stop)
node index.js --crew-loop --loop-interval 900 --loop-runs 5   # bounded: 5 passes, 15 min apart
node index.js --crew-train            # 5 wallets on one near-liftoff mission
node index.js --gb --create           # start a cycle (needs ~0.29 ETH)
node index.js --gb --back             # 5 backers buy in
node index.js --gb --fill             # finish the curve after the hold window
node index.js --points                # where do we stand?
node index.js --sweep --limit 6       # consolidate dust
```

Suggested schedule: `--daily` once a day, `--settle` two hours later, `--crew-scan` every 15 minutes
(windows are short), `--gb` whenever a cycle is open.

commands do not: a **random 0-4 minute delay** so the runs are not metronomic, and a **lock file**
(`data/crew.lock`) so two runs never overlap. It runs `--crew`, then `eligibility-catchup.js`, and it
does **not** fire `--crew-train` automatically: five wallets buying the same mission within minutes is
the most scripted-looking thing this bot can do, so the train is manual only.

Cron jobs in use:

| Job | Schedule | Script |
|---|---|---|
| `tzero-crew-sniper` | every 15m (delayed 0-4m) | `tzero-crew.py` |
| `tzero-fuel-ladder` | every day 09:30 WIB | `tzero-ladder.py` |
| `tzero-daily` | every day 10:00 WIB (delayed 0-18m) | `tzero-daily.py` |
| `tzero-gb3-driver` | every 10m (VOID706 only) | `tzero-gb3.py` |
| `tzero-settle-daily` | every day 21:30 WIB | `tzero-settle.py` |
| `tzero-settle-once` | one-shot | `tzero-settle.py` |
| `tzero-gb-fill` | every 5m, **paused** | `tzero-gb-fill.py` |

Windows Task Scheduler:

```
```

## Wallet fleet

| Where | What |
|---|---|
| `C:/mek/jkjke/Tzero Testnet Airdrop.txt` | legacy vault: label / seedphrase / private key / address |

After the 2026-11-08 sweep the fleet is 6 funded wallets (0.011-0.024 ETH each) and 15 dust wallets.
That is deliberate: Crew needs 0.01 ETH from a single wallet, so 21 wallets at 0.004 could never act.

Since 2026-11-09 every farming wallet holds at least 0.017 ETH (17 top-ups, 0.2968 ETH total), and the
two wallet closest to the 500 bar (0x7de5, 0xbc50) hold 0.034 each so they can take any snipe (max 0.03).

Optional, but this is the answer if you want to run a fleet from one internet connection. One proxy per

```
http://user:pass@proxy1.example.com:8080     # line 1 -> wallet 1
socks5://user:pass@proxy2.example.com:1080   # line 2 -> wallet 2
                                             # empty line -> that wallet goes direct
http://proxy3.example.com:3128               # line 4 -> wallet 4
```

Formats accepted (the same ones the rest of the LYID pipeline uses):

| Line format | Note |
|---|---|
| `http://user:pass@host:port` | most paid proxies |
| `http://host:port` | no auth |
| `https://user:pass@host:port` | TLS to the proxy |
| `socks5://user:pass@host:port` | `socks4://` and `socks5h://` also work |
| `host:port` | short form, treated as http |
| `host:port:user:pass` | provider export format |
| `http:host:port` | free-list format (Monosans) |
| empty line | that wallet connects direct |

Each wallet gets its own proxy **inside one process**: the agent is attached to that wallet's provider
through `ethers.FetchRequest.createGetUrlFunc({ agent })`, so JSON-RPC and the points API both leave
from the wallet's own IP. No child processes, no global dispatcher, wallets cannot leak into each
other's proxy.

Rules that matter:

- **A dead proxy skips that wallet.** It is never re-routed to a direct connection, otherwise your real
  IP would show up for exactly the wallets you are trying to separate.
- **Fewer lines than wallets** means some wallets share an IP. The bot warns at load time.

`3` direct. Flags: `--proxy` (refuse to run without proxies), `--no-proxy` (force direct),
`--proxy-check` (test every line).

Test before farming:

```
CEK-PROXY.bat                      # double-click on Windows
node index.js --proxy-check        # same thing from a terminal
node scripts/proxy-check.js 5      # only the first 5 lines
```

It sends `eth_chainId` through each proxy and prints the egress IP, so you can see at a glance which
lines are alive and whether two wallets are sharing one IP.

Verified on 2026-11-10 with a local HTTP CONNECT proxy and a local SOCKS5 proxy with user/pass: RPC
calls, the points API, per-wallet routing, and the dead-proxy skip all behave as described above.

### Funding: Sepolia -> Base Sepolia (Relay)

`scripts/sepolia.js` bridges native ETH through the **testnet** Relay API
(`https://api.testnets.relay.link` — the main `api.relay.link` does not list Sepolia).

```bash
node scripts/sepolia.js info                                   # bridge wallet + Sepolia balance
node scripts/sepolia.js quote 0.5 0x7490...A01                 # fee + expected output, no tx
node scripts/sepolia.js bridge 0.5 0x7490...A01 --execute      # sign + send the deposit tx
# repeat the bridge for each wallet, or use the faucet per wallet
```

`recipient`, so a bridge can land straight on the wallet you want to top up. Bridging one wallet and
fanning it out locally is cheaper, but it creates the star graph described below - the bot no longer
offers that shortcut.

## Anti-detection, and the sybil review

The point docs say it plainly: before the snapshot a **public sybil review removes farm clusters -
groups of wallets funded from the same source that act together - and scripted patterns.**

### Persona per wallet (`src/humanize.js`)

Uniform randomness is still a fingerprint: if every wallet draws from the same range, two wallets are
statistically identical. Each wallet now gets a **stable persona derived from its address** (SHA-256 ->
seeded PRNG), so it keeps its own habits every run and no two wallets behave alike:

| Knob | Range | What it changes |
|---|---|---|
| `sizeBias` | 0.80-1.32 | buy size relative to the config band |
| `missions` | 2-4 | missions traded per run |
| `sellMin`/`sellMax` | 20-70% | how much of a position it exits |
| `slipMinBps`/`slipMaxBps` | 140-560 | slippage tolerance |
| `restDayProb` | 4-14% | chance of staying quiet |
| `launchProb` | 35-80% | chance of launching a mission |
| `tempoMult` | 0.55-1.9 | pause length between actions |
| `crewTicketEth` | 0.0102-0.0165 | slice size when backing a liftoff |
| `gasPremium` | 1.00-1.22 | gas price multiplier |
| `settleAfterMin` | 72-180 | when this wallet exits a mature position |

`node index.js --personas` prints the whole table for inspection.

On top of personas:

- **Non-round amounts.** Every size is nudged +-1.5% and written with 6 decimals, so nothing lands on
  0.0005 / 0.0105 style numbers.
- **Log-normal tempo.** Pauses cluster short with occasional long ones, clamped per run, instead of a
  flat `randomInt(min,max)`.
- **Exit stagger.** `--settle` uses each wallet's `settleAfterMin`, so the fleet does not sell at the
  same minute (disable with `humanize.staggerSettle: false`).
- **Gas premium per wallet.** The node's price is nudged by that wallet's persona instead of every tx
  paying the identical amount.
- **RPC rotation.** Wallets are spread over `humanize.rpcPool` (default: sepolia.base.org,
  base-sepolia.publicnode.com, 84532.rpc.thirdweb.com) so one provider does not see the whole fleet.
- **Fill delay.** A group-buy cycle decides its own fill offset at creation (66-150 min) instead of
  every cycle filling exactly 65 minutes after the last backer.
- **Backer tickets and order.** Backers buy slightly different sizes in a shuffled order, not 5 x
  0.0105 in file order.

What that still does not fix: the funding graph, wallet age, and the fact that several wallets
backing the same brand-new mission is coordination by construction. Persona changes texture, not
topology.

What it cannot hide:

- **A star funding graph, if you create one.** The bot no longer funds wallets from a single source
  (1.0.3), but if every wallet is topped up from the same exchange account or the same faucet identity,
  the graph is still visible on-chain. Only you can avoid that.
- **Same RPC provider** for every wallet (one IP on the provider side).
- **Fresh wallets created the same day**, first activity clustered.
- **A crew train is by definition "wallets acting together".** Five buys of ~0.0105 ETH on one mission
  within minutes is the exact pattern the review describes.

The only real mitigation is funding independence: each wallet claimed from a faucet (or withdrawn from
an exchange) on its own identity, spread over days, with different behaviour. The bot can no longer
move ETH between wallets at all, so nothing it does re-links them.

### Wallet independence (1.0.3)

to another.

Why: one wallet paying gas to twenty wallets is a star graph. Whoever reviews the snapshot sees a single
address with twenty outbound transfers and can drop the whole cluster, no matter how random the amounts
or the delays look. This is the same pattern the project's published sybil review describes.

How to fund a fleet now:

| Option | Notes |
|---|---|
| Faucet per wallet | cleanest: different days, own identity, and its own proxy line if you have one |
| Testnet bridge straight to each wallet | `node scripts/sepolia.js bridge <eth> <wallet> --execute`; Relay accepts any recipient |
| Exchange/faucet withdrawal per address | one withdrawal each, slightly different amounts and days |

`--sweep` still exists because Crew needs one wallet holding 0.01 ETH, but it is manual, it is never part
of `--daily`, and it does link the dust wallets to the targets on-chain. If cluster hygiene matters more
to you than Crew, skip it and fund a single wallet properly instead.

## Configuration (`config.json`)

| Key | Default | Meaning |
|---|---|---|
| `daily.minBalanceEth` | 0.0012 | skip wallets below this |
| `daily.buyAmountMinEth` / `MaxEth` | 0.0006 / 0.0012 | Fuel buy size, floor is the 0.0005 rule |
| `daily.holdMinutes` | 65 | how long before `--settle` may sell |
| `daily.fuelDailyCapEth` | 1 | per-wallet daily Fuel cap |
| `daily.sellPercentMin` / `Max` | 25 / 65 | settlement sell size |
| `daily.createMission` / `launchProbability` | true / 0.55 | own mission launch |
| `daily.minHoursBetweenRuns` | 20 | resume guard for batches |
| `crew.scanMissions` | 2000 | missions scanned for near-liftoff (read via Multicall3, max 3000; ~13 s) |
| `crew.minProgressBps` | 8500 | only scan missions above 85% |
| `crew.maxFillEth` | 0.03 | max ETH a snipe may cost |
| `crew.minBuyEth` | 0.0103 | Crew needs 0.01, keep a buffer |
| `crew.maxPerRun` / `dailyCap` | 1 / 5 | snipes per run / Crew awards per day |
| `crew.loopIntervalMs` | 900000 | `--crew-loop` wait between passes (15 min), `--loop-interval <sec>` overrides |
| `crew.loopMaxRuns` | 0 | `--crew-loop` pass limit, 0 = run until every wallet is at `dailyCap` |
| `crew.skipEligible` | true | drop wallets already >=500 from the snipe pool, so the gas goes to wallets still short of the bar (falls back to eligible wallets when no unfinished one can pay) |
| `crew.siteCacheMaxHours` | 12 | how old `data/site-points.json` may be before the sniper ignores it and uses the local estimate |
| `crew.trainSize` / `trainAmountEth` | 5 / 0.0105 | crew train shape |
| `gb.backers` / `backAmountEth` | 5 / 0.0105 | group buy shape |
| `gb.initialBuyEth` | 0.012 | creator's opening buy |
| `gb.holdMinutes` | 65 | wait after the last backer before filling |
| `sweep.targets` | 6 | wallets dust is consolidated into by the **manual** `--sweep` only |

## Verification status (honest)

| Module | Status | Evidence |
|---|---|---|
| Read layer | **verified live** | 3,746 missions, creation fee 0.0001 ETH, curve supply 793.1M |
| Settle refuses immature | **verified live** | same position: `settle actions: 0, skippedImmature: 1`; after simulating 70 min it sells (dry) |
| Fuel cap enforcement | **verified** (code path + ledger math) | `capRemaining` uses the same ledger the settle test wrote |
| Sweep | **verified live** | 14 transfers, all status 1, 6 targets funded 0.0112-0.0241 ETH |
| `fillQuote` / remaining-to-liftoff | **verified live** | 0.286 ETH on a fresh mission, 0.0999 on an 87% mission after sells |
| Crew sniper | **verified live** | 4 snipes landed autonomously: ONDO #4382 (tx `0xe058fe34...`), BTC #4368 (tx `0x9aec8753...`), plus T0EX #3909 and ABB #3948. Each liftoff confirmed in the same tx -> +150 Crew |
| Crew sniper scan depth | **verified by measurement** | 2,000 missions through Multicall3 in ~13 s; a depth probe found 691 missions >=85%, and 3 live candidates were outside the old 250-mission window |
| Group buy create → back → fill | **verified live, full cycle** | CLUB67 `0x2Ae54Da...f345e` graduated at block 56,138,569; create `0xfecd10c2...`, 5 backer txs, fill `0x7416ffd1...` |
| Crew qualification (60 min hold) | **verified from chain timestamps** | 5/5 backers held 66.2–70.5 min at liftoff, 0.0105 ETH each |
| Sell after liftoff (TzeroSwap) | **verified live** | 8 sells after the GRADUATED revert fix, tx `0x06ff458a...` first, all status 1; a thin-pool exit returned 1.61 ETH on a 0.015 ETH position |
| Points — official API | **verified live** | `scripts/site-points.js` reads `/api/points/<addr>`: 21 wallets, **8 eligible**, 9,357.79 points, against 3,831 ranked wallets. The local estimator under-counts Crew by 150–300, so it is only a fallback |
| Fuel ladder (curve only) | **verified live** | 0.1669 ETH into NFL2 #4662 from a 0.169944 top-up; the wallet went 338 -> 504 points and became eligible, tx `0xb1f45ead...` |
| Staking | **reads verified, no funds** | totalStaked 935,284,979 T0, pending() = 0 for every wallet |

### Recent live txs (2026-11-09 → 2026-11-10)

| Action | Result | Tx |
|---|---|---|
| Exit a position whose pool was thin | 0.015 ETH in -> 1.6106 ETH out | `0xcdb68e10...` |
| Bridge Sepolia -> Base Sepolia (Relay, testnet) | 0.5 ETH, fee ~0.00001 | deposit `0x7f11cf47...`, arrival `0x02bd4d17...` |

### Recent live txs (2026-11-08)

| Action | Tx |
|---|---|
| GB backer 1-5 | `0xdedb0ef7...`, `0x3823a99f...`, `0xd6a46fea...`, `0xb43366ae...`, `0xb93d9165...` |

### One full group-buy cycle, measured end to end

| Line item | ETH | Result |
|---|---|---|
| creation fee + creator initial buy (0.012) | 0.0121 | creator earns **+500 Liftoff** |
| 5 backers × 0.0105 | 0.0525 | each earns **+150 Crew** |
| fill (bought the rest of the curve) | 0.14965 | liftoff inside our own tx |
| **total cycle cost** | **~0.2143** + gas | **+1,250 points** |

Crew qualification checked against chain timestamps, not hope: all five backers bought
13:42–13:46 UTC, liftoff was 14:53:02 UTC, so each held 66.2–70.5 minutes with tokens still in
the wallet. That satisfies the "0.01 ETH kept 60 minutes, still holding at liftoff" clause.

The project's own points page was stuck at "COUNTING POINTS... 80%" during and after the cycle, so
the site could not be used to confirm the credit. The on-chain conditions are verified; the
crediting itself is the project's side.

## File structure

```
tzero-bot/
├── index.js              # CLI menu (17 actions) + flags
├── src/
│   ├── contracts.js      # addresses + ABIs (launchpad, router, factory, staking, fee splitter)
│   ├── launchpad.js      # read layer (Multicall3) + fillQuote + near-liftoff scan + stake reads
│   ├── actions.js        # buy / sell / create / claim / sweep / settle / daily
│   ├── ledger.js         # per-buy positions, hold windows, daily Fuel cap
│   ├── points.js         # the published point rules + local estimator
│   ├── crew.js           # sniper + 5-wallet crew train
│   ├── gb.js             # group-buy state machine (create -> back -> fill)
│   ├── humanize.js       # per-wallet persona (stable per address) + log-normal tempo
│   └── staking.js        # $T0 stake / unstake / claim / yield report
├── scripts/              # 23 operator helpers (all read-first, --execute to act)
│   ├── site-points.js    # official points for every wallet (authoritative)
│   ├── proxy-check.js    # test every proxy line: RPC reachable + egress IP
│   ├── scan-depth.js     # how deep is the near-liftoff window, which candidates are outside it
│   ├── holdings.js       # quote every held token on its correct venue
│   ├── check-position.js  exit-position.js   fuel-topup.js   fuel-ladder.js
│   ├── eligibility-catchup.js               # push a 460-499 wallet over 500
│   ├── balance.js  send.js                  # balance snapshot + manual one-off transfer
│   ├── sepolia.js                           # Relay bridge Sepolia -> Base Sepolia
│   ├── gb3-bridge.js  gb3-driver.js  gb*-plan-init.js
│   └── legacy-audit.js  vault-extract.js  state-summary.js  status.js
├── utils/                # logger, banner, helpers (saveJson handles BigInt), proxy.js
├── docs/screenshots/     # the proof images used above
├── data/
│   ├── positions.json    # every buy with its timestamp (gitignored)
│   ├── gb.json           # group-buy cycles (gitignored)
│   ├── state.json        # launches + last run per wallet (gitignored)
│   └── runs.json         # last 60 runs (gitignored)
├── JALANKAN.bat          # double-click menu (Indonesian)
├── CEK-POIN.bat  CEK-STATUS.bat  CEK-PROXY.bat  LIHAT-CRON.bat
├── PANDUAN-MANUAL.md     # click-by-click guide + error table
├── test-hold.js          # operational test: buy, refuse settle, sell after hold
└── license-check.js
```

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `daily Fuel cap reached` | 1 ETH of buys already spent today (UTC) | wait for the next UTC day |
| `position(s) still inside the 65 min hold window` | settle ran too early | run `--settle` later; this is the point |
| `no mission is within 0.05 ETH of liftoff` | nobody is close right now | crew is opportunistic; scan again in 15 min |
| `only N/5 wallet(s) hold 0.011 ETH` | that wallet is short of gas | top that wallet up from outside (faucet/bridge); 1.0.3 has no `--fund` |
| `no wallet can afford the initial buy` | group buy needs 0.29 ETH | fund the fleet first |
| buy reverts | price moved past slippage | crew/gb use 9% slippage; daily uses 1.8-4.8% |
| `execution reverted: "GRADUATED"` | selling a mission that already lifted off, on the curve | fixed: sells now check the venue and use TzeroSwap's router after liftoff |
| RPC timeout | public endpoint rate limit | set your own `rpcUrl` |
| `wallet ini DILEWATI` | proxy preflight failed, wallet skipped on purpose | this is the safety rule, not a bug: a dead proxy never falls back to your real IP |
| `Socks5 proxy rejected connection - AddressNotSupported` | the socks proxy refuses the address type | use `socks5h://` (proxy-side DNS) or switch that line to `http://` |
| daily run goes quiet then dies at the 60 min cron limit | a tx never mined and `tx.wait()` had no ceiling, so the whole batch waited on it | fixed in 1.0.5: every wait is capped by `TZERO_TX_TIMEOUT_MS` (default 240000 ms) and that step is logged as an error while the run moves on |

## Changelog

**1.0.5 (2026-11-12)** — a dead tx can no longer freeze the batch

- every `tx.wait()` in `src/actions.js` now goes through `waitTx()`, capped by `TZERO_TX_TIMEOUT_MS`
  (default 240000 ms = 4 min); on timeout the step is recorded as an error and the next wallet runs
- why: on 2026-11-11 the 10:00 daily batch bought for 4 wallets, then sat on wallet 5 for 43 minutes
  with no output and was killed by the 60 min cron limit — one never-mined tx took the rest of the
  batch with it
- revert reasons still propagate unchanged, so the GRADUATED -> TzeroSwap fallback keeps working

**1.0.3 (2026-11-11)** — no funding wallet

- removed `--fund`, the "sebar saldo" menu item, `actions.fundWallets` and `scripts/distribute.js`
  no longer act as the fleet's gas station
- `scripts/fuel-ladder.js` no longer tops a wallet up from another wallet; it stops and tells you to
  fund that wallet from outside
- `daily.excludeSourceFromDaily` and the six `daily.fund*` keys are gone (an old `config.json` still
  loads; the keys are ignored)
- buyer-facing errors now say "top up a wallet from outside" instead of "fund the fleet"

**1.0.2 (2026-11-10)** — fail loudly instead of silently doing nothing

- if every wallet fails its proxy preflight, the run stops with a clear error and exit code 1,
  so a cron job cannot report success while no wallet actually ran

**1.0.1 (2026-11-10)** — per-wallet proxy support

- the proxy is attached to each wallet's provider (`ethers.FetchRequest.createGetUrlFunc({ agent })`),
  so JSON-RPC and the points API leave from that wallet's IP inside a single process
- a dead proxy skips its wallet instead of silently connecting direct
- new flags `--proxy`, `--no-proxy`, `--proxy-check`; menu item 18; `CEK-PROXY.bat`
- `socks5://` DNS is forced to IPv4 (IPv6 answers broke connections on IPv4-only networks)
- new dependencies: `https-proxy-agent`, `http-proxy-agent`, `socks-proxy-agent`

**1.0.0** — first build: daily fuel run, settle with hold window, crew sniper and train, group buy
state machine, staking, persona humanization, points estimator.

## Disclaimer

Testnet automation for research and early positioning. No token on mainnet yet, no confirmed airdrop,
no guaranteed reward. Points are provisional until the announced snapshot, and the project's own sybil
review may remove clustered wallets. Never share private keys.

Contact: Telegram [@rmndkyl](https://t.me/rmndkyl) · Channel [@layerairdrop](https://t.me/layerairdrop)