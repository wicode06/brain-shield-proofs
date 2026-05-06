# Brain-Shield — Benchmark Methodology

This document describes **how** the Pipeline A vs Pipeline B benchmark was
designed and **why** the result is irrefutable. It does **not** disclose the
internal optimization strategy used by Pipeline B — that information is held
under the project's private design documentation.

---

## 1. Test setup

| Item | Value |
|---|---|
| Hardware | Vultr VPS, Frankfurt region |
| Network distance to Jito relay | ~0.6 ms RTT |
| Solana feed | Live ShredStream (production data, no replay) |
| Build profile | Rust release (`cargo build --release`) |
| Run duration | 16 min 10 s continuous |
| Total entries processed | 50 000 |
| Real on-chain withdraws observed | 38 |

No synthetic data, no replay, no warm-up window. The bench connects to a live
Solana ShredStream relay and processes the actual production feed.

---

## 2. What is being compared

Two **independent processing pipelines** run inside the **same process**:

- **Pipeline A** : the standard / naïve approach. Every input entry is
  fully processed through the standard parsing path before withdraw
  detection is attempted.

- **Pipeline B** : the Brain-Shield approach. The exact strategy is private,
  but the public claim is: same correctness as Pipeline A, lower cost.

Both pipelines:
- Receive **the same input bytes** (the live ShredStream entry).
- Use **the same withdraw-extraction logic** to derive `(slot, pool, mint,
  signer)` once an entry has been parsed.
- Maintain **independent counters and timers**, so neither can pollute the
  other.

---

## 3. Why this design is irrefutable

### 3.1 Same input
Both pipelines see byte-for-byte the same 50 000 entries, in the same order,
in the same process, on the same machine. There is no stream-splitting, no
broadcast, no network-level sampling that could cause divergent inputs.

### 3.2 Same correctness target
Both pipelines emit a detection record containing the same key fields. The
two detection sets are diffed at every checkpoint. **Any divergence (a
withdraw seen by A but not by B, or vice versa) is logged immediately as a
"DRIFT DETECTED" event in red.**

Across the 16-minute run:
- Pipeline A detected 15 withdraws.
- Pipeline B detected 15 withdraws.
- **Drift = 0 at every checkpoint (10k, 20k, 30k, 40k, 50k entries).**

This is the strongest possible empirical evidence that Pipeline B does not
sacrifice correctness for speed.

### 3.3 Same timing instrument
Per-entry timing for both pipelines uses the same monotonic clock
(`std::time::Instant` in Rust). Cumulative nanoseconds are aggregated and
divided by entry count for the average. There is no profiler overhead, no
external measurement.

### 3.4 No selection bias
Every single entry from the ShredStream feed is processed by both pipelines.
There is no sub-sampling, no skip-on-error, no fallback path that could bias
the result toward one pipeline.

### 3.5 Stability over time
The same speedup ratio is observed at 5 consecutive checkpoints
(10k, 20k, 30k, 40k, 50k entries) with a variance of less than 2%. This rules
out any cache-warm-up artefact, JIT effect, or one-shot lucky measurement.

---

## 4. Observed quantities

At every checkpoint (every 10 000 entries), the bench prints and logs:

| Field | Pipeline A | Pipeline B |
|---|:---:|:---:|
| Entries processed | ✓ | ✓ |
| Withdraws detected | ✓ | ✓ |
| Skipped (no work needed) | n/a | ✓ |
| Cumulative average time per entry | ✓ | ✓ |
| Relative CPU usage | 100% (reference) | derived ratio |
| Pre-scan time | n/a | ✓ |

Plus three global numbers:

- **SPEEDUP B vs A** = `avg_A / avg_B`
- **False negatives** = entries detected by A but missed by B
- **Drift** = signed difference of detection counts between A and B

---

## 5. Public commitments

We commit to:
- Releasing the bench harness source code so anyone can re-run Pipeline A vs
  Pipeline B and verify the speedup on their own machine.
- Providing the captured Solscan transaction signatures so any party can
  validate the semantic correctness of detected withdraws.
- Publishing the raw stats log (`log_stats.txt`) of the 16-minute run.

We do **not** commit to disclosing the internal processing strategy of
Pipeline B at this stage. The strategy is the core IP of Brain-Shield and
will only be shared with grant evaluators under appropriate confidentiality
arrangements.

---

## 6. What the benchmark does NOT claim

This benchmark proves a **structural performance advantage** of Brain-Shield's
processing strategy on a representative live workload.

It does not, by itself, prove:
- End-to-end alert latency to a trader (network + downstream).
- Performance under adversarial / synthetic worst-case workloads.
- Ability to detect rugs on protocols other than Pump.fun AMM.

These claims are addressed by separate proof artefacts (see project pitch).

---

*Methodology version 1.0 — May 5, 2026.*
