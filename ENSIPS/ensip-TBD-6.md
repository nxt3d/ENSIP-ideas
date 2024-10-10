---
ensip: TBD  
title: Hooks  
status: Idea  
type: Standards Track  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>  
created: 2024-10-7  
---

# Abstract 

This ENSIP introduces `hooks`, a new ENS field to support secure onchain data resolution, including cross-chain data access and ZK-based credentials. Hooks expand ENS functionality beyond simple text resolution, establishing a framework for resolving verified onchain data on ENS names and profiles.

# Motivation

Currently, ENS supports various field types, including addresses (ENSIP-1 and ENSIP-9), text records (ENSIP-5), and content hashes (ENSIP-7). Like text records, hooks use key-value pairs but add the requirement that clients resolve a hook on a specific resolver and chain, allowing for secure, immutable records.

With text records, verifying data can be challenging, as users can change resolvers at will. In the NameWrapper on L1, a fuse can be burned to permanently prevent resolver changes, allowing for secure onchain verified text records. However, most users are unwilling to permanently burn their resolver record. Hooks address this issue: if a user switches to a new resolver, any hooks tied to the original resolver will no longer resolve, ensuring security and maintaining the integrity of hook records.

# Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

This ENSIP introduces a new field, `hook`, which enables the resolution of secure, verified onchain data, including ZK-based credentials, and extends ENSIP-5 (Text Records) to support more complex data requirements. 

```
function hook(bytes32 node, string calldata key, address resolver, uint256 coinType) public returns (string memory)
```

### Syntax

The `hook` function takes the following parameters:
- `node`: The ENS node (namehash) for which the `hook` is being queried.
- `key`: A string representing the specific key for a key-value pair. Keys MUST follow ENSIP-5 standards but may include an exception: the ENSIP-5-compliant key can be appended with a `:` followed by any format, enabling user-developed protocols and extended functionality (e.g., `eth.isprime:17`).
- `resolver`: The address of the resolver smart contract.
- `coinType`: An unsigned integer that specifies the chain for which the resolver is valid, as per ENSIP-11.

When resolving a `hook`, clients MUST supply the address of a resolver and the `coinType` matching the chain of the resolver, in addition to the `node` and `key`. The resolver MUST verify that the provided address matches its own address and that the `coinType` matches the chain of the resolver. This ensures secure and trustless resolution of `hook` records.

While `hook` keys build upon the general structure defined by ENSIP-5, they extend its functionality. Specifically, a key may include a `:` after the ENSIP-5-compliant portion, followed by any additional format, allowing for custom protocol data and expanded use cases. For example, `eth.isprime:17` uses `eth.isprime` as a base key and appends `:17` to accommodate user-specific protocols and more complex data structures.

# Rationale 

It has become increasingly clear that ENS is more than a domain name service; it also serves as a universal onchain profile. However, until now, including verified onchain data in user profiles has been challenging due to the potential for users to change their resolvers. The introduction of `hooks` addresses this limitation, paving the way for applications that incorporate proven onchain data into ENS profiles. 

Since introducing a new field type is a significant change to the ENS protocol, it is necessary to consider the new applications that will be enabled by hooks. One example is `eth.dao.votes`, a hook that can resolve in realtime an ENS Delegate's voting power. Some other applications include resolving verified data onchain using zk-proofs, such as proving a donation to a charity onchain without disclosing the amount or the wallet address (`eth.landfund.donated`), and verifying onchain attestations, like graduating from a coding bootcamp (`eth.codingbootcamp2024.graduated`).

# Security Considerations

`Hooks` enhance security by enabling trustless resolution of onchain data. Unlike text records, which can be vulnerable to manipulation if a user changes their resolver, `hooks` ensure that records are tied to a specific resolver on a specific chain. This prevents spoofing, as any attempt to change the resolver would result in the hook no longer resolving, allowing for verified data to be resolved on ENS names and profiles.

However, it is necessary for clients to adhere strictly to the `hooks` specification's rules. Specifically, a `hook` MUST NOT be resolved from a different resolver or chain than the hook specifies. If clients attempt to resolve a `hook` on an unspecified resolver, it compromises the trustless nature of the hook. However, if clients follow the hooks specification, `hooks` can offer a secure and reliable means of resolving verified onchain data.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


