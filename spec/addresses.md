# Addresses (stub)

**Status: stub.** Source of record: whitepaper §2, §9 (Figure 9), §10 and
Table 1, [`whitepaper/timecoin-whitepaper.pdf`](../whitepaper/timecoin-whitepaper.pdf).
Items marked *open* are not in the paper.

## Encoding: bech32m, human-readable part `tic`

- Addresses are **bech32m** (BIP-350). The paper writes them as `tic1…`
  (Table 1), the Timechain counterpart of Bitcoin's `bc1…`.
- Human-readable part: **`tic`** on mainnet.
- Data part: a witness version followed by the 32-byte witness program
  converted to 5-bit groups, then the 6-character bech32m checksum.
- Length: `tic` (3) + `1` (1) + version (1) + 52 data characters + 6 checksum
  = **63 characters**.
- There is no legacy base58 format and no bech32 (BIP-173) variant. Every
  Timechain address uses the bech32m checksum.

*Open:* the witness version number. Starting at version 1 would let unmodified
BIP-350 decoders (which require bech32, not bech32m, for version 0) accept the
format; version-1 addresses would begin `tic1p`. *Open:* human-readable parts
for testnet and regtest.

## Witness program: a two-leaf script tree (paper §2, §9)

An output locks a 32-byte **witness program**. The program is the SHA-256 hash
of a two-leaf script tree:

```
leaf_key      = tagged_hash(ML-DSA-65 tag,         mldsa65_pubkey)      // ordinary path
leaf_recovery = tagged_hash(SLH-DSA-SHA2-128s tag, slhdsa128s_pubkey)   // recovery path
root          = merkle_pair(leaf_key, leaf_recovery)
witness_program = SHA-256(root)
```

- The **key-path leaf** is an ML-DSA-65 public key (FIPS 204).
- The **recovery leaf** is an SLH-DSA-SHA2-128s public key (FIPS 205).
- Neither public key is revealed by the address. A spend reveals only the leaf
  it uses: `leaf_tag`, that leaf's public key, its signature, and the
  `partner_hash` of the other leaf, which reconnects the revealed leaf to the
  witness program. Verification is in [`spec/transaction.md`](transaction.md).

This is not a hybrid signature: the common payment carries only the ML-DSA-65
signature, never the 7,856-byte hash-based one. The tree gives crypto-agility
instead. If module lattices are broken, or credibly attacked, wallets spend by
the recovery leaf into a new address whose key path is itself hash-based, or a
later standardized algorithm activated by flag day (§9).

Every address commits to **both** leaves. *Open:* whether an output with a
single leaf is ever valid.

## Spend paths

| Path | Leaf | Revealed in witness | Signature |
|------|------|---------------------|-----------|
| Ordinary | ML-DSA-65 | 1,952 B public key + 32 B `partner_hash` | 3,309 B |
| Recovery | SLH-DSA-SHA2-128s | 32 B public key + 32 B `partner_hash` | 7,856 B |

## Key hygiene and derivation (paper §10)

- A **new key-path key and a new recovery key for every output**. An opened
  ML-DSA public key is 1,952 bytes of identifying material, and a reused
  address republishes a key that an adversary can file for the day a quantum
  computer arrives.
- Key derivation is **not** BIP-32 elliptic-curve tweaking. A chain code feeds
  SHA-256 into the ML-DSA and SLH-DSA key generators, and derivation stays off
  chain.
- Wallets do not reuse addresses. Multi-input transactions still link their
  inputs to one owner, as on Bitcoin.

*Open:* the exact derivation function (chain code → seed → `ML-DSA.KeyGen` /
`SLH-DSA.KeyGen`), its path structure, and how a wallet backs up both key
families from one secret. Any such document must contain no example seeds.

## Open items

- Witness version value and testnet/regtest human-readable parts (above).
- Constants for the two leaf tags and the `merkle_pair` rule (fixed order
  key-then-recovery, or lexicographic as in BIP-341).
- Whether `tagged_hash` here is the BIP-340 construction
  `SHA-256(SHA-256(tag) || SHA-256(tag) || msg)` or a different tagging rule.
- Test vectors, generated from published FIPS 204 / FIPS 205 test keys only.
- Payment-request and URI format.
