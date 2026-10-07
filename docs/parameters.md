# Consensus and monetary parameters

**Status: draft.** Source of record: whitepaper §3–§7 and Table 1,
[`whitepaper/timecoin-whitepaper.pdf`](../whitepaper/timecoin-whitepaper.pdf).
Every value is a design-stage parameter, not a constant of a launched network.
Nothing has been mined, deployed, or committed to.

## Summary

| Parameter | Value | Paper § |
|-----------|-------|:-------:|
| Chain | Timechain. New genesis, no shared history. A code descendant of Bitcoin's transaction and proof-of-work design, not a fork of its ledger or UTXO set. | 1 |
| Currency | Timecoin, ticker **TIC**. TIC is not BTC. | 1 |
| Ordinary spend signature | **ML-DSA-65** (NIST FIPS 204), module-lattice-based | 9 |
| Recovery signature | **SLH-DSA-SHA2-128s** (NIST FIPS 205), stateless hash-based | 9 |
| Proof-of-work hash | **SHA-256d** over the 80-byte header | 3, 4 |
| Target block interval | **60 seconds** | 4 |
| Difficulty adjustment | Every **1,008 blocks**, clamped to a **factor of four** per adjustment | 4 |
| Chain selection | Greatest cumulative work | 4 |
| Supply cap | **21,000,000 TIC** | 6 |
| Initial block subsidy | **5 TIC** | 6 |
| Halving interval | **2,100,000 blocks** | 6 |
| Genesis coinbase | **Unspendable**; not in the UTXO set. Genesis block not yet generated. | 1, 6 |
| Premine / treasury | **None.** No founder allocation, no treasury output. Miners receive subsidy plus fees. | 6 |
| Base block size | **1,000,000 bytes** | 9 |
| Witness weighting | Witness bytes counted at **one quarter** | 9 |
| Merkle root | Commits to **witness transaction identifiers** (wtxids) | 3 |
| Time rules | Median-time-past unchanged from Bitcoin; a block time may not be more than **2 hours** ahead of network-adjusted time | 3 |
| Address format | bech32m, `tic1…` | 9 |
| Network | New message-start magic and new port; Bitcoin seeds not used; peers speaking Bitcoin's magic are disconnected | 5 |

## Block header (paper §3)

80 bytes in the Bitcoin layout: version, previous-block hash, Merkle root,
Unix time, compact target, nonce. The Merkle root is over wtxids, so a valid
signature is inside the proof-of-work.

## Proof-of-work (paper §4)

Scan for a nonce such that `SHA-256d(header)` begins with the required number
of zero bits. SHA-256 is retained on purpose: Grover's algorithm gives a
quadratic, not exponential, speedup, and doubling the zero-bit requirement
restores the classical work against a Grover adversary. Replacing the hash
would discard every existing miner and tested header implementation for a
problem that is not the binding constraint. See
[`docs/threat-model.md`](threat-model.md) §2.

Difficulty retargets every 1,008 blocks with Bitcoin's four-times clamp, so one
bad interval cannot move the target by more than 4× in either direction.

## Monetary schedule (paper §6)

Subsidy at height `h`:

```
epoch(h)   = floor(h / 2,100,000)
subsidy(h) = 5 TIC / 2^epoch(h)        (integer division in the smallest unit; rounds down)
```

| Epoch | Heights | Subsidy per block | Issued in epoch | Cumulative |
|------:|---------|------------------:|----------------:|-----------:|
| 0 | 0 – 2,099,999 | 5 TIC | 10,500,000 | 10,500,000 |
| 1 | 2,100,000 – 4,199,999 | 2.5 TIC | 5,250,000 | 15,750,000 |
| 2 | 4,200,000 – 6,299,999 | 1.25 TIC | 2,625,000 | 18,375,000 |
| 3 | 6,300,000 – 8,399,999 | 0.625 TIC | 1,312,500 | 19,687,500 |
| 4 | 8,400,000 – 10,499,999 | 0.3125 TIC | 656,250 | 20,343,750 |
| … | … | halves each epoch | … | → 21,000,000 |

`5 × 2,100,000 × (1 + 1/2 + 1/4 + …) = 21,000,000`: the same cap as Bitcoin,
reached on a faster clock. Because the subsidy rounds down in the smallest
unit, the total actually issued is strictly below the cap, as in Bitcoin.

The genesis output (height 0) is not in the UTXO set, so the maximum
*spendable* supply is strictly below 21,000,000 TIC. Issuance begins with the
first mined subsidy. Once the subsidy is exhausted the incentive is fees only.

## Derived values

From the 60-second target. Real timing depends on hashrate and the retarget.

| Quantity | Value |
|----------|-------|
| Blocks per hour / day | 60 / 1,440 |
| Blocks per year (365 d) | 525,600 |
| Retarget period | 1,008 blocks = 16.8 hours at target (see whitepaper errata 2) |
| Halving interval | 2,100,000 blocks ≈ 1,458 days ≈ 4 years |
| Six confirmations | 6 minutes. A receiver who wants Bitcoin's one-hour wait waits 60 blocks (§11). |
| Header storage | 80 B × 525,600 ≈ **42 MB/year** (see whitepaper errata 1) |

Relative to Bitcoin: 10× the blocks per epoch, 1/10 the subsidy per block,
half the retarget block count, same ≈ 4-year halving cadence, same 21,000,000
cap.

## Signature schemes (paper §9)

Sizes from FIPS 204 Table 2 and FIPS 205 Table 2.

| Scheme | Role | Public key | Signature | NIST category |
|--------|------|-----------:|----------:|:-------------:|
| ML-DSA-65 | Ordinary key path | 1,952 B | 3,309 B | 3 |
| SLH-DSA-SHA2-128s | Recovery leaf | 32 B | 7,856 B | 1 |

SLH-DSA-SHA2-128s is slow to produce and acceptable because it is not the
common case. Both verifiers are consensus rules from genesis. How an address
commits to both is in [`spec/addresses.md`](../spec/addresses.md).

## Storage (paper §7–§8)

Witnesses are the new cost: an ML-DSA-65 spend is a few kilobytes, not
64 bytes. Pruning of spent witnesses is part of the design, not an
optimization: a node that has validated a witness may drop it once the output
is spent and the block is buried, keeping the header and the UTXO set.
Lightweight clients must fetch the witness on first confirmation, since a
header proves inclusion, not that the signature was post-quantum.

## Open parameters

Not fixed by the paper. Each is a candidate design issue.

- Smallest unit: name and precision. If TIC uses 10⁸ subunits like Bitcoin,
  the subsidy reaches zero after 29 halvings (height 60,900,000, ≈ 116 years).
- Maximum block weight as a single number (4,000,000 weight units if Bitcoin's
  `base × 4 + witness` formula is used), and inputs-per-block capacity.
- Message-start magic bytes and default port values.
- Coinbase maturity.
- Witness version for addresses; testnet/regtest parameters and prefixes.
- Genesis block (timestamp, message, `nBits`): deliberately **not** generated
  in this repository.
