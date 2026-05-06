# Brain-Shield — Performance Benchmark Results

**3-Layer Anti-Rug Shield for Solana memecoin traders**
**Layer 1 (LP Withdrawal) — Live Performance Benchmark — 24h continuous run**

Brain-Shield is a unified anti-rug protection layer for Solana memecoin
traders, covering the 3 dominant rug vectors observed across pump.fun,
BONK, LetsBonk, Believe, Moonshot, Daos.fun, PumpSwap AMM and any future
launchpad:

| Layer | Rug vector |
|---|---|
| **L1** | LP Withdrawal (pool drain) |
| **L2** | Coordinated Dump (multi-wallet sell bundle) |
| **L3** | Cartel Operation (coordinated wallets pump + dump) |

**This document presents the production-validated benchmark for Layer 1.**

---

## Executive Summary

Brain-Shield (Layer 1) processes the live Solana ShredStream feed and
detects pool liquidity withdrawals (rug events) before they finalize
on-chain.

A side-by-side benchmark was run on a Frankfurt VPS (~0.6 ms to Jito relay)
comparing two pipelines on the **same** ShredStream input:

- **Pipeline A — baseline** : standard processing path
- **Pipeline B — Brain-Shield** : optimized processing path

Both pipelines run in the **same process**, on the **same entries**,
**sequentially** — making the comparison strictly deterministic.

---

## Headline numbers (live ShredStream, 24h continuous run)

| Metric | Pipeline A (baseline) | Pipeline B (Brain-Shield) |
|---|---:|---:|
| Total entries processed | **3 750 000** | **3 750 000** |
| Withdraws detected | **1 359** | **1 359** |
| False negatives | — | **0 / 1 359** |
| Avg processing time per entry | **985.0 µs** | **13.5 µs** |
| Relative CPU usage | 100% | **1.4%** |
| Bundle-sell events captured (Layer 2) | — | **2 507** |

### **Speedup B vs A : 72.9× — sustained over 24h — zero drift**

---

## Stability across the full 24-hour run

| Checkpoint | Elapsed | Entries | Withdraws A=B | Avg A | Avg B | Speedup |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 203.6 s | 10 000 | 3 / 3 | 995.5 µs | 13.0 µs | **76.5×** |
| 2 | 411.9 s | 20 000 | 5 / 5 | 996.9 µs | 12.9 µs | **77.1×** |
| 3 | 611.5 s | 30 000 | 6 / 6 | 1 018.0 µs | 13.4 µs | **75.8×** |
| 4 | 790.5 s | 40 000 | 9 / 9 | 1 019.2 µs | 13.3 µs | **76.5×** |
| 5 | 969.8 s | 50 000 | 15 / 15 | 1 016.6 µs | 13.3 µs | **76.7×** |
| 6 | 1 159 s | 60 000 | 19 / 19 | 1 023.4 µs | 13.4 µs | **76.4×** |
| 7 | 1 346 s | 70 000 | 23 / 23 | 1 021.2 µs | 13.4 µs | **76.2×** |
| 8 | 1 541 s | 80 000 | 26 / 26 | 1 022.4 µs | 13.5 µs | **75.9×** |
| 9 | 20 820 s | 910 000 | 307 / 307 | 981.1 µs | 13.1 µs | **74.9×** |
| 10 | 21 067 s | 920 000 | 309 / 309 | 982.0 µs | 13.1 µs | **74.9×** |
| 11 | 21 317 s | 930 000 | 310 / 310 | 982.1 µs | 13.1 µs | **74.9×** |
| 12 | 31 916 s | 1 360 000 | 476 / 476 | 980.0 µs | 13.3 µs | **73.9×** |
| 13 | 32 174 s | 1 370 000 | 479 / 479 | 980.1 µs | 13.3 µs | **73.9×** |
| 14 | 59 788 s | 2 470 000 | 863 / 863 | 973.0 µs | 13.4 µs | **72.4×** |
| 15 | 59 984 s | 2 480 000 | 865 / 865 | 973.2 µs | 13.4 µs | **72.4×** |
| 16 | 60 180 s | 2 490 000 | 869 / 869 | 973.4 µs | 13.4 µs | **72.4×** |
| 17 | 78 884 s | 3 270 000 | 1 175 / 1 175 | 982.0 µs | 13.5 µs | **72.7×** |
| 18 | 84 376 s | 3 650 000 | 1 327 / 1 327 | 984.2 µs | 13.5 µs | **72.8×** |
| 19 | 86 477 s | 3 750 000 | 1 359 / 1 359 | 985.0 µs | 13.5 µs | **72.9×** |

**Speedup variance across 19 checkpoints over 24h : 72.4× – 77.1× (≈ 6%)**
— speedup is structural, not a cache or warm-up artefact.

**Cumulative withdraws captured : 1 359 real on-chain events, 0 missed.**

---

## What this proves

1. **Correctness at scale** — Pipeline B detected exactly the same set of
   withdraws as Pipeline A across all 19 checkpoints over 24 hours of continuous run. `drift = 0` maintained on 1 359 real rug
   events. No false negatives at any point.

2. **Sustained performance** — A ~72-77× sustained speedup over 24 hours
   means Brain-Shield can run on a fraction of the hardware required by a
   naïve implementation, or reserve the saved compute budget for
   downstream protection logic (alerting, auto-exit, risk scoring,
   ML inference).

3. **Production-ready stability** — **24 hours of continuous run, 3.75
   million entries** processed, no degradation, no memory growth observed,
   stable 1.4% CPU footprint. The system is ready to be deployed
   long-running on a single inexpensive VPS.

4. **Detection coverage** — 1 359 real withdrawals captured in 24 hours
   (≈ 57 / hour, ≈ 1 every 64 seconds) shows the underlying market
   opportunity: liquidity withdrawals are frequent enough on pump.fun AMM
   that a real-time protection layer is materially useful.

---

## On-chain verification

A captured withdrawal can be verified directly on Solscan to validate that
the extracted fields (pool, mint, signer) correspond to a real liquidity
removal on Pump.fun AMM.

Example captured event (slot 417 822 435):

- **Transaction signature** :
  `5aFJw4rNWMrhUQmw1NzrduC6Fnvaiy8ZwXBx7MUxC6ZnUxyttdt2qEDygBoYdkuEB8K88fQ8C1a5VepgCrKftgrM`
- **Solscan link** :
  https://solscan.io/tx/5aFJw4rNWMrhUQmw1NzrduC6Fnvaiy8ZwXBx7MUxC6ZnUxyttdt2qEDygBoYdkuEB8K88fQ8C1a5VepgCrKftgrM
- **Action** : Withdraw 133.43 WSOL + 482 346 quote tokens from Pump.fun AMM
  pool
- **Pool, signer, base mint, quote mint** all match Brain-Shield's extracted
  output exactly.

Public, third-party-verifiable validation that the detection output is
semantically correct and corresponds to real on-chain liquidity events.

---

## Methodology (high level)

The benchmark is structured as a single-process A/B comparison:

1. The same live ShredStream entries are routed to both Pipeline A and
   Pipeline B in sequence.
2. Each pipeline produces an independent set of detections and an
   independent timing measurement.
3. Detection sets are diffed at every checkpoint (every 10 000 entries) —
   any divergence (`W_A ≠ W_B`) would surface immediately as a "DRIFT
   DETECTED" event.
4. Per-entry timing is averaged over the cumulative window using the
   monotonic clock.

Both pipelines share the same input feed, the same Solana parsing
primitives, and the same withdrawal-extraction logic. The only difference
is the **processing strategy** chosen by Pipeline B — described in the
project's private design documentation.

---

## Screenshots

Raw, unedited terminal screenshots showing the bench running live are
available in the `screenshots/` directory.

Each screenshot displays the A/B comparison table at one or more
checkpoints, including:
- Entries processed (A and B)
- Withdraws detected (A and B)
- Skipped count (B only)
- Average per-entry time (A and B)
- Relative CPU usage (A and B)
- Speedup ratio
- False-negative count (drift)
- Bundle-sell count (control metric)

---

## Reproducibility

The bench is reproducible by anyone running a Solana ShredStream relay
client. Numbers may vary slightly with relay distance and VPS specs, but
the **relative speedup ratio between Pipeline A and Pipeline B is
invariant** since both pipelines process the same entries in the same
process.

Source code of the harness will be released to grant evaluators under
appropriate confidentiality arrangements.

---

*Document generated from a live 24h continuous bench run on local hardware — May 6, 2026.*

---

## Roadmap — Layers 2 & 3

The same ShredStream pipeline that produces Layer 1 detections will be
extended in two further layers:

### Layer 2 — Coordinated Dump Detection
A coordinated-dump heuristic runs in parallel inside the same bench
process : N distinct wallets selling the same mint in the same blockhash.
**Over the 24h run, 2 507 coordinated-dump events were captured**,
in addition to the 1 359 LP rugs — including multiple **TIER ≥15**
captures (15+ distinct wallets sharing the same blockhash on the same
mint, in the same Solana slot, mathematically impossible by chance).

See **`PITCH_L2.md`** for four detailed case studies including the
strongest captures, with terminal alerts and matching GMGN price-collapse
charts that are independently verifiable on Solscan.

### Layer 3 — Cartel Operation Detection
A pre-existing offline component of the project has indexed **432
historical cartels** of coordinated wallets from Solana memecoin trade
data, and validated their predictive value on a hold-out set
(**95% WR on 101 alerted migrations**, offline backtest).

Layer 3 ships the cartel database and the matching logic in the
ShredStream pipeline so cartels are detected as they enter a token,
not after the fact. Implementation details and the cartel database are
part of the project's private IP.
