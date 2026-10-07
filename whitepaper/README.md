# Whitepaper

**Timecoin: A Peer-to-Peer Electronic Cash System with Post-Quantum Authorization**
Williem Doe — <jsonrpc@icloud.com>

| File | Language | Pages | SHA-256 |
|------|----------|------:|---------|
| [`timecoin-whitepaper.pdf`](timecoin-whitepaper.pdf) | English | 11 | `7cfd4731572c64d93ff327bdec8057b28797feb221224d674ef32a7b326541dd` |
| [`timecoin-whitepaper-zh.pdf`](timecoin-whitepaper-zh.pdf) | 中文 (时间币：一种具备后量子授权的点对点电子现金系统) | 11 | `7716791398c2d8b5d4569fc06a156a15d3eddf2d544716e4b2d6dc55329c001e` |

Both versions are dated 2026-10-06 and carry the same parameters. The paper
is the source of record for the design; the Markdown documents in
[`docs/`](../docs) and [`spec/`](../spec) restate and elaborate it. Where
they disagree, the paper wins until a design issue changes the paper.

The paper describes a design. It does not report a launched mainnet, a traded
market, or an audit (§12).

## Abstract

> A purely peer-to-peer version of electronic cash would allow online payments
> to be sent directly from one party to another without going through a
> financial institution, and would remain sound after a cryptographically
> relevant quantum computer exists. Digital signatures provide part of the
> solution, but the elliptic-curve signatures used by Bitcoin are not expected
> to survive Shor's algorithm. We propose Timecoin, a unit of account on a new
> chain called Timechain. The chain is a code descendant of Bitcoin's
> transaction and proof-of-work design, with its own genesis block and no
> shared history. Ordinary authorization is an ML-DSA-65 signature under NIST
> FIPS 204. Each address also commits to an SLH-DSA recovery path under FIPS
> 205, whose only hardness assumption is a cryptographic hash. Proof-of-work
> remains double SHA-256. Grover's algorithm gives only a quadratic speedup
> against that hash, so the mining function is not the first thing that has to
> change. The familiar incentives, the Merkle tree, and the longest-chain rule
> are kept. What changes is the lock on the coin.

## Sections

| § | Title | Restated in |
|---|-------|-------------|
| 1 | Introduction | [`README.md`](../README.md) |
| 2 | Transactions | [`spec/transaction.md`](../spec/transaction.md) |
| 3 | Timechain | [`docs/parameters.md`](../docs/parameters.md) |
| 4 | Proof-of-Work | [`docs/parameters.md`](../docs/parameters.md) |
| 5 | Network | [`docs/parameters.md`](../docs/parameters.md) |
| 6 | Incentive | [`docs/parameters.md`](../docs/parameters.md) |
| 7 | Reclaiming Disk Space | [`docs/parameters.md`](../docs/parameters.md) |
| 8 | Simplified Payment Verification | [`docs/threat-model.md`](../docs/threat-model.md) |
| 9 | Post-Quantum Authorization | [`spec/addresses.md`](../spec/addresses.md), [`docs/threat-model.md`](../docs/threat-model.md) |
| 10 | Privacy | [`spec/addresses.md`](../spec/addresses.md) |
| 11 | Calculations | [`docs/threat-model.md`](../docs/threat-model.md) |
| 12 | Conclusion | [`README.md`](../README.md) |

## Errata and open points found while transcribing

Noted here so they can be fixed in a revision of the paper. None of them
changes a consensus parameter. Each is a candidate design issue.

1. **§7, header storage.** "At 60-second blocks that is about 4.2 MB a year."
   4.2 MB/year is the figure for Bitcoin's 600-second interval
   (52,560 headers × 80 B). At 60 seconds it is 525,600 headers × 80 B
   ≈ **42 MB/year**. The conclusion that headers remain cheap to keep still
   holds. Present in both language versions.
2. **§4, retarget period.** "Difficulty adjusts every 1,008 blocks, about a
   day." 1,008 blocks × 60 s = **16.8 hours**. Either the wording or the block
   count may want adjusting (1,440 blocks would be one day).
3. **§11, Table 2.** The `q = 0.10` row matches the Poisson expression printed
   in the section (and Nakamoto's original table). The rows for
   `q = 0.15, 0.20, 0.30, 0.40` do not; for example at `q = 0.30, z = 10` the
   expression gives ≈ 0.0417 and the table gives 0.0306, and at
   `q = 0.40, z = 10` the expression gives ≈ 0.360 and the table gives 0.191.
   The qualitative conclusion (exponential decay in `z`, independent of wall
   time) is unaffected. Recommend recomputing the table from the printed
   expression.
4. **§9, capacity sentence.** "A simple ML-DSA payment is on the order of 100
   ordinary inputs a block after the discount" does not pin down a number.
   An explicit weight budget and inputs-per-block calculation belongs in
   [`spec/transaction.md`](../spec/transaction.md) once the maximum block
   weight is fixed.
