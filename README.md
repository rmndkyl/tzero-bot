# TZERO BOT — LYID Automation

Automation for **Tzero**, the first launchpad on **Zero** (LayerZero's L1), currently running on
**Base Sepolia** (chain 84532). Zero mainnet is expected Q4 2026 and Tzero Season 0 "Pre-flight
Points" is not live yet, so every on-chain action now is early positioning — and it costs **0 real
money**.

Built by [LYID — Layer Airdrop ID](https://t.me/layerairdrop) · Telegram [@rmndkyl](https://t.me/rmndkyl)

---

## What the bot does

Daily loop, per wallet, all on testnet ETH:

1. Reads the live mission board straight from the contract (`tokenCount`, `tokenAt`, `curve`,
   `progressBps`).
2. Picks the missions with the lowest curve progress (most room before liftoff).
3. **Buys** a configurable amount on each (default 0.0004 ETH), then **sells** a slice back
   (default 40%) so the wallet produces both buy and sell volume.
4. **Launches** its own mission once per wallet (`create` with an optional initial buy, both in the
   same transaction).
5. **Claims** accumulated creator fees from the FeeSplitter (50% of every trading fee on your
   missions is yours, on the curve and after liftoff).

Everything is wrapped in per-wallet try/catch, a balance guard, randomized delays and a slippage
cap, so one bad wallet never kills the run.

## Features

| Feature | Status |
|---|---|
| Read-only status + live mission table | working |
| Dry quote (buy + sell estimate, no gas) | working |
| Daily activity run (buy -> sell per mission) | working |
| Launch mission (`create` with initial buy) | working |
| Claim creator fees | working |
| Fund the fleet from one wallet (`--fund`, one faucet drip -> N wallets) | working |
| Multi-wallet (`privateKey.txt`, one key per line) | working |
| `--daily` / `--dry-run` flags for cron | working |
| Per-wallet state (`data/state.json`) + run history (`data/runs.json`) | working |
| Owner bypass license check (`.lyid-owner`) | working |

## Requirements

| Item | Required? | Cost |
|---|---|---|
| Node.js 18+ (tested on v22) | yes | free |
| Base Sepolia test ETH | yes | **free** (faucet) |
| EVM wallet / private key per account | yes | free |
| Proxy | no | - |
| Captcha solver | no | - |
| Real money | **no** | Rp 0 |

### Where to get free Base Sepolia ETH

| Faucet | Gate | Notes |
|---|---|---|
| https://console.optimism.io/faucet (Superchain) | GitHub login or onchain identity | 1 claim per identity, enough for weeks |
| https://portal.cdp.coinbase.com/products/faucet | Coinbase account | small drip |
| https://www.alchemy.com/faucets/base-sepolia | needs 0.001 ETH on Ethereum mainnet | not usable with zero capital |

Fund **one** wallet, then send 0.003-0.005 ETH to the rest with a normal transfer — Base Sepolia
gas is a few thousandths of a cent, so one drip covers many wallets.

**Budget per wallet per day** (default config): 3 buys x 0.0004 + 1 launch (0.0001 fee + 0.0002
initial buy) + gas ≈ **0.0017 ETH/day** of free test ETH.

## Setup

```bash
cd tzero-bot
npm install
cp config.json.example config.json     # optional: edit amounts
cp privateKey.txt.example privateKey.txt
# paste one private key per line into privateKey.txt
node index.js --status                  # read-only check, no gas
node index.js --daily --dry-run         # simulate the daily loop, no gas
node index.js --daily                   # real run
```

## Usage

Interactive menu:

```
node index.js
```

| # | Menu | What it does |
|---|---|---|
| 1 | Status + live missions | tokenCount, creation fee, 12 newest open missions |
| 2 | Dry quote | buy/sell estimate on the lowest-progress mission |
| 3 | Daily activity run | buy -> sell -> launch -> claim, all wallets |
| 4 | Launch a mission | one `create` per wallet, tracked in state |
| 5 | Claim creator fees | claim from FeeSplitter |
| 6 | Daily run (dry) | same as 3 without sending transactions |
| 7 | Fund wallets | sends `fundAmountEth` from wallet #1 to every other wallet |

Flags (for cron / Task Scheduler):

```bash
node index.js --status
node index.js --daily
node index.js --daily --dry-run
node index.js --launch
node index.js --claim
node index.js --fund
```

Fan out one faucet drip to the whole fleet (wallet #1 in `privateKey.txt` is the source):

```bash
node index.js --fund --dry-run   # see who needs what
node index.js --fund             # 0.003 ETH each (config: daily.fundAmountEth)
```

Windows Task Scheduler / cron example (daily 09:00):

```
schtasks /create /tn "tzero-daily" /tr "node C:\Users\<user>\LYID-BOTS\tzero-bot\index.js --daily" /sc daily /st 09:00
```

## Configuration (`config.json`)

| Key | Default | Meaning |
|---|---|---|
| `daily.minBalanceEth` | 0.0025 | wallet skipped below this balance |
| `daily.buyAmountEth` | 0.0004 | native spent per mission |
| `daily.missionsPerRun` | 3 | missions traded per wallet per run |
| `daily.sellPercent` | 40 | % of the bought tokens sold back |
| `daily.createMission` | true | launch one mission per wallet |
| `daily.createInitialBuyEth` | 0.0002 | initial buy bundled into `create` |
| `daily.maxLaunchesPerWallet` | 1 | cap on launches per wallet |
| `daily.claimCreatorFees` | true | claim fees every run |
| `daily.scanMissions` | 40 | how many recent missions to scan |
| `daily.maxProgressBps` | 9500 | skip missions near liftoff |
| `daily.slippageBps` | 300 | 3% min-out protection |
| `daily.delayMinMs` / `delayMaxMs` | 8000 / 20000 | randomized pause between actions |

## Project info

| Field | Value |
|---|---|
| Project | Tzero — launchpad for Zero (LayerZero L1) |
| Network | Base Sepolia, chain id 84532 |
| App | https://testnet.tzer0.fun |
| Docs | https://testnet.tzer0.fun/docs |
| X | https://x.com/T0dotfun |
| Token | $T0 (testnet) — 20% of every trading fee goes to stakers |
| Points | Season 0 "Pre-flight Points" announced, **not live yet** — "earned only from verifiable on-chain actions" |
| TGE | Zero mainnet expected Q4 2026 |
| Launchpad | `0xbce48917C41BB45Ad15f859E4Bf0e4EEa74bAd5c` |
| FeeSplitter | `0x1C69a90c69fdb2F2E459fCA23cA6B22ACD707B3b` |
| Staking | `0x221De22612E65093Fc43164dae84fa6Bd79E6239` |

## Verification status (honest)

| Module | Status | Evidence |
|---|---|---|
| RPC + contract reads | **Verified** | live run: `missions: 975`, `creation fee: 0.0001 ETH`, mission table with real progress |
| `quoteBuy` / `quoteSell` | **Verified** | live quotes returned (0.0004 ETH -> ~4.2M tokens on 0.1% progress mission) |
| `buy` calldata | **Verified (simulation + live)** | simulation `tokensOut = 3,771,851.79`; live tx `0x473c9c1c6227fc6c00a1696755d09c0c6d4d5374d8704963bcdf56d3ee4274a9` |
| `create` calldata | **Verified (simulation + live)** | simulation predicted a deploy; live tx `0x65dffc6f...` deployed `0xc72a2168...` (PHOTONEC) |
| Daily loop wiring | **Verified** | `node test-daily.js`: 4 actions, 0 errors (dry run) |
| `sell` broadcast | **Verified** | tx `0xdaef6ae0b2e293f9b4bde4f214169ecf14f29f8d5e7ff5ca4f90c685afbb44e4` (approve + sell, status 0x1, 139,966 gas) |
| `create` broadcast | **Verified** | tx `0x65dffc6fa9d8bc1c33bb7a631f654bcca2bd032ef5d5cc83ff5738a469df6d59`, deployed `0xc72a2168a7E6036A0E614bF23e23AA47630523ae` (PHOTONEC / "Photon Echo 12"), 1,002,765 gas |
| `claimCreatorFees` broadcast | **Verified** | tx `0xcfab13a8166022fc7e2d433e86325af0c5a0bcf65568d018a801de38978ac7f4`, paid 0.000001 ETH |

### First live run (2026-11-06, Base Sepolia)

Wallet `0x7490F634EaadE1A76E43216C3D155bcDBa289A01`, funded 0.1124 ETH from the Superchain faucet.
8 transactions, all `status: 0x1`:

| # | Action | Mission | Tx |
|---|---|---|---|
| 1 | buy 0.0004 | JW12 #983 | `0x473c9c1c6227fc6c00a1696755d09c0c6d4d5374d8704963bcdf56d3ee4274a9` |
| 2 | sell 40% | JW12 #983 | `0xdaef6ae0b2e293f9b4bde4f214169ecf14f29f8d5e7ff5ca4f90c685afbb44e4` |
| 3 | buy 0.0004 | JW114 #995 | `0x7f001fa132f4bfbc58ef1aaa8f4e44c1ce7da0d927c81af5b64ad79a36d27431` |
| 4 | sell 40% | JW114 #995 | `0x85693bdb8497e03b53c82492d60faff3f77f122c4a1e834bd37e7baa66c44d9b` |
| 5 | buy 0.0004 | JW115 #996 | `0x785576e78511dee28d7c04a75c566636088f8144abd38b08855cd42c2eb62728` |
| 6 | sell 40% | JW115 #996 | `0xabd1b1ae252b1e371f08ec14363349d696ff76a7ce4c8229cf5e4d6514291844` |
| 7 | create | Photon Echo 12 | `0x65dffc6fa9d8bc1c33bb7a631f654bcca2bd032ef5d5cc83ff5738a469df6d59` |
| 8 | claim fees | - | `0xcfab13a8166022fc7e2d433e86325af0c5a0bcf65568d018a801de38978ac7f4` |

Total spend: **0.00104 ETH** testnet (balance 0.122391 -> 0.121350), cheaper than the 0.0017 estimate.

## File structure

```
tzero-bot/
├── index.js              # CLI menu + flags
├── src/
│   ├── contracts.js      # addresses + human-readable ABIs
│   ├── launchpad.js      # read layer (stats, missions, quotes)
│   └── actions.js        # write layer (buy, sell, create, claim, daily)
├── utils/
│   ├── logger.js
│   ├── banner.js
│   └── utils.js
├── config.json.example
├── privateKey.txt.example
├── license-check.js
├── data/state.json       # per-wallet launches + last run (gitignored)
└── data/runs.json        # last 60 runs (gitignored)
```

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `balance below minBalanceEth` | wallet empty | get test ETH, or lower `minBalanceEth` |
| `missing revert data` on estimateGas | sender has 0 balance | fund the wallet |
| `no tradeable missions found` | all scanned missions graduated | raise `scanMissions` |
| buy reverts | price moved past 3% slippage | raise `slippageBps` |
| `no token balance to sell` | buy failed earlier | check the buy error above it |
| RPC timeout | public RPC rate limit | set your own `rpcUrl` (Alchemy/QuickNode Base Sepolia) |
| license error | no `.lyid-owner` | owner file ships with the repo; buyers get a license key |

## Disclaimer

Testnet automation for research and early-positioning purposes. No token, no confirmed airdrop,
no guaranteed reward. Use wallets you control and never share private keys.

Contact: Telegram [@rmndkyl](https://t.me/rmndkyl) · Channel [@layerairdrop](https://t.me/layerairdrop)
