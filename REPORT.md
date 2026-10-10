# Report — TN10 public-node load run, Sat 10 Oct 2026

Times: **CEST first**, UTC second. Finalized ~16:24 CEST.

## Summary (Claim — measured on TN10)

| Metric | Value |
|---|---|
| Window | ~14:23–15:57 CEST (12:23–13:57 UTC); senders/launcher torn down at 16:18 CEST after overrun of 16:10 target |
| Peak bot submitted | **~3,168 tx/s** (14:38 CEST, 22 runners; acc ~2,969) |
| Earlier high | **~2,415 tx/s** (14:27 CEST, 21 runners) |
| End-of-hold clean | **~880–985 bot sub/s** at **14 runners** (last ~10 samples avg ~919; last row 15:57 CEST / 13:57 UTC: 984.7 sub/s, 976.4 acc/s, 0 rejects) |
| Max runners tried | 25 (then RAM / OOM pressure) |
| Max **stable** runners | **14** (singleton launcher hard cap) |
| Fees | 200 sompi/gram → **400** from 14:26 CEST (stp) |
| Nodes | muon / boson / electron / neutrino / vector (resolver TN10 Borsh wss) |
| Box limit hit | **RAM** — OOM-killer killed n2 kaspad ~14:51 CEST at high runner counts; hard-capped after that |

## Per-node behaviour (Claim — measured)

During the clean 12–14 runner phase, almost all successful submits went to **muon-10.kaspa.blue** and **vector-10.kaspa.green**. boson / electron / neutrino often failed `pub_connect` at boot under load, so runners fell back to the working pair.

| Node | Notes |
|---|---|
| muon-10.kaspa.blue | Lowest RTT earlier; large share of ok submits; 0 rejects in clean phase |
| vector-10.kaspa.green | Large share; 0 rejects in clean phase |
| boson / electron / neutrino | Frequent connect failures when many runners booted together (**Needs more testing**) |

Rejects spiked only during chaotic overspawn windows (multiple launchers), not during the locked 12–14 runner hold.

## What went wrong / fixes (Claim)

1. **Multiple `launch.mjs` processes** respawned senders → boot watchdog / 0 submit. **Fix:** singleton lock + kill by name; adopt PIDs; replace at most one dead runner / 15 s; hard cap `MAX_RUN=14`.
2. **Boot hangs** on parallel public WSS fan-out. **Fix:** n2 first; public conns serial; stagger; fewer lanes/conns under load.
3. **Coin list hit 0 unclaimed ~15:10 CEST**. **Fix:** +250k tKAS fill into `keel-coins.json` (same claim ledger).
4. **OOM killed n2** ~14:51 CEST after scaling past ~20 runners. **Fix:** hard cap 14; abort trim if avail RAM &lt; 1.8 GB.
5. **n2 tip freeze ~15:57–16:20 CEST** (virtual DAA stalled, sinkAge climbing, Gate 1 FAIL). **Fix:** kaspad restart with utxoindex; n2 resynced ~16:20 CEST. Storm `pubMode` patched so n2 lag/unsynced no longer zeros public-node senders.

## Build interaction (Claim)

Build also launched on the **same 5 public nodes** from ~14:22 CEST with its own coins. Network unique-accepted / slot rates are **combined** bot+Build. Bot submitted counts in `data/minute.csv` are bot-only.

## Teardown / handoff to 17:00 storm (Claim)

- Parent killed pub1010 senders/launcher/teardown at **16:18 CEST** (overran 16:10). **Do not restart** the 2h pub experiment.
- 17:00 storm (`storm1010p`): **ALWAYS** submit via `pub-sender.mjs` to the 5 public nodes (stp 14:36 + 16:19 overrides). n2 for VCC/coin reads when healthy.
- Coin buffer at arm: **61,401** keel coins (~49 tKAS each class) — enough for bot 25% share rates.
- Smoke (16:22 CEST): one `pub-sender` ~30 tx/s target → ~30 sub/s accepted on public path (vector/muon), then killed.

## Certainty

| Label | Use |
|---|---|
| **Claim (measured on TN10)** | Peak/sustained rates, OOM, fee change, Build overlap, launcher failure, end-hold ~950–980 @ 14 runners |
| **Not sure / open** | Why boson/electron/neutrino connect failed under fan-out |
| **Needs more testing** | Multi-hour public-node endurance; full 5-node balance at 14 runners |

## Data files

- `data/minute.csv` — 1 Hz aggregated bot rates (final through ~13:57 UTC / 15:57 CEST)
- `data/host-10s.csv`, `data/events.jsonl`, `data/events.csv`
