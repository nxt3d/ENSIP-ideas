---
ensip: TBD  
title: Wrapper Function for ENS Record Resolution  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy (@raffy.eth) <raffy@unruggable.com>  
created: 2024-11-03  
---

# Abstract 

This ENSIP introduces the `get()` wrapper function, which wraps and validates calls to resolver functions such as `addr`, `text`, and `contenthash`. The `get()` function ensures that resolver addresses, chain IDs, ENS version numbers, and optional gateway URLs are checked to ensure that the expected values are used before a resolver function is executed. This allows clients to "lock" resolver records, ensuring that any relied-upon onchain records can't be changed by the user simply changing their resolver record on their ENS name. The `get` function should be implemented by any Universal Resolver. If a client is not using a Universal Resovler, it should follow the steps in this specification to unwrap resolver calls, wrapped in a `get` function. 

# Motivation

ENS records like addresses, text records, and content hashes are critical for various applications. However, if a resolver changes unexpectedly, clients relying on these values may end up getting completely unexpected data. The `get()` function addresses this by enforcing checks on the resolver address, chain ID, version numbers, and gateway URLs of any offchain lookups before executing resolver functions. In the case of the ENS `hook` field, it is necessary to check the resolver address and chain ID of the hook before resolving the record.

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, etc., are to be interpreted as described in RFC 2119.

## `get()` Function Definition

The `get()` function wraps calls to resolver functions, ensuring that specified checks are made. Overloaded versions of `get()` are included to provide flexibility.

### Function Signatures

```
function get(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId,
    string[] calldata urls,
    uint256[] calldata versions
) public returns (string memory)
```

```
function get(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId,
    string[] calldata urls
) public returns (string memory)
```

```
function get(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId,
    uint256[] calldata versions
) public returns (string memory)
```

```
function get(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId
) public returns (string memory)
```

```
function get(
    bytes calldata encodedFunction,
    string[] calldata urls,
    uint256[] calldata versions
) public returns (string memory)
```

```
function get(
    bytes calldata encodedFunction,
    string[] calldata urls
) public returns (string memory)
```

### Parameters

- **`encodedFunction`**: The ABI-encoded function call to the resolver function (e.g., `hook`, `addr`, `text`).
- **`resolver`**: The address of the resolver contract that must be used.
- **`chainId`**: The chain ID where the resolver resides.
- **`urls`**: An array of URLs for any offchain lookups (used in CCIP-Read).
- **`versions`**: An array of `uint256` specifying the allowed versions of the ENS protocol (ENSv2 is currently in development, and will be identified with the number 2).

### Usage: Universal Resolver

Clients MUST use the `get()` function when resolving ENS records using a compatible Universal Resolver that require validation of one or more of the resolver address, chain ID, gateway URLs, and version numbers. When used with the address and chain ID parameters, this function ensures that the resolver has not changed unexpectedly and that the client is interacting with the correct contract on the correct chain. When the gateway URLs are used, the client knows exactly which gateway URLs can be used. If version numbers are used, the client knows up to which version of ENS can be used.

### Usage: Client Implementation

If a client is using their own implementation of the ENS name resolution process or using a library that does not use a Universal Resolver, it is necessary for the client to ensure that only the resolver address and chain ID specified in the `get()` function are used to resolve records on the ENS name. Additionally, if the resolver reverts with an Offchain Lookup, the gateways must be checked against the `urls` specified in the `get()` function to make sure that the gateway used is included. If using the new ENSv2, the client should check to make sure that version 2 is included in the list of versions. If any of the checks do not pass at any time, the client must immediately halt resolving the name and may return an error but MUST NOT return a result.



#### Example Usage

Resolving a `hook` function using `get()` with all parameters:

```
get(
    abi.encodeWithSignature("hook(bytes32,string)", node, key),
    resolverAddress,
    1,        // Mainnet chain ID
    ["https://example.com/hook-endpoint"], // Array of URLs
    [2]       // Array of version numbers
)
```

Resolving an `addr` function using `get()`:

```
get(
    abi.encodeWithSignature("addr(bytes32)", node),
    resolverAddress,
    1         // Mainnet chain ID
)
```

Resolving a `text` function using `get()` with URLs:

```
get(
    abi.encodeWithSignature("text(bytes32,string)", node, key),
    ["https://example.com/text-endpoint"] // Array of URLs
)
```

## Rationale 

By enforcing checks on parameters such as the resolver address and chain ID, the `get()` function can prevent security issues that may arise from unexpected resolver changes. This is especially important for applications that rely on the permanence and integrity of onchain records. Clients in the past were not explicitly given permission to do these types of checks, and it therefore was not possible to resolve onchain text records securely because the name owner could change their resolver at any time, effectively changing the values of the onchain resolver records.

Including arrays for URLs and version numbers allows clients to specify multiple acceptable gateways and versions, providing flexibility and redundancy.

## Security Considerations

Clients MUST ensure that the parameters provided to the `get()` function are correct and trusted. Incorrect parameters may lead to failed resolutions or security vulnerabilities. The `get()` function enhances security by making resolver validations explicit and mandatory for critical ENS record resolutions.

## Backwards Compatibility

Legacy clients that do not implement the `get()` function will not be able to resolve records that require these checks, such as `hooks`. This is intentional to ensure that only clients adhering to this ENSIP can resolve such records securely.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
