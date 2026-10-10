# TN10 public-node load run — Sat 10 Oct 2026

**Certainty labels used in this repo**

| Label | Meaning |
|---|---|
| **Claim (measured on TN10)** | Observed on Testnet-10 during this run, with numbers. |
| **Not sure / open for debate** | Plausible reading; other explanations fit. |
| **Needs more testing** | One run is not enough. |

## What

A short, maximum-effort load run of the bot's P2W hop senders against **public** TN10 wRPC nodes (Kaspa community resolver endpoints), while the bot's own private node (n2) stays the source of truth for sync, UTXO index, and confirmation tracking.

- Window: about **14:25–16:10 CEST** (12:25–14:10 UTC), ~1 h 45 min. Ended before the 16:15 CEST gate of the scheduled private-node storm (STP-KAS/tn10-storm-1010).
- Shape: unsigned P2SH(push4 OP_DROP OP_TRUE) hop lanes, fee **200 sompi/gram**, 4 connections per runner, depth 4.
- Public nodes used (resolver `*.kaspa.{stream,red,green,blue}` → TN10 Borsh):
  - `muon-10.kaspa.blue`
  - `boson-10.kaspa.red`
  - `electron-10.kaspa.blue`
  - `neutrino-10.kaspa.stream`
  - `vector-10.kaspa.green` (skipped when its normal fee quote > 200 sompi/gram)
- Private node n2 on the box: health, virtual-chain-changed, chain counts, mempool. Not used for submit in this run.

This run is **not** the GO agenda in `tn10-storm-1010`. It is a separate experiment ordered at 14:20 CEST to find how hard public nodes can be pushed from one box.

## Why

1. The bot's private node (n2) is disk-constrained after a full IBD. Public nodes give a second path for submit capacity without putting more write load on n2's RocksDB.
2. We wanted a measured answer to: can 15–30 runners × 4 connections sustain hundreds of tx/s against the public TN10 resolver set for ~2 hours from one host?
3. Results feed the later private-node storm (17:00 CEST T0) with a clear separation: public-node capacity vs own-node capacity.

## How

1. Copy of the bot sender (`pub-sender.mjs`): submits to a randomly assigned public host list; sync / UTXO / VCC always on n2 loopback.
2. Shared coin list with the private-node storm (`keel-coins.json`) and the same claim ledger (O_EXCL claim file + append-only used list), so the two runs cannot double-spend.
3. Orchestrator scales runners up until free RAM ~1.8–2.2 GB or CPU ~85%, then holds. Per-host cool-down on disconnect / error bursts.
4. Logging every ~10 s (host) and every ~60 s (rates, per-node ok/rej/err, n2 mempool, blocks/s). No per-tx publish. No keys, seeds, or private paths in this repo.

## Data

- `data/minute.csv` — per-minute rates and per-node counters
- `data/host-10s.csv` — box CPU / RAM / disk / runner count
- `data/events.jsonl` — scale steps, host drops, abort reasons (no secrets)
- `REPORT.md` — final numbers and labels (written at end)

## Credit

Kaspa Pulse feedback shaped the earlier GO plans this tooling came from. Public nodes are community-operated via the Kaspa resolver.
