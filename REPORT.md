# Report — TN10 public-node load run, Sat 10 Oct 2026

Times: **CEST first**, UTC second.

## Summary (Claim — measured on TN10)

| Metric | Value |
|---|---|
| Window | ~14:23–16:10 CEST (12:23–14:10 UTC) |
| Peak bot submitted | **~3,168 tx/s** (14:38 CEST, 22–25 runners) |
| Sustained clean (stable phase) | **~530–750 tx/s** at 12–14 runners (15:23–15:30+ CEST), 0 rejects |
| Peak network unique accepted | **~2,686 tx/s** (14:37 CEST; includes Build on the same 5 nodes from ~14:22) |
| Max runners tried | 25–27 (then RAM abort / OOM) |
| Max **stable** runners | **14** (singleton launcher, hard cap; 12 held clean for long stretches) |
| Fees | 200 sompi/gram → **400** from 14:26 CEST (stp) |
| Nodes | muon / boson / electron / neutrino / vector (resolver TN10 Borsh) |
| Box limit hit | **RAM** — OOM-killer killed n2 kaspad ~14:51 CEST at high runner counts; hard-capped after that |

## Per-node behaviour (Claim — measured)

During the clean 12–14 runner phase, almost all successful submits went to **muon-10.kaspa.blue** and **vector-10.kaspa.green**. boson / electron / neutrino often failed `pub_connect` at boot under load, so runners fell back to the working pair.

| Node | Notes |
|---|---|
| muon-10.kaspa.blue | Lowest RTT earlier (~6 ms); carried large share of ok submits; 0 rejects in clean phase |
| vector-10.kaspa.green | Carried large share; 0 rejects in clean phase; busier mempool / higher fee quotes earlier |
| boson / electron / neutrino | Frequent connect failures when many runners booted together (**Needs more testing** whether rate-limit or transient) |

Rejects spiked only during chaotic overspawn windows (multiple launchers), not during the locked 12–14 runner hold.

## What went wrong / fixes (Claim + open)

1. **Multiple `launch.mjs` processes** (relative argv hid them from `pkill pub1010/launch`) respawned senders → 48–72 processes, 0 submit (boot watchdog). **Fix:** singleton lock file + kill by `launch.mjs` / `pub-sender.mjs` name; adopt existing PIDs; replace at most one dead runner / 15 s; never above `MAX_RUN`.
2. **Boot hangs** when many runners opened 4 public WSS conns in parallel. **Fix:** n2 connect + health first; public conns serial; 128 lanes / 2 conns; 1.5–2 s stagger.
3. **Coin list hit 0 unclaimed ~15:10 CEST** → submit collapsed. **Fix:** emergency +250k tKAS fill from old coinbases (same claim ledger), merge into `keel-coins.json`.
4. **OOM killed n2** ~14:51 CEST after scaling past ~20 runners. Disk guard restarted n2 (synced again). **Fix:** hard cap 14; abort trim if avail RAM &lt; 1.8 GB.

## Build interaction (Claim)

stp noted Build also launched on the **same 5 public nodes** from ~14:22 CEST with its own coins. Network unique-accepted and slot rates therefore measure **combined** bot+Build load. Bot submitted counts below are bot-only.

## 17:00 private-node storm readiness (Claim — config on disk)

- `storm1010p/storm.mjs`: **ALWAYS** `pub-sender.mjs` (stp 14:36 override); no `n2-sender` launch path. n2 kept for coin/UTXO/VCC/reads.
- Deviation logged in `out/timeline.md` / `events.csv`.
- Scheduler restart via watchdog scheduled in teardown (~16:05 CEST) so the running process loads the pub-only path before G1 16:15.
- Coin buffer after fills: on the order of **~1M+ tKAS** merged earlier; top-up +250k at 15:13. Recheck at arm time.
- Watchdog pid expected under `storm1010p/watchdog.sh`.

## Certainty

| Label | Use in this report |
|---|---|
| **Claim (measured on TN10)** | Peak/sustained rates, OOM event, fee change, Build overlap, launcher failure mode |
| **Not sure / open for debate** | Why boson/electron/neutrino connect failed under fan-out |
| **Needs more testing** | Whether 14 runners × 150 tx/s is the true box ceiling with all 5 nodes healthy; multi-hour public-node endurance |

## Data files

- `data/minute.csv`, `data/host-10s.csv`, `data/events.jsonl`, `data/events.csv`
