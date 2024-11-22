---
simd: "XXXX"
title: Transaction V1 with Resource Requests
authors:
  - keith@metaplex.foundation
category: Standard
type: Core
status: Draft
created: 2024-11-22
feature:
---

## Summary

Add explicit resource requests to the `VersionedTransaction` type rather than to the ComputeBudget Program. No longer force inclusion of the ComputeBudget instruction in all transactions and remove it from signed data entirely.

## Motivation

Core devs have expressed a desire to better define transaction resource constraints by including explicit resource requests in a transaction. Currently this is done by overloading the ComputeBudget Program. However, this has several drawbacks such as transaction size, overhead, and difficulty building transactions. This data would be better represented and more efficient as native fields in the `VersionedTransaction` object.

Additionally, most transactions originating from frontend wallets contain a `ComputeBudgetInstruction` injected via the wallet provider. For the sake of clarity and DevEx, these resource request fields should be removed from the signed data entirely to clearly indicate that these fields have no guaranteed originator.

## Alternatives Considered

- **Further Extension of the ComputeBudgetInstruction** - Not the optimal representation of the data.

## New Terminology

**ResourceRequest** - An enum type for the different resources that can be requested for a transaction's execution.

**Resources** - Part of the `V1` `VersionedTransaction` variant. An array of the `ResourceRequest`s that can are requested for a transaction.

## Detailed Design

```rust
pub enum ResourceRequest {
    HeapFrame(u32),
    ComputeUnitLimit(u32),
    ComputeUnitPrice(u64),
    LoadedAccountsDataSizeLimit(u32),
    ...
    ChilliPeppers(u32),
    MeowMeowBeenz(u32),
}

pub enum VersionedMessage {
    Legacy(Message),
    V0(Message),
    V1{message: Message, resources: ResourceRequest[]}
}

impl VersionedTransaction {
  ...

  /// Return the message containing all data that should be signed.
  pub fn message(&self) -> &Message {
      match (&self.message) {
        VersionedMessage::V1{message, resources} => message,
        Legacy(message) | V0(message) => message,
      }
  }

  ...
}
```

## Impact

- Requires Transaction parsing to happen on VersionTransaction instead of converting it to legacy transactions
- Additional SDK work to support a new transaction format

## Security Considerations

- A more in depth discussion should occur on removing the resource requests from the signed data in case it case there are attack vectors not foreseen in this proposal.
