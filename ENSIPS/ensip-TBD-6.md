---
ensip: TBD  
title: Hooks for Secure Onchain Data Resolution and Onchain Agents  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy (@raffy.eth) <raffy@unruggable.com>  
created: 2024-10-07  
---

# Abstract 

This ENSIP introduces `hooks`, a new ENS field to support secure onchain data resolution, including zero-knowledge (ZK)-based credentials. Hooks establish a framework for resolving verified onchain data directly from smart contracts (resolvers) as well as ENS names.

# Motivation

Currently, ENS supports various field types, including addresses (ENSIP-1 and ENSIP-9), text records (ENSIP-5), and content hashes (ENSIP-7). Like text records, hooks use key-value pairs but add the requirement that clients resolve a hook on a specific resolver and chain, allowing for secure, immutable records.

With text records, verifying data can be challenging, as users can change resolvers at will. In the NameWrapper on L1, a fuse can be burned to permanently prevent resolver changes, allowing for secure onchain verified text records. However, most users are unwilling to permanently burn their resolver record. Hooks address this issue: if a user switches to a new resolver, any hooks tied to the previous resolver will no longer resolve, ensuring security and maintaining the integrity of hook records.

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

## Hook Function Definition

Hooks are similar to text records; however, they MUST make sure that the intented resolver and chainid are used for the hook. When using a Universal Resolver that supports the `get()` wrapper function, specified in [ENSIP: Wrapper Function for ENS Record Resolution](./ensip-TBD-9.md), it can be used with the resolver address argument and chainId, to make sure the hook is always resolved from the correct resolver contract and on the correct chain.

The `hook` function is defined as:

<code>
function hook(bytes32 node, string calldata key) public returns (string memory)
</code>

### Hook Function Parameters

The `hook` function takes the following parameters:

- **`node`**: The ENS node (namehash) for which the `hook` is being used. Smart contracts that resolve hooks directly (not in the context of resolving an ENS name) can use the `node` value as a `bytes32` ID.

- **`key`**: A string representing the specific key for a key-value pair.

Hook keys have a format that includes two sequential parts. Part one must use the format from ENSIP-5 Text Records. Part two is prefixed with `:` and then followed by UTF-8 text, which has no defined format, allowing hook implementers to use whatever format suits their particular application. For example, `eth.isprime:17` uses `eth.isprime` as a the prefix (ENSIP-5) which is followed by the followed by `:17` to handle user inputs.

### Resolving Hooks

To resolve a hook, clients MUST use make sure the hook is resolved using the intented resolver address and chain id. When using a compatible UR, the `get()` wrapper function can be used, as specified in ENSIP: Wrapper Function for ENS Record Resolution, to wrap the hook with an intented resolver address and chainid, which makes sure the hook is not resolved on an unintented resolver. 

When documenting a hook for clients to resolve it is recommended that a hook be documented wrapped in a `get()` function, so that it is clear that the hook must be resolved using the specified resovler and chainid. 

<code>
get(hook(0x123...abc, "eth.dao.votes:vitalik.eth"), 0x345...def, 1);
</code>

For example, where, 0x123...abc is the node, 0x345...def is the resovler address, and 1 is the chain id. 


An example of resolving a hook using the `get()` function:

<code>
get(
    abi.encodeWithSignature("hook(bytes32,string)", node, key),
    resolverAddress,
    1, // Mainnet chain ID
)
</code>

# Rationale 

ENS serves as a universal onchain profile, but including verified onchain data in user profiles has been challenging due to the potential for users to change their resolvers at will. The introduction of `hooks` addresses this limitation, paving the way for applications that incorporate verified onchain data.

An example of using hooks with ENS is an ENS sub-protocol resolved using the name get-votes-protocol.eth that resolves the votes of any ENS name. The hook could be if called within a `get` wrapper function to make sure the correct resolver and chain id are used. 

<code>
get(hook(0x123...abc, "eth.dao.votes:vitalik.eth"), 0x345...def, 1);
</code>

`vitalik.eth` would be forward-resolved to obtain an Ethereum address to look up the voting power based on ENS token delegations.

# Security Considerations

`Hooks` enhance security by ensuring that records are tied to a specific resolver on a specific chain. Clients using a compatible Universal Resolver MUST use the `get()` wapper function to resolve hooks with the resolver address and chain id parameters, as this ensures the correct resolver and chain ID are used. Resolving hooks without this function may compromise the trustless nature of the hook.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
