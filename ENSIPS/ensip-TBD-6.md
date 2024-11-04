---
ensip: TBD  
title: Hooks for Secure Onchain Data Resolution and Onchain Agents  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy (@raffy@unruggable.com>  
created: 2024-10-07  
---

# Abstract 

This ENSIP introduces the `hook()` wrapper function, which wraps and validates calls to resolver functions such as `addr`, `text`, and `contenthash`. The `hook()` function ensures that the resolver address and chain ID are checked to confirm that the expected values are used before a resolver function is executed. This allows clients to "lock" resolver records, ensuring that any relied-upon onchain records can't be changed by the user simply changing their resolver record on their ENS name. The `hook()` function should be implemented by any Universal Resolver. 

# Motivation

ENS records like addresses, text records, and content hashes are critical for various applications. However, if a resolver changes unexpectedly, clients relying on these values may end up getting completely unexpected data. The `hook()` function addresses this by enforcing checks on the resolver address and chain ID before executing resolver functions. 

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, etc., are to be interpreted as described in RFC 2119.

## `hook()` Function Definition

The `hook()` function wraps calls to resolver functions, ensuring that specified checks are made. 

### Function Signatures

```
function hook(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId
) 
```

### Parameters

- **`encodedFunction`**: The ABI-encoded function call to the resolver function (e.g., `text`, `contenthash`, `addr`).
- **`resolver`**: The address of the resolver contract that must be used.
- **`chainId`**: The chain ID where the resolver resides.

### Usage: Universal Resolver

Clients MUST use the `hook()` function when resolving ENS records using a compatible Universal Resolver that require validation of the resolver address and chain ID. This function ensures that the resolver has not changed unexpectedly and that the client is interacting with the known contract on the correct chain. 

#### Example Usage

Resolving a `text` function using `hook()`:

```
hook(
    abi.encodeWithSignature("text(bytes32,string)", node, key),
    resolverAddress,
    1 // Mainnet chain ID
)
```

Resolving a `contenthash` function using `contenthash()`:

```
hook(
    abi.encodeWithSignature("contenthash(bytes32)", node, key),
    resolverAddress,
    1 // Mainnet chain ID
)
```

Resolving an `addr` function using `hook()`:

```
hook(
    abi.encodeWithSignature("addr(bytes32)", node),
    resolverAddress,
    1 // Mainnet chain ID
)
```

## Rationale 

By enforcing checks on the resolver address and chain ID, the `hook()` function can prevent security issues that may arise from unexpected resolver changes. This is especially important for applications that rely on the permanence and integrity of onchain records. Clients in the past were not explicitly given permission to do these types of checks, and it therefore was not possible to resolve onchain text records securely because the name owner could change their resolver at any time, effectively changing the values of the onchain resolver records.

## Security Considerations

Clients MUST ensure that the arguments, i.e., address and chain ID, provided to the `hook()` function are correct and trusted. Incorrect values may lead to failed resolutions. The `hook()` function enhances security by making resolver validations explicit and mandatory for critical ENS record resolutions.

## Backwards Compatibility

Legacy clients that do not implement the `hook()` function will fail to resolve the underlying records, which is the intented result. This is intentional to ensure that only clients adhering to this ENSIP can resolve `hook()` wrapped records securely.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
