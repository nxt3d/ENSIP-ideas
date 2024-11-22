---
ensip: TBD  
title: Hooks for Secure Onchain Data Resolution  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy.eth <@raffy@unruggable.com>  
created: 2024-10-07  
---

# Abstract 

This ENSIP introduces Hooks, a new ENS resolution method that enables the secure resolution of onchain data through ENS records. For example, Hooks make it possible to securely resolve the number of ENS token votes delegated to an ENS profile via a specialized text record, such as `"eth.ens.get-votes"`.

# Motivation

The Name Wrapper, currently used for all newly registered .eth ENS names on L1 Ethereum, includes a feature called 'fuses.' This allows a name owner to burn a fuse to permanently prevent updates to the resolver record of their name. Preventing resolver updates enables, for instance, secure resolution of text records with onchain data. However, many owners of high-value ENS names, such as those representing their projects, are hesitant to burn permanent fuses. The goal of this ENSIP is to propose an alternative method for securely resolving onchain records. Hooks allow clients to specify a resolver address and chain ID that must be used for resolving ENS records. This achieves the same level of security as burning the `CANNOT_SET_RESOLVER` fuse in the Name Wrapper by ensuring the resolver record matches the intended one, without requiring permanent modifications to the ENS name. 

# Specification

The key words "MUST", "MUST NOT", "REQUIRED", etc., are to be interpreted as described in RFC 2119.

## Using Hooks to Resolve ENS Records on 'Nameless' Resolvers

Hooks are designed to securely resolve ENS records from known resolvers. Resolvers can be any smart contract that implements a `resolve` method and follows this ENSIP.

Hooks allow for resolving ENS records from a smart contract directly, with or without an associated ENS name. A hook comprises:

1. A single ABI-encoded ENS resolver function, such as `text(bytes32 node, string key)` or `contenthash(bytes32 node)`.

2. A resolver address.

3. A chain ID.

Clients may resolve hooks directly from a 'nameless' resolver (a resolver without an associated ENS name). All resolvers MUST implement the `IExtendedResolver` interface specified in ENSIP-10. To call a 'nameless' resolver, a client may leave the `name` argument of the `resolve(name, data)` function blank, such as `bytes("")`. The `data` field must contain the single ABI-encoded ENS resolver function call (as specified in ENSIP-1 and elsewhere), for example `text(node, key)`. The `resolve` function MUST either return valid return data for the specified function or revert if it is not supported.

In some cases, it may be useful to compose a hook using function notation, such as:

```
hook(text(0x123...abc, "isOver18"), 0x234...bcd, 123...456)
```

### Hook Function
```
function hook(
    bytes calldata encodedFunction,
    address resolver,
    uint256 chainId
) 
```

The bytes value of the function selector for `hook()` is `0x8d74c3e9`.

### Parameters

- **`encodedFunction`**: The ABI-encoded function call to the resolver function (e.g., `text`, `contenthash`, `addr`).
- **`resolver`**: The address of the resolver contract that must be used.
- **`chainId`**: The chain ID where the resolver resides.

It is also possible to ABI-encode a hook function by encoding the `hook` function with the resolver function, resolver address, and chain ID as arguments, for compact storage either on or off-chain. However, when a client uses a `hook` with a nameless resolver, the client MUST call the ENS resolver function argument within the hook function on the specified resolver and chain ID and must not use the `hook` function as-is.  

## Using Hooks to Resolve ENS Records from an ENS Name **Without Using** a Universal Resolver

When a client resolves a record using the Hook resolution method without using a Universal Resolver, the client should follow the ENSIP-10 steps to: 

1. Determine the resolver of the name using the steps of ENSIP-10.

2. Check to ensure that the resolver address argument and chain ID of the hook matches the resolver of the name and verify that the resolver implements `IExtendedResolver` using the ERC-165 interface detection specified in ENSIP-10.

3. If the resolver address and chain ID **does not** match the resolver of the name, or the resolver of the name does not implement the `IExtendedResolver` interface, do not resolve the ENS resolver function of the hook.

4. Otherwise, call the `resolve` function of the resolver, using the single ABI-encoded ENS resolver function call as the `data` argument of the call and the DNS-encoded ENS name as the `name` argument, according to ENSIP-10.

5. Get the result according to ENSIP-10, which may require performing an off-chain lookup according to ERC-3668, and also may require decoding according to the ENS resolver function return value type.  

## Using Hooks to Resolve ENS Records from an ENS Name Using a Universal Resolver

When resolving ENS records using a Universal Resolver that supports Hooks, the client MUST use the ABI-encoded `hook` function, wherein the ABI-encoded ENS resolver function is the first argument of the `hook` function, and the known resolver address and chain ID are the second and third arguments. The steps are:

1. Call the `resolve(bytes calldata name, bytes calldata data) external view returns (bytes memory result, address resolver)` function of the Universal Resolver, where the `name` argument is the ENS name to resolve, and `data` is the ABI-encoded `hook` function including an ABI-encoded ENS resolver function as the `data` argument.

2. Get the result, which may require performing an off-chain lookup according to ERC-3668. It may also be necessary to decode the result according to the ENS resolver function return value type.

### Example Client Pseudocode Implementation

Clients implementing this ENSIP must allow users to specify a hook that checks if the address and chain ID of the resolver match the expected values before resolving the ENS record.

Example client call:

```
const textRecord = await resolver.hook(resolverAddress, chainId).text(node, key);
```

#### Explanation

- **Functionality**: This function adds a `hook` to the `text` function call, ensuring that the `resolver` address and `chainId` match the provided values.
- **Verification**: The client verifies:
  - The `resolver` address obtained for the ENS name matches the `resolver` argument.
  - The `chainId` provided matches the chain ID of the resolver.
- **Purpose**: This ensures that the ENS record is only resolved if the specified resolver and chain ID match the `hook`, preventing any unexpected changes due to resolver modifications by the ENS name owner.

#### Example of Encoded Functions for Use with a Universal Resolver

Resolving a `text` function using `hook`:

```
abi.encodeWithSignature(
    "hook(bytes,address,uint256)",
    abi.encodeWithSignature("text(bytes32,string)", node, key),
    resolverAddress,
    1 // Mainnet chain ID
)
```

Resolving a `contenthash` function using `hook`:

```
abi.encodeWithSignature(
    "hook(bytes,address,uint256)",
    abi.encodeWithSignature("contenthash(bytes32)", node),
    resolverAddress,
    1 // Mainnet chain ID
)
```

## Rationale 

By enforcing checks on the resolver address and chain ID, the `hook` function mitigates security risks associated with unexpected resolver changes. This is particularly critical for applications that depend on the security and integrity of onchain records. Previously, clients lacked explicit mechanisms to perform these checks, making it impossible to securely resolve onchain text records. A name owner could change their resolver at any time, effectively altering the values of the onchain resolver records.

## Security Considerations

Clients MUST ensure that the arguments, i.e., the address and chain ID, provided to the `hook` function are correct and trusted. Incorrect values may result in failed resolutions or incorrect results. The `hook` function enhances security by making resolver validations explicit and mandatory for critical ENS record resolutions.

## Backwards Compatibility

Legacy Universal Resolvers that do not implement the `hook` function will fail to resolve the underlying records, which is the intended result. This ensures that only Universal Resolvers adhering to this ENSIP can securely resolve `hook`-wrapped records.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).