---
title: Arbitrary Data Storage in the Multichain Address Field
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
discussions-to: <URL>
status: Idea
created: 2025-01-24
---

## Abstract

This ENSIP proposes an expanded use of the existing `addr(bytes32 node, uint256 key)` ENS name resolver field to store arbitrary data as bytes. Building on [ENSIP-9](#), this proposal introduces a hash-based approach for generating a `key` using the following function: 

```
function keyGen(string memory x) pure returns (uint256) {
    return (uint256(keccak256(bytes(x))) - 1) << 32;
}
```

This `key` generation approach avoids conflicts with [ENSIP-9](#) as well as [ENSIP-11](#). It is possible that in the future coinTypes will be derived using a similar hash based function to accommodate large numbers of coinTypes. However, the key generation of this ENSIP will also not conflict with possible future coinType addressing schemes using hashes because of the extremely low chance of collisions of large hashes. Adding an additional function to handle bytes retrieval was considered. However, we believe that the `addr` function is sufficient for retrieving arbitrary `bytes` data (```addr(node, keyGen(key))```), as well as the `setAddr` to store arbitrary `bytes` data (```setAddr(node, keyGen(key), data)```).

## Motivation

ENS currently has a single dedicated record for storing multimedia content, the `contenthash` record (see [ENSIP-7](#)), which encodes web or content address data as a [multicodec](https://github.com/multiformats/multicodec). However, as new use cases emerge—particularly involving AI, where an ENS name may need to store rich contextual data, AI agent graphs, or agent workflows—there is a desire to use richer record types that can store unstructured binary data. While this ENSIP does not specify the types of data that may be used, it is possible, for example, to imagine multicodec records representing IPFS CIDs, URIs, and dataURLs.

While ENS does support `text` records ([ENSIP-5](#)), these are intended for human-readable text data and are limited to key-value string pairs. The `addr` field already allows for storing arbitrary `bytes` values (see [ENSIP-9](#)). By introducing arbitrary key-value pairs in this ENSIP, developers can take advantage of the existing functionality of the `addr` field. For instance, `keyGen("aiContext")` generates a unique key that could be used to store a context as part of an AI agent, as pre-context for LLM prompts.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Overview

To generate a `uint256` key according to this ENSIP, the following function MUST be used, where a descriptive string `x` is input and a `uint256` key is the output:

```
function keyGen(string memory x) pure returns (uint256) {
    return (uint256(keccak256(bytes(x))) - 1) << 32;
}
```

To store data, the function `setAddr` is used: 

```
setAddr(node, keyGen("keyName"), data);
```

To retrieve data, the function `addr` is used:

```
addr(node, keyGen("keyName"));
```

Resolvers that already implement [ENSIP-9](#) are compatible with this approach, as no changes to onchain storage or function signatures are required. 

Resolvers MUST emit the same event used for addresses:

```
event AddressChanged(bytes32 indexed node, uint key, bytes newAddress);
```

### Example

Below is an illustrative snippet that shows how to set and retrieve arbitrary data with `keyGen`:

```
pragma solidity ^0.8.0;

interface IExtendedResolver {
    function setAddr(bytes32 node, uint256 key, bytes calldata data) external;
    function addr(bytes32 node, uint256 key) external view returns (bytes memory);
}

contract ExampleUsage {
    IExtendedResolver public resolver;

    constructor(address resolverAddress) {
        resolver = IExtendedResolver(resolverAddress);
    }

    function keyGen(string memory x) public pure returns (uint256) {
        return (uint256(keccak256(bytes(x))) - 1) << 32;
    }

    function storeArbitraryData(bytes32 node, string memory key, bytes memory data) public {
        uint256 generatedKey = keyGen(key);
        resolver.setAddr(node, generatedKey, data);
    }

    function retrieveArbitraryData(bytes32 node, string memory key) public view returns (bytes memory) {
        uint256 generatedKey = keyGen(key);
        return resolver.addr(node, generatedKey);
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

ENS names have become widely used as identifiers, with the primary use case of user IDs. We believe that a new use case is imminent, namely identifiers for AI agents. AI agents often need access to rich data, including text and images, which could be in the form of a PDF document, for example, to use as part of the context of prompts that bring the AI agent to life. The development of AI agents is still nascent, but it is clear that in order for an ENS name to represent the identity of an AI agent, it is necessary to store data beyond the existing `contenthash` field, which is broadly understood to be used in the context of web browsers.

One approach that was considered was creating a two-dimensional `contentHash` field, such as `hash(bytes32 node, string key) returns (bytes)`. For example, it would be possible to have an `aicontext` key used to retrieve a media file stored as `bytes`. However, a new field would be incompatible with existing resolvers and would also require clients to be updated. This is a significant hurdle to adoption.

It is already possible to use the existing resolver field `addr` to store arbitrary `bytes`, but there is currently no existing standard. With this ENSIP, it is possible to use the existing `addr` field in a generic way to support any type of application, including AI agents. For most applications, data can be stored offchain or in a content-addressable storage system (e.g., IPFS or Arweave), with only references stored onchain. This ENSIP does not specify the ways in which the `bytes` data is encoded, leaving it intentionally unspecified so that future use cases can be developed for specific applications.

## Backwards Compatibility

This proposal does not break existing usage of the `addr` function. It ensures smooth coexistence with current implementations under [ENSIP-9](#) by adopting the same interface and event definitions.

## Security Considerations

None.

## Conclusion

This ENSIP expands the `addr(bytes32,node, uint256 key)` usage by introducing a structured key. It preserves backward compatibility with existing ENS resolution libraries and provides a flexible mechanism for storing and organizing rich data records in ENS names.
