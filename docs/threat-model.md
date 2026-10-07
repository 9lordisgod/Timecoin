# Threat model (stub)

**Status: stub.** Source of record: whitepaper §1, §4, §8, §9 and §11,
[`whitepaper/timecoin-whitepaper.pdf`](../whitepaper/timecoin-whitepaper.pdf).
Headings are placeholders for fuller analysis. Signature and key sizes are
from FIPS 204 and FIPS 205.

## Scope

Adversary: every classical capability (hashrate, network position, mempool
observation, full node) plus, at an unknown future date, a cryptographically
relevant quantum computer (CRQC). Goals: steal funds, double-spend, censor,
rewrite history.

The paper's framing (§1): the urgent defect in Bitcoin's design is the
signature, not the timestamp server. Grover against SHA-256 is "a different
and smaller problem"; Shor against secp256k1 is the one that forges spends.

Out of scope for this stub: eclipse and DoS attacks, wallet software, mining
economics, and side channels in signing implementations. They matter; they are
not quantum-specific.

## 1. Shor against secp256k1

- Shor's algorithm solves the discrete-logarithm problem on secp256k1 in
  polynomial time on a sufficiently large quantum computer. A revealed public
  key — in a spent output, a reused address, or an old pay-to-public-key output
  — is then enough to forge a spend (§1).
- Bitcoin migration drafts must carry an installed base, a miner vote, and
  coins whose keys are already public. Timechain does not: it publishes the
  rules before the first block (§1).
- **Timechain response:** no secp256k1 in consensus. Ordinary spends are
  ML-DSA-65; the recovery leaf is SLH-DSA-SHA2-128s, whose only hardness
  assumption is a cryptographic hash (§9).
- **Residual risk:** module-lattice assumptions are younger than hash-function
  assumptions. A break, or a credible attack, moves wallets to the recovery
  leaf, spending into new addresses whose key path is itself hash-based or a
  later standardized algorithm activated by flag day. Both verifiers exist
  from genesis, so this needs no soft fork (§9).

## 2. Grover against SHA-256

- Grover's algorithm square-roots an unstructured search: SHA-256 preimage
  resistance falls from 2²⁵⁶ to about 2¹²⁸ queries. It is a quadratic
  speedup, not an exponential one (§1, §4).
- **Timechain response:** keep SHA-256d. Doubling the zero-bit requirement
  restores the classical work against a Grover adversary, and difficulty
  already adjusts to hashrate. Replacing the hash would discard every existing
  miner and tested header implementation for a problem that is not the binding
  constraint (§4). 256-bit outputs keep ≥ 128-bit post-quantum preimage
  security for the witness program, txids, and Merkle roots.
- A quantum attacker who has not broken SHA-256 still has to win the
  cumulative-work race in §4 below (§11).
- **Open:** quantified centralization risk from a quantum miner's speedup at a
  60-second interval (fewer coherent Grover iterations per block than at
  600 seconds); cf. Aggarwal, Brennen, Lee, Santha, Tomamichel, *Ledger* 3
  (2018), paper ref. [9].

## 3. Signature size

| Scheme | Public key | Signature | Per-input witness (approx.) |
|--------|-----------:|----------:|----------------------------:|
| ECDSA secp256k1 (Bitcoin, comparison) | 33 B | 71–72 B | ~105 B |
| Schnorr secp256k1 / BIP-340 (comparison) | 32 B, in output | 64 B | ~64 B |
| **ML-DSA-65** (ordinary) | 1,952 B | 3,309 B | ~5.3 KB + 32 B partner hash |
| **SLH-DSA-SHA2-128s** (recovery) | 32 B | 7,856 B | ~7.9 KB + 32 B partner hash |

- Roughly 50× the witness bytes per input versus ECDSA. The paper accepts this
  as a deliberate capacity cut, taken in exchange for a lock Shor's algorithm
  does not break (§9).
- Mitigations in the design: witness bytes weighted at one quarter so a
  3.3 KB signature does not price a payment out of the block and a recovery
  spend remains possible (§9); the common payment never carries the 7.9 KB
  hash-based signature (§9); spent witnesses are prunable once buried (§7);
  public keys are committed by hash, so unspent outputs reveal nothing (§9).
- No practical aggregation exists for ML-DSA; every input carries a full
  signature.
- **Open:** explicit maximum weight and inputs-per-block figure
  ([`whitepaper/README.md`](../whitepaper/README.md), errata 4); standardness
  caps on inputs per transaction; whether recovery spends get a different
  weight.

## 4. Mempool race after a key is revealed

The paper separates two exposures (§9):

- **Long-range theft** of coins sitting in untouched outputs whose keys are
  already on chain — the exposure identified for Bitcoin in ref. [9]. Closed:
  Timechain public keys stay hidden until the spend, and every output uses
  fresh keys (§9, §10).
- **Short-range race** against a spend already in the mempool. Once a key is
  revealed, an adversary who can forge during the confirmation window can
  broadcast a conflicting, higher-fee spend of the same output. Under current
  assumptions ML-DSA-65 cannot be forged, so revealing the key does not open
  this race; it becomes live only if ML-DSA is broken.
- **Mitigation in the paper is depth.** Receivers of large payments wait.
  Wallets never reuse an address, because reuse republishes a key that has
  already been opened (§9). Six confirmations are six minutes; a merchant who
  wants Bitcoin's one-hour wait waits 60 blocks (§11).
- A quantum attacker who has broken ML-DSA and sees a key in the mempool is
  "forging, not out-hashing": he is outside the catch-up table below. That
  case is why the recovery leaf exists and why a large receipt waits for depth
  even after the first signature verified (§11).
- **Open:** whether consensus should give the recovery leaf precedence over a
  forged key-path spend during a declared lattice break (e.g. a relative
  timelock on key-path spends, or a flag-day "recovery mode"), or whether
  depth and wallet policy are sufficient. The paper takes the latter position.

## 5. Classical reorganization race (paper §11)

Nakamoto's gambler's-ruin analysis is unchanged. With honest fraction `p` and
attacker fraction `q`, the probability an attacker ever catches up from `z`
blocks behind is `(q/p)^z` for `p > q`, and the probability of still catching
up after the receiver has waited `z` confirmations is

```
1 − Σ_{k=0..z} [ λ^k e^{−λ} / k! ] × [ 1 − (q/p)^{z−k} ],   λ = z·q/p
```

The decay in `z` depends on relative hashrate, not wall time, so `z` is a
count of blocks: the same confidence arrives ten times sooner than on Bitcoin.
At `q = 0.10`, after six blocks `P < 0.001`.

Note: the paper's Table 2 rows for `q ≥ 0.15` do not match this expression;
see [`whitepaper/README.md`](../whitepaper/README.md), errata 3.

## 6. Lightweight clients (paper §8)

A header proves inclusion, not that the signature was post-quantum. An SPV
client must fetch the witness on first confirmation and may drop it after
several blocks, since reversing it then requires redoing the work. Businesses
receiving frequent payments should run full nodes.
