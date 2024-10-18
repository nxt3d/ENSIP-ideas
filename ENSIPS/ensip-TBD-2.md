---
ensip: TBD
title: Data URL and URI Contenthash
status: Idea
type: ENSRC
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
created: 2024-6-7
---

# Abstract 

This ENSIP extends the `contenthash` field to support two additional content types: data URL and URI.

# Motivation

The `contenthash` field has become the standard for using ENS names for decentralized websites and dapps. With ENSIP-10 and CCIP-Read (EIP-3668), resolving ENS records from L2s reduces the cost of using the `contenthash` field. This makes adopting the [data URL](https://datatracker.ietf.org/doc/html/rfc2397) standard feasible, allowing content like webapps, images, and videos to be stored onchain. This ENSIP also introduces a new URI content type for the `contenthash` field, allowing browsers to redirect to a standard URI when loading an ENS name. While URIs, such as ethereum.org, are not decentarlized or onchain, it makes ENS names more reverse compatible with Web2 and is a convenience for users. 

# Specification

ENSIP-7 introduced the `contenthash` field for resolving ENS names to content hosted on distributed systems such as IPFS and Swarm. The value returned by `contenthash` is represented as a machine-readable multicodec, which permits a wide range of protocols to be supported by ENS names. The format is specified as follows:

```
<protoCode uvarint><value []byte>
```

protoCodes and their meanings are specified in the [multiformats/multicodec](https://github.com/multiformats/multicodec) repository.

This ENSIP intruduces two new types of new multicodecs, uri and eth-calldata (which will be used for the Data URL).  

Until final protoCodes are approved the "Private Use Area" temporary codes should be used.

uri: 0x3000f2

eth-calldata: 0x30009b

## New Formats 
**URI**

Format: `uvarint(codec1) + <URI as utf8 bytes>`

**Data URL**

For Data URL we use ENSIP-TBD-6 Hooks, to direct clients to a smart contract with a specified contract address and coinType (chain id), to resolve the data for the Data URL. 

The format of the hook is the abi encoded bytes (Ethereum calldata) of the function:

```
function hook(bytes32 node, string calldata key, address resolver, uint256 coinType) public returns (string memory)
```
`node` - the node of the ENS name
`key` - a string comprising "" + <Reverse ENS Name i.e. eth.vitalik> + ":" + "dataURL" + <Resolver Address> + <coinType> + ""  
`resolver` - the address of the smart contract (resolver) where the data can be resolved
`coinType` - the coinType (ENSIP-11) of the chain, which also includes EVM chain ids. 

Format: `uvarint(codec2) + <ABI encoded 'hook' function call as bytes>`

## Web Gateway Resolution (e.g. .limo)

**URI:** 

* The HTTP response MUST be a `HTTP 307` Temporary Redirect.
	
* The response `Location` MUST be `$URI` eg. https://domain.com/a/b.c?d=e.

A reasonable limit may be placed by clients on the number of characters in the URI, but at least 256 bytes of UTF-8 characters should be suppported. 

**Data URL:**

* The HTTP reponse MUST be a `HTTP 200` OK.

* The HTTP response MUST be of `Content-type: $MIME`.

When resolving Data URLs, the URL of the request to the gateway is only used to determine the ENS name. Any path or query data of the request URL is ignored. For example `https://name.eth.limo` returns the same data URL as `https://name.eth.limo/a/b/c`. Single page applications (SPA) that are resovled in the browser may use the path information if necesasry to modify the view of the SPA. 

# Rationale 

[ENSIP-7](https://github.com/ensdomains/ensips/blob/master/ensips/7.md) makes it possible to resolve contenthash records, allowing decentralized websites using decentralized storage such as IPFS and Swarm to be resolved using ENS names. Many users, however, would prefer to simply redirect their ENS name to a URI. It is currently possible to include a URI in the text record 'url'; however, this has traditionally been used as a profile record to link to a website of the user, for example, a blog or homepage. This ENSIP makes it possible to redirect the ENS name to a URI using the contenthash field, intended for resolving within the web browser. With the addition of the Data URL contenthash type, it is possible to resolve a decentralized website that is fully onchain, avoiding the need for pinning data, for example, using IPFS.

An ENSIP was previously proposed by NameSys on the ENS DAO forum, [[Draft] ENSIP-17: DataURI Format in Contenthash](https://discuss.ens.domains/t/draft-ensip-17-datauri-format-in-contenthash/18048/7). Several methods for encoding that Data URL were discussed, including bypassing the multicodec and using the IPFS multicodec format among other methods. Adding two new protoCodes was also discussed, and this ENSIP takes that approach to avoid overloading the top-level IPFS codec with other subtypes that aren’t necessarily related to IPFS. Previously, a new data-url protoCode was proposed; however, it became necessary to separate the data URL contenthash and the onchain data, and this ENSIP takes the approach of using an Ethereum calldata protoCode with a special hook to resolve data URIs.



# Security Considerations

Data URLs and URIs are intended for use in web browsers or other user-facing clients, so their security considerations are similar to any web application. However, onchain Data URLs can be safer than a traditional DNS website because the content can be stored entirely onchain, preventing attackers from altering or compromising the website.
  
# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


