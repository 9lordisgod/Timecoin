# Timechain

Experimental Timechain / Timecoin (TIC) design. New genesis, not a Bitcoin ledger fork. Post-quantum signatures. Not a launched mainnet.

> **Status: design only.** Nothing in this repository runs a network, mines a block, or holds funds.

## Read this first

- Experimental design only. No mainnet, no exchange listing, no premine.
- Timechain does not share Bitcoin's genesis or UTXO set. TIC is not BTC.
- Ordinary spend path: ML-DSA-65 (NIST FIPS 204). Recovery path: SLH-DSA-SHA2-128s (NIST FIPS 205).
- Proof-of-work stays SHA-256d. Target block interval 60 seconds.
- Supply cap 21,000,000 TIC. Initial subsidy 5 TIC. Halving every 2,100,000 blocks.
- Genesis coinbase is unspendable.

## Owner / contact

Williem Doe — <jsonrpc@icloud.com>

## Whitepaper

*Timecoin: A Peer-to-Peer Electronic Cash System with Post-Quantum Authorization*

- English: [`whitepaper/timecoin-whitepaper.pdf`](whitepaper/timecoin-whitepaper.pdf)
- 中文: [`whitepaper/timecoin-whitepaper-zh.pdf`](whitepaper/timecoin-whitepaper-zh.pdf)
- Section map, checksums, and errata: [`whitepaper/README.md`](whitepaper/README.md)

The paper is the source of record. The Markdown in `docs/` and `spec/`
restates it section by section and marks anything the paper leaves open.

## What this is

Timechain is a code descendant of Bitcoin's transaction and proof-of-work
design with its own genesis block and no shared history. It keeps what known
quantum algorithms do not break — SHA-256d proof-of-work, the UTXO model, the
Merkle tree, the longest-chain rule, the fixed issuance schedule — and replaces
what Shor's algorithm does break: the secp256k1 lock on the coin.

Every address commits to a two-leaf script tree:

| Leaf | Scheme | Standard | Assumption | Role |
|------|--------|----------|------------|------|
| Key path | ML-DSA-65 | NIST FIPS 204 | Module lattices | Ordinary spend |
| Recovery | SLH-DSA-SHA2-128s | NIST FIPS 205 | Hash functions only | Fallback if lattices fail |

Neither key appears in the address; a spend reveals only the leaf it uses.
This is not a hybrid signature: the common payment carries one ML-DSA-65
signature. The recovery leaf exists so a break of the lattice assumption does
not strand the coin, and it needs no soft fork because both verifiers are
consensus rules from genesis.

The name *Timechain* is the one Satoshi used in early notes for the block
chain. It is reused as homage, not as a claim on Bitcoin's chain, ticker, or
genesis.

## What this is not

- **Not a fork of Bitcoin's ledger.** Timechain has its own genesis, port, and
  address family. No Bitcoin UTXO, balance, or history carries over. Holders
  of bitcoin receive nothing on Timechain.
- **Not a Bitcoin Core fork in this repository.** No Bitcoin Core source is
  vendored here.
- **Not running software.** There is no node, wallet, miner, genesis block, or
  CI that mines. Design documents only.
- **Not a launched asset.** No mainnet, no traded market, no listing, no sale,
  no audit, no premine, no founder allocation, no treasury output.

## Repository layout

| Path | Contents |
|------|----------|
| [`whitepaper/`](whitepaper/) | The whitepaper (English and Chinese PDFs) with section map and errata |
| [`docs/parameters.md`](docs/parameters.md) | Consensus and monetary parameters |
| [`docs/threat-model.md`](docs/threat-model.md) | Threat model (stub) |
| [`spec/transaction.md`](spec/transaction.md) | Version-2 transaction and domain-separated sighash (stub) |
| [`spec/addresses.md`](spec/addresses.md) | bech32m `tic1…` addresses and the two-leaf witness program (stub) |
| [`.github/ISSUE_TEMPLATE/design.md`](.github/ISSUE_TEMPLATE/design.md) | Template for design proposals |
| [`LICENSE`](LICENSE) | MIT |

## Standards referenced

- NIST FIPS 204 — Module-Lattice-Based Digital Signature Standard (ML-DSA)
- NIST FIPS 205 — Stateless Hash-Based Digital Signature Standard (SLH-DSA)
- NIST FIPS 180-4 — Secure Hash Standard (SHA-256)
- BIP-350 — bech32m address format

## Contributing to the design

Open an issue with the **Design proposal** template and name the whitepaper
section it touches. Discussion happens in issues; specification text lands in
`docs/` and `spec/` through pull requests. Everything in those directories is
a draft until stated otherwise. Implementations, when they exist, must call a
reviewed library for both signature verifiers rather than a private
reimplementation.

## Secrets policy

This repository must never contain private keys, seed phrases, mnemonics,
wallet files, RPC credentials, or any other secret material, including for test
networks. The [`.gitignore`](.gitignore) excludes common key and wallet file
patterns as a safety net, not as permission to keep such files nearby.

## License

MIT. See [LICENSE](LICENSE).
