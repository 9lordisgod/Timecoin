---
name: Design proposal
about: Propose or discuss a change to the Timechain design (consensus, transactions, addresses, threat model, whitepaper errata)
title: "[design] "
labels: design
assignees: ''
---

<!--
Design-only repository. Never paste private keys, seed phrases, mnemonics,
wallet files, or RPC credentials into an issue, including for test networks.
-->

## Summary

One paragraph. What changes, and what it is a change from.

## Whitepaper section

Which section(s) of `whitepaper/timecoin-whitepaper.pdf` this touches, or
"none" if it fills a gap the paper leaves open.

## Area

- [ ] Consensus / parameters (`docs/parameters.md`)
- [ ] Transactions / sighash (`spec/transaction.md`)
- [ ] Addresses / witness program (`spec/addresses.md`)
- [ ] Threat model (`docs/threat-model.md`)
- [ ] Whitepaper erratum (`whitepaper/README.md`)
- [ ] Other

## Motivation

What problem this solves. Which threat-model item it addresses, if any.

## Proposal

Concrete description: byte layouts, constants, algorithms, rule changes.

## Rationale and alternatives

Why this and not the alternatives. What was considered and rejected.

## Security considerations

Post-quantum impact (Shor, Grover), signature size and block weight, the
mempool race after key reveal, any new attack surface.

## Compatibility

Which existing documents need to change, and whether the whitepaper needs a
revision.

## Open questions

## Checklist

- [ ] No secrets, keys, seed phrases, or wallet files included
- [ ] Does not assume Bitcoin's genesis, UTXO set, or vendored Bitcoin Core code
- [ ] Does not propose a mainnet launch, listing, premine, or founder allocation
- [ ] Where the proposal differs from the whitepaper, says so explicitly
