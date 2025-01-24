---
ensip: TBD
title: ENSIP-XX: Arbitrary Data Storage in the Multichain Address Field
status: Idea
type: Core
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
created: 2025-01-24
---

## Abstract

This ENSIP proposes an expanded use of the existing `addr(bytes32 node, uint256 coinType)` record to store arbitrary data as bytes. Under [ENSIP-9](#), `coinType` is specified as a [SLIP-44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) index, and the returned bytes represent an address for the specified blockchain. However, this proposal extends `coinType` usage so that any arbitrary data can be keyed by a `uint256` value. Specifically, by hashing a descriptive string key using `keccak256(key)`, the resulting `uint256` can serve as the `coinType`, while the associated `bytes` represent arbitrary or unstructured data.

## Motivation

ENS currently has a single dedicated record for storing content, the `contenthash` record (see [ENSIP-7](#)), which encodes web or content address data as a multicodec. However, as new use cases emerge—particularly involving AI, where an ENS name may need to store rich contextual data, chat logs, or agent instructions—there is a desire to use richer record types that can store unstructured binary data.

While ENS does support `text` records ([ENSIP-5](#)), these are intended for human-readable text data and are limited to key-value string pairs. The `addr` record already allows for arbitrary bytes in the second parameter (see [ENSIP-9](#)), but it is tied to SLIP-44 coin type identifiers. By superseding and reinterpreting the `coinType` parameter in this ENSIP as a `keccak256` hash of a descriptive string, the existing resolution methods can store and return any arbitrary bytes under a convenient key scheme. For instance, `aiContext` might become `coinType = uint256(keccak256("aiContext"))`, and the stored bytes can represent AI agent parameters, chain-of-thought, or other binary data.

This extension allows:

1. Backward compatibility with current libraries and resolvers that already use the `addr` interface.
2. A robust mechanism for developers to attach arbitrary data to ENS names without introducing a new record type.
3. The possibility for advanced AI or automation workflows, where AI agents or other services query an ENS name for structured or unstructured data keyed by descriptive strings.

## Specification

### Overview

Instead of using the SLIP-44 coin type (e.g., 60 for Ethereum), a user can derive a `coinType` by `keccak256(keyString)`. For example, `coinType = uint256(keccak256("aiContext"))`.

The resolver function remains as defined in [ENSIP-9](#):

```
function setAddr(bytes32 node, uint coinType, bytes calldata a) external;
function addr(bytes32 node, uint coinType) external view returns (bytes memory);
```

Users call `setAddr(node, keccak256("myKeyString"), myBytesData)` to store arbitrary data. Clients retrieve the data via `addr(node, keccak256("myKeyString"))`.

A resolver that already implements [ENSIP-9](#) can transparently support this proposal. There is no change to the onchain storage or function signatures. The only difference is that the `coinType` integer is now used for a key that is not necessarily a SLIP-44 ID. Resolvers MUST emit the same event used for addresses:

```
event AddressChanged(bytes32 indexed node, uint coinType, bytes newAddress);
```

The concept of “coin type 60 (Ethereum) must reflect the `addr(bytes32)` function in [ENSIP-1](#) still applies, but it remains unaffected by the additional usage for hashed keys.

### Example

Below is an illustrative snippet that shows how to set and retrieve arbitrary data with a hashed string key:

```
pragma solidity ^0.8.0;

interface IExtendedResolver {
    function setAddr(bytes32 node, uint256 coinType, bytes calldata data) external;
    function addr(bytes32 node, uint256 coinType) external view returns (bytes memory);
}

contract ExampleUsage {
    IExtendedResolver public resolver;

    constructor(address resolverAddress) {
        resolver = IExtendedResolver(resolverAddress);
    }

    function storeArbitraryData(bytes32 node, string memory key, bytes memory data) public {
        uint256 hashedKey = uint256(keccak256(abi.encodePacked(key)));
        resolver.setAddr(node, hashedKey, data);
    }

    function retrieveArbitraryData(bytes32 node, string memory key) public view returns (bytes memory) {
        uint256 hashedKey = uint256(keccak256(abi.encodePacked(key)));
        return resolver.addr(node, hashedKey);
    }
}
```

In this example:

- To store data for “aiContext”:
  ```
  storeArbitraryData(myNode, "aiContext", hex"0001ABCD...");
  ```
- To retrieve it:
  ```
  bytes memory result = retrieveArbitraryData(myNode, "aiContext");
  ```

### Rationale

This ENSIP is backward compatible, as it uses the same function signatures from [ENSIP-9](#), requiring no changes to existing tooling or libraries that support `addr(bytes32, uint256)`. Although collisions are theoretically possible due to the hashing of keys, they are improbable with `keccak256`. Implementors should adopt naming schemes, such as namespace-qualified keys (e.g., "eth.vitalik.key"), to avoid collisions. Reusing the `addr` mapping is gas-efficient, with minimal additional overhead, while allowing developers to flexibly store structured or unstructured data. Compared to `text` records ([ENSIP-5](#)), this proposal is better suited for large or binary data, enabling robust and versatile data storage mechanisms.

## Backwards Compatibility

- This proposal does not break existing usage of the `addr(uint256)` function. It simply adds a new convention for choosing the `coinType` value.
- Ensures smooth coexistence with purely SLIP-44 coin-based addresses.

## Implementation

Any resolver implementing [ENSIP-9](#) can immediately support arbitrary data by interpreting the `coinType` parameter as either a SLIP-44 coin type or a hashed string key. The recommended practice is to store large data offchain or in a content-addressable storage if needed (e.g., IPFS or Arweave) and only store references onchain, similar to existing best practices.

## Security Considerations

None.

## Conclusion

This ENSIP expands the `addr(bytes32,node, uint256 coinType)` usage beyond SLIP-44 coin types to store arbitrary bytes keyed by `keccak256` hashes of descriptive strings. It preserves backward compatibility with existing ENS resolution libraries, opening new possibilities for AI-based context storage, complex workflows, and any future applications requiring rich data records on ENS.
