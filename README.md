> **Experimental. We are just trying this.** Kaspa Testnet-10 only; tKAS has no value.

# tn10-storm-1010-public-nodes: the bot on the five public TN10 nodes

**Status:** done. Sat 10 Oct 2026, about 14:23 to 16:18 CEST (12:23 to 14:18 UTC). Private for now; made public after tonight's run, once the logs are on it.

## Map: where to find what

| Folder or file | What is in it |
|---|---|
| [README.md](README.md) | This page: map, intro, briefing, results |
| [REPORT.md](REPORT.md) | The full report: numbers, per-node behaviour, what went wrong and the fixes, Build overlap, handoff, labels |
| [data/](data/) | The raw counts (no txids, keys or addresses) |
| [data/minute.csv](data/minute.csv) | Bot rates every ~10 s to 60 s: senders, submitted, rejected, accepted, blocks, mempool, CPU, RAM, disk, per-node ok/rej/err |
| [data/host-10s.csv](data/host-10s.csv) | Box CPU, RAM, disk and sender count every 10 s |
| [data/events.csv](data/events.csv), [data/events.jsonl](data/events.jsonl) | Scale steps, node drops, fee change, aborts, in order |
| [tn10-storm-1010](https://github.com/STP-KAS/tn10-storm-1010) | **The new plan** (private): tonight's bot run and build run, T0 20:30 CEST, built on these lessons |
| [Kaspa Pulse (@gokugalax)](https://x.com/gokugalax) | The outside counter whose method shaped the plans this tooling came from |

## Intro

**What.** A short, maximum-effort load run: the bot's hop senders submitted to the **five public TN10 nodes** (`muon-10.kaspa.blue`, `boson-10.kaspa.red`, `electron-10.kaspa.blue`, `neutrino-10.kaspa.stream`, `vector-10.kaspa.green`), while the bot's private node n2 stayed the reader (sync, UTXO index, accepted counts). Ordered by stp at 14:20 CEST.

**Why.** n2 is disk-tight after a full sync. We wanted a measured answer: how hard can public nodes be pushed from one box, and does a second submit path take load off our own node?

**How.** A copy of the bot sender, random public nodes per sender, the shared coin list and claim ledger (no double spends with other runs), a launcher that scales up to the RAM/CPU limit, per-node cool-down on errors, counts every 10 s and every minute. Details: [REPORT.md](REPORT.md).

## Briefing

- Build sent to the same five public nodes from about 14:22 CEST with its own coins, so network counts were combined; the bot's counts in [data/minute.csv](data/minute.csv) are bot only.
- Fees went from 200 to **400 sompi/gram** at 14:26 CEST (stp).
- The run ended before tonight's plan; its lessons are folded into [tn10-storm-1010](https://github.com/STP-KAS/tn10-storm-1010).

## Results (Claim (measured on TN10) unless marked)

| Key number | Value | Detail |
|---|---|---|
| Peak bot submitted | about **3,168 tx/s** at 14:38 CEST (22 senders), about 2,969 accepted | [REPORT.md](REPORT.md#summary-claim--measured-on-tn10) |
| Earlier highs | about 2,415 tx/s at 14:27 CEST (21 senders); about **2,430 tx/s** at about 14:35 CEST (19 senders, fees 400) | [data/minute.csv](data/minute.csv) |
| Clean hold | about **880 to 985 tx/s** submitted on **14 senders**; last row 15:57 CEST: 984.7 submitted, 976.4 accepted, 0 rejects | [REPORT.md](REPORT.md#summary-claim--measured-on-tn10) |
| Rejects | 0 in the clean phase for most of the run; spikes only during the launcher overspawn | [REPORT.md](REPORT.md#per-node-behaviour-claim--measured) |
| Per-node behaviour | muon-10 and vector-10 took most ok submits; boson-10, electron-10 and neutrino-10 often refused connections when many senders booted at once (why: **Not sure / open for debate**) | [REPORT.md](REPORT.md#per-node-behaviour-claim--measured) |
| Launcher overspawn | two launchers, senders past 20, rate collapsed; fixed with one locked launcher and a hard cap of 14 | [REPORT.md](REPORT.md#what-went-wrong--fixes-claim) |
| Box limit | RAM: the out-of-memory killer took n2 at about 14:51 CEST above about 20 senders | [REPORT.md](REPORT.md#what-went-wrong--fixes-claim) |
| n2 tip freeze | about 15:57 to 16:20 CEST; fixed by a restart; since then n2 lag never zeros public-node senders | [REPORT.md](REPORT.md#what-went-wrong--fixes-claim) |
| Endurance over hours, full five-node balance | **Needs more testing** | [tn10-storm-1010](https://github.com/STP-KAS/tn10-storm-1010) |

**After the run, once the logs are on GitHub, TN10 ops makes this repo public** (after tonight's run, together with tn10-storm-1010).

<details>
<summary><b>Certainty labels</b></summary>

| Label | Meaning |
|---|---|
| **Claim (measured on TN10)** | Observed on Testnet-10 during this run, with numbers. |
| **Not sure / open for debate** | Plausible reading; other explanations fit. |
| **Needs more testing** | One run is not enough. |

</details>

Thank you, Kaspa Pulse ([@gokugalax](https://x.com/gokugalax)): his feedback shaped the GO plans this tooling came from. The public nodes are community-operated via the Kaspa resolver. Counts only: no txid, key, seed, IP, port or wallet address in this repo.
