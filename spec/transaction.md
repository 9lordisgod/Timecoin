# Transaction format, version 2 (stub)

**Status: stub.** Source of record: whitepaper §2 (Figure 2) and §9 (Listing 2),
[`whitepaper/timecoin-whitepaper.pdf`](../whitepaper/timecoin-whitepaper.pdf).
Items marked *open* are not in the paper and need a design issue before they
are normative.

## Goals (paper §1–2)

1. Keep Nakamoto's transaction model: a coin is a chain of digital signatures;
   each spend signs a hash of the previous transaction and the next owner's
   lock. Only the signature algorithm changes.
2. One transaction version from genesis: **version 2**. There is no version-1
   transaction on Timechain and no serialization compatibility with Bitcoin.
3. The signed message commits to an **algorithm identifier** and to the
   **genesis hash**, so a lattice signature cannot be replayed into a
   hash-based program and a Timechain transaction cannot be replayed onto
   Bitcoin or any other chain.

## Structure (paper Figure 2)

```
tx_v2 {
  version: 2
  inputs[]  { prevout, sequence }
  outputs[] { value, witness_program }     // 32-byte SHA-256 of script tree
  witness[] { leaf_tag, pubkey, signature, partner_hash }
  locktime
}
```

Field notes:

| Field | Meaning |
|-------|---------|
| `version` | MUST be 2. Other values are invalid. |
| `prevout` | Transaction identifier and output index being spent. |
| `sequence` | As in Bitcoin; used by relative locktime and median-time-past rules, which the paper keeps unchanged (§3). |
| `value` | Amount in the smallest unit. *Open:* unit name and precision. |
| `witness_program` | 32 bytes: `SHA-256(root)` of the two-leaf script tree described in [`spec/addresses.md`](addresses.md). |
| `leaf_tag` | Which leaf is being spent: ML-DSA-65 (ordinary) or SLH-DSA-SHA2-128s (recovery). Exactly one tag per spend. |
| `pubkey` | The public key of the revealed leaf: 1,952 B (ML-DSA-65) or 32 B (SLH-DSA-SHA2-128s). |
| `signature` | 3,309 B (ML-DSA-65) or 7,856 B (SLH-DSA-SHA2-128s). |
| `partner_hash` | The 32-byte sibling leaf hash that reconnects the revealed leaf to the witness program. |
| `locktime` | As in Bitcoin. |

The witness is the only place a public key appears on chain (paper Table 1).

## Signature hash (paper Figure 2)

```
sighash = SHA-256d( version | prevouts | outputs | locktime
                    | input_index | algorithm_id | genesis_hash )
```

Domain separation comes from two committed fields:

- `algorithm_id` — identifies the verifier the signature is for. A signature
  produced for the ML-DSA-65 leaf cannot verify under the SLH-DSA leaf or vice
  versa, even for the same transaction (§2).
- `genesis_hash` — the hash of the Timechain genesis block. A signature over a
  Timechain transaction is meaningless on Bitcoin or any other chain, and a
  signature from another chain is meaningless here (§2). The genesis block is
  not generated in this repository, so this value is not yet known.

The signer signs the 32-byte `sighash` directly. Each input is signed with its
own `input_index`, so one witness cannot be moved to another input.

## Consensus verification (paper Listing 2)

```
verify(tx, witness):
    leaf = tagged_hash(witness.tag, witness.pubkey)
    root = merkle_pair(leaf, witness.partner)
    require SHA-256(root) == tx.input.witness_program
    if witness.tag == ML-DSA-65:
        require ml_dsa_verify(witness.pubkey, sighash, witness.sig)
    else if witness.tag == SLH-DSA-128s:
        require slh_dsa_verify(witness.pubkey, sighash, witness.sig)
    else reject
```

Exactly one algorithm tag is accepted per spend. Both verifiers are consensus
rules from the genesis block; using the recovery leaf needs no soft fork (§9).
Implementations must call a reviewed library for both verifiers, not a private
reimplementation (§12).

## Block commitment and weight (paper §3, §9)

- The block header's Merkle root commits to **witness transaction identifiers**
  (wtxids), so a valid signature is inside the proof-of-work.
- A base block is limited to **1,000,000 bytes**. Witness bytes are counted at
  **one quarter** (the SegWit discount), so a 3.3 KB signature does not price a
  payment out of a block and a recovery spend remains possible.
- Nodes may prune witnesses once the output is spent and the block is buried,
  keeping headers and the UTXO set (§7). Lightweight clients fetch the witness
  on first confirmation and may drop it after depth (§8).

## Open refinements

Not in the paper. Each needs a design issue.

- **Exact serialization**: endianness, compact-size encodings, and the byte
  layout of `prevouts` and `outputs` inside the sighash preimage.
- **Transaction identifiers**: the paper defines the Merkle root over wtxids
  but does not separately define the identifier used in `prevout`. Proposal: a
  `txid` over the non-witness fields so that signature re-randomization does
  not change how an output is referenced.
- **Sighash coverage**: the paper's preimage commits to `prevouts`, `outputs`,
  and `locktime`. Whether to also commit to input amounts, the spent
  `witness_program`s, and `sequence` values (BIP-143 / BIP-341 style, which
  lets offline signers skip fetching previous transactions) is open.
- **Sighash types**: the paper describes a single mode equivalent to ALL.
  NONE, SINGLE, and ANYONECANPAY are not specified.
- **`algorithm_id` and `leaf_tag` constants**: byte values and whether they
  are the same field.
- **FIPS context strings**: FIPS 204 and FIPS 205 accept a `ctx` input (up to
  255 bytes). Using a fixed string such as `TIC/tx/v2` there would add a second
  layer of domain separation at no on-chain cost.
- **Hedged vs. deterministic signing**: FIPS 204 permits both and verification
  is identical, so consensus accepts either. Hedged signing produces different
  valid signatures for the same message, so `wtxid` is malleable by the signer;
  relay and compact-block effects need analysis.
- **Maximum block weight**: the paper gives the base limit and the discount;
  the resulting weight ceiling (4,000,000 weight units if Bitcoin's formula is
  used) and the inputs-per-block estimate should be stated explicitly
  (see [`whitepaper/README.md`](../whitepaper/README.md), errata 4).
- **Coinbase transaction** layout. The genesis coinbase is unspendable and is
  not in the UTXO set (§6).
- **Standardness**: maximum inputs and maximum witness bytes per transaction.
