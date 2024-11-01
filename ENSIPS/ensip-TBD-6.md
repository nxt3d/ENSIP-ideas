---
ensip: TBD  
title: Hooks for Secure Onchain Data Resolution and Onchain Agents  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, Raffy (@raffy.eth) <raffy@unruggable.com>  
created: 2024-10-07  
---

# Abstract 

This ENSIP introduces `hooks`, a new ENS field to support secure onchain data resolution, including zero-knowledge (ZK)-based credentials and AI agent interactions. Hooks establish a framework for resolving verified onchain data directly from smart contracts (resolvers) as well as ENS names.

# Motivation

Currently, ENS supports various field types, including addresses (ENSIP-1 and ENSIP-9), text records (ENSIP-5), and content hashes (ENSIP-7). Like text records, hooks use key-value pairs but add the requirement that clients resolve a hook on a specific resolver and chain, allowing for secure, immutable records.

With text records, verifying data can be challenging, as users can change resolvers at will. In the NameWrapper on L1, a fuse can be burned to permanently prevent resolver changes, allowing for secure onchain verified text records. However, most users are unwilling to permanently burn their resolver record. Hooks address this issue: if a user switches to a new resolver, any hooks tied to the previous resolver will no longer resolve, ensuring security and maintaining the integrity of hook records.

Additionally, hooks enable the creation of secure text interfaces for smart contracts, which can be used to create verified onchain agents. Any prompt can be directed to the smart contract using a hook, and the result of the hook, using ERC-3668 CCIP-Read (Offchain Lookup), can be directed to a URL, which can be an AI agent model that responds within the context of the onchain agent. The onchain agent smart contract can also process the returned data, verifying the data received from the offchain agent. For example, the callback function can look for a structured response that includes onchain calls which the callback function can execute to verify the response. 

For example a hook can be sent to an onchain agent (resovler) with the prompt, "How many ENS votes do you have?" The hook would be sent to the offchain AI model, along with the bot's context data, which may include any number of text or multimedia resources that the offchain AI model can use as part of the prompt's pre-context. This context could be onchain or offchain. The AI model's response might be "I have <<calldata bytes>> ENS delegated votes." The calldata bytes could then be returned to the client as text, where the calldata bytes are replaced with the text-encoded answers to the calls, such as "I have 500 ENS delegated votes." This ENSIP does not attempt to define any standard for how prompts are resolved and intentionally leaves that up to implementers who may develop their own standards.

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

This ENSIP introduces a new resolver field, `hook`, which enables the resolution of secure, verified onchain data, including ZK-based credentials and interactions with AI agents.

```
function hook(bytes32 node, string calldata key, address resolver, uint256 chainId) public returns (string memory)
```

The hook is resolved against a smart contract (resolver) that supports the Extended Resolver interface specified in ENSIP-10, using the `resolve` function. A smart contract that supports `hooks` does not need to follow ENSIP-10 beyond implementing the Extended Resolver interface; however, it can. To resolve a hook, the `resolve` function is called with the encoded calldata of the `hook` function (function signature `0x6643fd22`). If the resolver is called directly (not in the context of resolving an ENS name), the `name` parameter from the resolve function can be left blank.

```
interface IExtendedResolver {
    function resolve(bytes calldata name, bytes calldata data) external view returns (bytes memory);
}
```

The resolver must conform to the Extended Resolver interface as specified in ENSIP-10 and must return bytes when the `resolve` function is successfully called. It MUST return `true` when `supportsInterface()` (EIP-165) is called on it with the interface's ID, `0x9061b923`. Resolvers that support the `hook` field may also choose to have a public function called `hook`, with identical inputs and output results as resolving the hook field with the `resolve` function, but it is not required.

When resolving a hook from an ENS name, a client MUST find the resolver address of the ENS name (ENSIP-10). The client MUST ensure that the resolver of the ENS name matches the resolver specified in the hook, including the chain ID. This ensures that if the ENS name owner changes their resolver, the hook will no longer resolve, thereby protecting clients who depend on the resolved data from receiving spoofed or incorrect information. If the name is resolved using the Universal Resolver, a new ENS resolution method where an onchain utility is used to discover the resolver of the ENS name and resolve one or more ENS records using a single call, the Universal Resolver must support `hooks` and must perform the resolver check to ensure the hook resolver address matches the resolver address of the name.

## Resolver Validation 

The resolver smart contract MUST verify that the hook contract address and chain ID match its own address and chain ID. If the check fails, the contract MUST revert with a custom error called `UnknownHook`, providing the expected and provided values for both the address and chain ID. Additionally, hook resolvers must implement the `IHookResolver` interface and MUST return `true` when `supportsInterface()` is called with the interface ID corresponding to `IHookResolver`. The required interface is as follows:

```
interface IHookResolver {

    // Custom error for incorrect resolver address or chain ID
    error UnknownHook(address expectedResolver, address providedResolver, uint256 expectedChainID, uint256 providedChainID);
}
```

## Hook Function Parameters

The `hook` function takes the following parameters:

- **`node`**: The ENS node (namehash) for which the `hook` is being used. Smart contracts that resolve hooks directly (not in the context of resolving an ENS name) can use the `node` value as a `bytes32` ID (which can be converted internally to `uint256`), for example, an ID of an NFT.

- **`key`**: A string representing the specific key for a key-value pair. 

- **`resolver`**: The address of the resolver smart contract.

- **`chainId`**: An unsigned integer that specifies the chain ID for which the resolver is valid.

Hook keys have a format that includes two sequential parts, part one must use the format from ENSIP-5 Text Records. Part two is prefixed with `:` and then followed by UTF-8 text, which has no defined format, allowing hook implementers to use whatever format suits their particular application. For example, `eth.isprime:17` uses `eth.isprime` as a base key (ENSIP-5) and appends `:17` to handle user inputs.

## Interaction with Offchain Agents

By using ERC-3668 CCIP-Read (Offchain Lookup), the client can be directed to a URL of a gateway of an AI agent model. The AI agent model can process the request and respond within the context of the onchain agent. The returned results of the gateway can be supplied to the callback function of the onchain agent (resovler). The resolver can then process and verify the data returned from the offchain agent.

This allows for secure and verifiable interactions with AI agents, enabling the creation of verified onchain agents that can interact with offchain AI models while maintaining security and trustlessness.

# Rationale 

It has become increasingly clear that ENS is more than just a domain name service; it also serves as a universal onchain profile. However, until now, including verified onchain data in user profiles has been challenging due to the potential for users to change their resolvers at will. The introduction of `hooks` addresses this limitation, paving the way for applications that incorporate verified onchain data, such as with ENS profiles.

An example of using hooks with ENS is an ENS sub-protocol that resolves the votes of any ENS name. The hook could be resolved against the domain `votes.dao.eth`, and the hook itself could be:

```
hook(0x123, "eth.dao.votes:vitalik.eth", 0xabc...234, 0x3c)
```

`vitalik.eth` would be forward-resolved to obtain an L1 Ethereum address to look up the voting power based on ENS token delegations.

Hooks can also be used to resolve onchain single-page web applications and multimedia content. For example, there could be a hook with the key "eth.vitalik:dataURL" that returns a Data URL with a complete onchain website. 

Other applications include resolving verified onchain data using ZK-proofs, such as proving a donation to a charity onchain without disclosing the amount or the wallet address, and verifying onchain attestations, such as graduating from a coding bootcamp. For ENS names on Layer 2s, for example `name.layer2.eth`, it is possible to resolve any kind of onchain data using hooks, using the L1 resolver of `layer2.eth`, to resolve the hook using the data stored on L2 for the name. An example of a hook could be, for example, a proof-of-humanity verification, or a multi-address profile that identifies multiple addresses for each chain owned by a the L2 name holder. 

# Security Considerations

`Hooks` enhance security by enabling trustless resolution of onchain data. Unlike text records, which can be vulnerable to manipulation if a user changes their resolver, `hooks` ensure that records are tied to a specific resolver on a specific chain. This prevents spoofing, as any attempt to change the resolver would result in the hook no longer resolving, allowing for verified data to be resolved on ENS names and profiles.

However, it is necessary for clients to adhere strictly to the `hooks` specification, as well as for resolver smart contracts to verify the hook's contract address and chain ID. Specifically, a `hook` MUST NOT be resolved from a different resolver or chain than the hook specifies. If clients attempt to resolve a `hook` on an unspecified resolver, it may compromise the trustless nature of the hook. 

When interacting with offchain agents via CCIP-Read, the smart contract should verify the returned data to prevent malicious responses. Clients should ensure that any data received from offchain sources is properly validated, either by the callback function, through additional rounds of offchain lookups, or by the client itself before use. 

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
