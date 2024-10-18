---
ensip: TBD  
title: Hooks  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>  
created: 2024-10-7  
---

# Abstract 

This ENSIP introduces `hooks`, a new ENS field to support secure onchain data resolution including ZK-based credentials. Hooks establish a framework for resolving verified onchain data directy from smart contracts (resovlers) as well as ENS names and profiles.

# Motivation

Currently, ENS supports various field types, including addresses (ENSIP-1 and ENSIP-9), text records (ENSIP-5), and content hashes (ENSIP-7). Like text records, hooks use key-value pairs but add the requirement that clients resolve a hook on a specific resolver and chain, allowing for secure, immutable records.

With text records, verifying data can be challenging, as users can change resolvers at will. In the NameWrapper on L1, a fuse can be burned to permanently prevent resolver changes, allowing for secure onchain verified text records. However, most users are unwilling to permanently burn their resolver record. Hooks address this issue: if a user switches to a new resolver, any hooks tied to the original resolver will no longer resolve, ensuring security and maintaining the integrity of hook records.

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

This ENSIP introduces a new resolver field, `hook`, which enables the resolution of secure, verified onchain data, including ZK-based credentials, and extends ENSIP-5 (Text Records) to support more complex data requirements. 

```
function hook(bytes32 node, string calldata key, address resolver, uint256 coinType) public returns (string memory)
```

The hook is resolved against a smart contract (resolver) that supports the Extended Resolver (ENSIP-10) interface using the 'resolve' function. To resolve a hook, the 'resolve' function is called with the encoded calldata of the hook function (i.e., the function signature of the 'hook' function is 0x6643fd22). If the resolver is called directly, the 'name' parameter can be left blank or used by the resolver as needed.

```
interface ExtendedResolver {
    function resolve(bytes calldata name, bytes calldata data) external view returns(bytes);
}
```
The resolver must conform to the Extended Resolver interface as specified in ENSIP-10 and must return bytes when the resolve function is successfully called. However, it is not necessary for clients to follow the resolution steps of ENSIP-10 when resolving hooks, though they may choose to. If a resolver implements the Extended Resolver interface, it MUST return 'true' when `supportsInterface()` (EIP-165) is called on it with the interface's ID, 0x9061b923. Resolvers that support the `hook` field may also choose to have a public function called `hook`, with identical inputs and output results as resolving the hook field with the resolve function, but it is not required.

When resolving a hook from an ENS name, a client MUST find the resolver address of the ENS name (ENSIP-1 or ENSIP-10). The client MUST ensure that the resolver of the ENS name matches the resolver specified in the hook, including the coinType (ENSIP-11). This ensures that if the ENS name owner changes their resolver, the hook will no longer resolve, thereby protecting clients who depend on the resolved data from receiving spoofed or incorrect information.

## Resolver Validation 

The resolver smart contract MUST verify that the hook contract address and coinType match its own address and coinType (chain ID). If the check fails, the contract MUST revert with a custom error called `UnknownHook`, providing the expected and provided values for both the address and coinType. Additionally, contracts supporting the `IHookResolver` interface MUST return `true` when `supportsInterface()` is called with the interface ID corresponding to `IHookResolver`. The required interface is as follows:

```
interface IHookResolver {
    
    // Custom error for incorrect resolver address and coinType
    error UnknownHook(address expectedResolver, address providedResolver, uint256 expectedCoinType, uint256 providedCoinType);
}
```

## Hook function parameters

The `hook` function takes the following parameters:
- `node`: The ENS node (namehash) for which the `hook` is being queried.
- `key`: A string representing the specific key for a key-value pair. Keys MUST follow ENSIP-5 standards but may include an exception: the ENSIP-5-compliant key can be appended with a `:` followed by any format, enabling user-developed protocols and extended functionality (e.g., `eth.isprime:17`).
- `resolver`: The address of the resolver smart contract.
- `coinType`: An unsigned integer that specifies the chain for which the resolver is valid, as per ENSIP-11.

While `hook` keys build upon the general structure defined by ENSIP-5, they extend its functionality. Specifically, a key may include a `:` after the ENSIP-5-compliant portion, followed by any additional format, allowing for custom protocol data and expanded use cases. For example, `eth.isprime:17` uses `eth.isprime` as a base key and appends `:17` to be able to handle user inputs.

# Rationale 

It has become increasingly clear that ENS is more than just a domain name service; it also serves as a universal onchain profile. However, until now, including verified onchain data in user profiles for example has been challenging due to the potential for users to change their resolvers at will. The introduction of `hooks` addresses this limitation, paving the way for applications that incorporate verified onchain data, for example, with ENS profiles.

If hooks are used with ENS resolvers, this would represent the introduction of a new ENS field type, which would be a significant change to the ENS protocol. It is therefore necessary to make a case for why this change creates significant value for the ENS protocol.

One example could be an ENS sub-protocol that resolves the votes of any ENS name. The hook could be resolved against the domain votes.dao.eth, and the hook itself could be:

```
hook(0x123, "eth.dao.votes:vitalik.eth",0x234,0x3c)
```

Vitalik.eth would be forward-resolved to obtain an L1 Ethereum address to look up the voting power based on ENS token delegations.

A hook is also used in ENSIP-TBD-2: Data URL and URI Contenthash, along with a new eth-calldata protocode to resolve onchain sigle page web applications and multimeida content. 

Other applications include resolving verified onchain data using zk-proofs, such as proving a donation to a charity onchain without disclosing the amount or the wallet address, and verifying onchain attestations, such as graduating from a coding bootcamp. For ENS names on Layer 2s, for example name.layer2.eth, it is possible to resolve any kind of onchain data in real time using hooks, with the L1 resolver of layer2.eth. This could include, for example, a multi-address profile that identifies multiple addresses for each chain owned by a user, proven onchain in real time. 

# Security Considerations

`Hooks` enhance security by enabling trustless resolution of onchain data. Unlike text records, which can be vulnerable to manipulation if a user changes their resolver, `hooks` ensure that records are tied to a specific resolver on a specific chain. This prevents spoofing, as any attempt to change the resolver would result in the hook no longer resolving, allowing for verified data to be resolved on ENS names and profiles.

However, it is necessary for clients to adhere strictly to the `hooks` specification's rules. Specifically, a `hook` MUST NOT be resolved from a different resolver or chain than the hook specifies. If clients attempt to resolve a `hook` on an unspecified resolver, it compromises the trustless nature of the hook. However, if clients follow the hooks specification, `hooks` can offer a secure and reliable means of resolving verified onchain data.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


