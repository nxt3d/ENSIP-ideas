---
title: Semantic Arrow Format for Structured Identifiers  
author: Prem Makeig (premm.eth) <premm@unruggable.com>  
discussions-to: <URL>
status: Idea  
created: 2025-03-22  
---

## Abstract

This ENSIP proposes a semantically meaningful, agent- and human-readable format for expressing blockchain identities and contextual metadata using arrow notation (→). The format supports a progressive structure with optional components for the chain, target, and prompt. It is compatible with ENS resolution infrastructure, particularly ENS text records and contenthash, and introduces a foundation for agent- or prompt-driven applications built on ENS.

## Motivation

ENS names are often used to identify accounts and smart contracts. However, there is currently no easy way to specify an account on a particular chain or express an intent to take an action—such as sending a specific token such as USDC to an account on a particular chain ID. Current formats (like DID or CAIP-10) can be difficult to interpret semantically, are not very human-friendly, and are not optimized for future interactions involving ai agents. With the rise of AI agents, wallet automation, and identity-rich interactions, there is a growing need for a flexible, extensible, and human-readable structure for describing intent and identity.

This ENSIP introduces a new arrow-based format to express such identity chains in a more intuitive and semantic way.

## Specification

The format consists of up to four parts, separated by the UTF-8 character → (U+2192) or the ASCII equivalent -->. Up to one space is optional on either side of the arrow.

```
{address} → {chain} → {target} → {prompt}
```

It is also valid for parts to be separated by a newline character, or visually wrap, such as:
```
{address} → {chain} → 
{target} → {prompt}
```

All parts except the first are optional.

### Part 1: {address}

Required. This identifies the user, wallet, or smart contract. It can be an ENS name (e.g. tom.base.eth) or an Ethereum address (e.g. 0x1234...abcd). ENS names should resolve via standard ENS name resolution using the chain ID specified in Part 2. If Part 2 is omitted, the chain ID of Ethereum Mainnet (i.e. 60) should be assumed.

### Part 2: {chain} (optional)

An ENS name or literal chain ID that represents the blockchain context. If using an ENS name (e.g. `optimism.xyz`), it MUST contain a `text` record with the key `chain_id` whose value is the chain ID (e.g. `10`).

### Part 3: {target} (optional)

A resource being referenced, such as a token or NFT contract, which can be an ENS name (e.g. `usdc.eth`). The native currency of the chain (e.g. ETH) can be represented using the address `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee`. The address used MUST be on the chain specified in Part 2 (e.g. chain ID `10`).

### Part 4: {prompt} (optional)

A semantic instruction that adds context, resolved via an ENS name's `contenthash`. The prompt may specify an intent, a limit, or instruction metadata. Parts 1–3 are fully deterministic and can be handled procedurally by following the protocol. Part 4 is future-looking and non-deterministic, allowing AI agents to reason about and complete actions using the context provided.

For token transfer intents, this field can be used to specify an amount. For example:

- `1000.numtokens.eth` resolves to a plain-text string that defines the number of tokens to be transferred.

Plain text example:
```
- USDC uses 6 decimals. 1000 USDC = `1000000000`
- DAI uses 18 decimals. 1000 DAI = `1000000000000000000000`
- Any other token with `n` decimals: `1000 × 10^n`
```
This allows agents to understand token amounts and execute contextual actions accordingly.

## Examples

1. Basic identifier:

    tom.eth

2. On a specific chain:

    tom.eth → optimism.xyz

3. Token interaction:

    tom.eth → optimism.xyz → usdc.eth

4. With semantic prompt:

    tom.eth → optimism.xyz → usdc.eth → 1000.numtokens.eth

5. With Ethereum addresses:

    0x1234...abcd → 10 → 0xa0b8...

## Rationale

The format improves semantic clarity, agent compatibility, and visual expressiveness. It works alongside ENS infrastructure, leveraging ENS `text` records and `contenthash` for resolution. Unlike tightly packed formats like CAIP-10 or DIDs, this prioritizes readability and contextuality.

It is also backward-compatible with ENS, as all parts after the first part—an ENS name or address—are optional and can be parsed using existing resolution logic.

## Backwards Compatibility

This proposal is fully compatible with current ENS names and resolution logic. It introduces no new onchain requirements and simply proposes a naming convention for inputs that reference ENS-based data.

## Security Considerations

None. Resolution is based on existing ENS infrastructure.

## Conclusion

This ENSIP proposes a lightweight, semantic, and extensible structure for identifying users, chains, contracts, and intents using ENS. It enables intuitive expression of actions and contexts that can be interpreted by humans and intelligent agents alike.

