---
ensip: TBD
title: Coin Type Priority List
status: Idea
type: Core
author: Prem Makeig (premm.eth) <premm@unruggable.com>
created: 2024-10-28
---

## Abstract

This ENSIP proposes a standard for specifying a priority list of coinTypes using an ENS text record. The `cointype-priority-list` text record key will store a list of `uint256` coinTypes, enabling applications and wallets to determine the preferred blockchain networks for sending funds or interacting with smart contracts associated with an ENS name.

## Motivation

Currently, there is no way for an ENS name owner to express their preference as to which chain they prefer to receive funds. If an ENS name is used to name a smart contract or smart account, there is no way to discover which chain or chains that smart account is on.

By introducing a coinType priority list, we provide a standardized method for users to indicate their preferred networks in a priority list. There are many ways in which clients and applications can use this information, including suggesting which chain the ENS name owner prefers to receive funds, as well as discovering the coinType of a smart account or smart contract only deployed to a single chain, for example.

## Specification

### Cointype Priority List Text Record

Defines the use of the `cointype-priority-list` record.  
This record aims to store a prioritized list of `coinType` identifiers, as defined in [ENSIP-11](https://docs.ens.domains/ensip/ensip-11), indicating a preferred list of blockchains.  
Under ideal circumstances, this record relies on a yet-to-be-written ENSIP that allows for multidimensional records.  
However, currently the `cointype-priority-list` record stores a list of `uint256` values that represent coinTypes, encoded as UTF-8 strings, where each coinType is represented as a 10-digit, zero-padded integer. This format provides a simple and consistent way to store and retrieve coinTypes.

#### Encoding Format

Each coinType is represented as a UTF-8 encoded 10-digit string, zero-padded to ensure a consistent length. The coinTypes are concatenated in order of priority without delimiters.

For example:
- **Ethereum Mainnet** (coinType **60**) would be represented as `0000000060`
- **Base** (coinType **2147492101**) would be represented as `2147492101`
- **Arbitrum One** (coinType **2147525809**) would be represented as `2147525809`

The complete `cointype-priority-list` for the above example, concatenated in priority order, would be stored as:

```000000006021474921012147525809```

#### Priority Order

The list should be interpreted from most to least priority, starting with the first `coinType` at index 0. Each 10-character section of the concatenated string represents a single coinType, with higher priority given to earlier entries.

Applications should interpret the priority from the first element (highest priority) to the last (lowest priority).

### Multidimensional Text Records (Future Consideration)

If multidimensional text records are implemented in the future, it would be beneficial to have a key and index structure, where each coinType in the priority list can be accessed by its index (e.g., index 0 for the highest priority coinType, index 1 for the next, and so on).

This would allow for more efficient storage and retrieval of the priority list within ENS records and could simplify the parsing process for applications.

## Security Considerations

None.

## Backwards Compatibility

This ENSIP introduces a new text record key, `cointype-priority-list`, and does not interfere with existing ENS records or functionality. Applications that do not support this standard will simply ignore the new text record, ensuring backward compatibility.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
